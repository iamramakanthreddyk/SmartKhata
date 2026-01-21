# Troubleshooting: Architecture Problems & Solutions

**Purpose**: Debug common architectural issues that arise during development  
**When to use**: You're stuck and need to know WHY something is broken  

---

## Problem 1: "Adding field to database broke the app"

### Symptoms
- API returns 500 errors
- Frontend can't parse response
- Database query works but service layer fails

### Root Cause
The adapter layer (where snake_case → camelCase) wasn't updated when database schema changed.

### Solution

**Step 1**: Find where field is transformed
```typescript
// Repository returns: { invoice_number: 100 }
// API expects: { invoiceNumber: 100 }
// Adapter should do: snakeToCamelCase(dbData)
```

**Step 2**: Update adapter function in `/src/adapters/`
```typescript
// src/adapters/InvoiceAdapter.ts
export const invoiceDbToApi = (dbInvoice: any) => ({
  invoiceNumber: dbInvoice.invoice_number,
  invoiceDate: dbInvoice.invoice_date,
  // ✅ ADD NEW FIELD HERE
  discountAmount: dbInvoice.discount_amount,  // ← NEW
});
```

**Step 3**: Update API response schema
```typescript
// src/types/api/InvoiceDto.ts
export interface InvoiceApiDto {
  invoiceNumber: string;
  invoiceDate: Date;
  discountAmount: number;  // ← NEW
}
```

**Step 4**: Update frontend types
```typescript
// frontend/src/types/Invoice.ts
export interface Invoice {
  invoiceNumber: string;
  invoiceDate: Date;
  discountAmount: number;  // ← NEW
}
```

✅ **Now adding a field is safe:**
1. Add column to DB
2. Update adapter (1 place)
3. Update API schema (1 place)
4. Update frontend types (1 place)

---

## Problem 2: "One query broke everything - connection pool exhausted"

### Symptoms
- All API calls fail with "connection timeout"
- Even simple queries fail
- Restarting fixes it temporarily

### Root Cause
Repository didn't close database connections. Each query opened a new connection, exhausting the pool.

### Solution

**Step 1**: Ensure all repositories use connection pool
```typescript
// ❌ WRONG: Creates new connection each time
const result = await postgres.query("SELECT * FROM invoices");

// ✅ RIGHT: Uses connection pool
import { pool } from '../db/pool';
const result = await pool.query("SELECT * FROM invoices");
```

**Step 2**: Use try-finally to ensure cleanup
```typescript
// ✅ GOOD: Explicit cleanup
async getInvoice(id: string) {
  const client = await pool.connect();
  try {
    return await client.query("SELECT * FROM invoices WHERE id = $1", [id]);
  } finally {
    client.release();  // ← CRITICAL
  }
}
```

**Step 3**: For transactions, rollback on error
```typescript
// ✅ GOOD: Handles errors safely
async createInvoiceWithItems(invoice: Invoice, items: Item[]) {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    const invoiceResult = await client.query(
      'INSERT INTO invoices (number) VALUES ($1) RETURNING id',
      [invoice.number]
    );
    // ... add items ...
    await client.query('COMMIT');
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
```

---

## Problem 3: "Deleting a customer deleted all their invoices too"

### Symptoms
- Deleted customer record
- All invoices disappeared
- Can't restore them

### Root Cause
Foreign key constraint with `ON DELETE CASCADE`. One delete triggers cascade deletes.

### Solution

**Step 1**: Change FK constraint from CASCADE to RESTRICT
```sql
-- ❌ WRONG: Deletes cascade
ALTER TABLE invoices 
  ADD CONSTRAINT fk_customer 
  FOREIGN KEY (customer_id) 
  REFERENCES customers(id) 
  ON DELETE CASCADE;

-- ✅ RIGHT: Prevents accidental deletes
ALTER TABLE invoices 
  ADD CONSTRAINT fk_customer 
  FOREIGN KEY (customer_id) 
  REFERENCES customers(id) 
  ON DELETE RESTRICT;
```

**Step 2**: Use soft delete instead of hard delete
```sql
-- Add is_deleted column
ALTER TABLE customers ADD COLUMN is_deleted BOOLEAN DEFAULT false;

-- Mark as deleted, don't actually delete
UPDATE customers SET is_deleted = true WHERE id = $1;

-- Queries automatically filter: WHERE is_deleted = false
```

**Step 3**: Query correctly to exclude deleted records
```typescript
// Repository should always filter soft-deleted records
async getCustomer(id: string) {
  return await pool.query(
    'SELECT * FROM customers WHERE id = $1 AND is_deleted = false',
    [id]
  );
}
```

✅ **Soft delete benefits:**
- Can restore deleted data
- Audit trail preserved
- Foreign keys stay intact

---

## Problem 4: "Naming is inconsistent - camelCase in API but snake_case in responses"

### Symptoms
- Frontend receives mixed field names
- Some fields: `invoiceNumber`, others: `invoice_date`
- Parsing fails or workarounds created

### Root Cause
Adapter layer wasn't applied consistently. Some responses transformed, others didn't.

### Solution

**Step 1**: Centralize all transformations
```typescript
// src/adapters/fieldNameAdapter.ts
export const snakeToCamelCase = (obj: any): any => {
  if (Array.isArray(obj)) return obj.map(snakeToCamelCase);
  if (obj === null || typeof obj !== 'object') return obj;
  
  return Object.keys(obj).reduce((acc, key) => {
    const camelKey = key.replace(/_([a-z])/g, (_, letter) => letter.toUpperCase());
    acc[camelKey] = snakeToCamelCase(obj[key]);
    return acc;
  }, {});
};
```

**Step 2**: Apply ONLY at API boundary
```typescript
// src/routes/invoices.ts
export async function getInvoice(req, res) {
  // Get from DB: snake_case
  const dbInvoice = await InvoiceRepository.get(req.params.id);
  
  // Transform at API boundary ONLY
  const apiResponse = snakeToCamelCase(dbInvoice);
  
  // Send camelCase
  res.json(apiResponse);
}
```

**Step 3**: Never transform in middle layers
```typescript
// ❌ WRONG: Transforming in service
export class InvoiceService {
  async getInvoice(id) {
    const dbInvoice = await repo.get(id);
    // ❌ DON'T transform here
    return snakeToCamelCase(dbInvoice);
  }
}

// ✅ RIGHT: Transform only at API boundary
export class InvoiceService {
  async getInvoice(id) {
    // Service works with raw DB data
    return await repo.get(id);
  }
}

export async function getInvoice(req, res) {
  const data = await service.getInvoice(req.params.id);
  // Transform here at boundary
  res.json(snakeToCamelCase(data));
}
```

---

## Problem 5: "API v1 broke when we added new feature to v2"

### Symptoms
- /api/v1/invoices returns wrong format
- Client apps built on v1 stopped working
- No clear backward compatibility

### Root Cause
Routes weren't versioned. Both v1 and v2 shared same handler.

### Solution

**Step 1**: Create separate routers for each version
```typescript
// src/routes/v1/invoices.ts
export const v1Router = express.Router();
v1Router.get('/:id', async (req, res) => {
  const invoice = await InvoiceService.get(req.params.id);
  // Return v1 format (without new fields)
  res.json({
    invoiceNumber: invoice.invoiceNumber,
    total: invoice.total,
  });
});

// src/routes/v2/invoices.ts
export const v2Router = express.Router();
v2Router.get('/:id', async (req, res) => {
  const invoice = await InvoiceService.get(req.params.id);
  // Return v2 format (includes new discount field)
  res.json({
    invoiceNumber: invoice.invoiceNumber,
    total: invoice.total,
    discountAmount: invoice.discountAmount,  // ← NEW in v2
  });
});
```

**Step 2**: Mount both versions
```typescript
// src/server.ts
app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);
```

**Step 3**: Update docs
- [03_API.md](03_API.md) lists both versions
- Deprecation timeline for v1

✅ **Versioning benefits:**
- Old clients keep working
- New clients can use v2
- Clean deprecation path

---

## Problem 6: "Circuit breaker keeps failing - external API is down"

### Symptoms
- All payment attempts fail
- Razorpay API is slow
- App keeps retrying forever
- Eventually timeouts

### Root Cause
No circuit breaker. App keeps hitting failing service instead of failing fast.

### Solution

**Step 1**: Implement circuit breaker pattern
```typescript
// src/patterns/CircuitBreaker.ts
export class CircuitBreaker {
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';
  private failureCount = 0;
  private lastFailureTime = 0;
  private failureThreshold = 5;
  private resetTimeout = 60000; // 1 minute

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.resetTimeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN - service unavailable');
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }

  private onFailure() {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
    }
  }
}
```

**Step 2**: Use circuit breaker for external calls
```typescript
// src/services/PaymentService.ts
const razorpayBreaker = new CircuitBreaker();

export async function createPayment(invoice: Invoice) {
  try {
    return await razorpayBreaker.execute(() =>
      razorpay.orders.create({ amount: invoice.total })
    );
  } catch (error) {
    // Circuit breaker is open - fail fast
    throw new PaymentUnavailableError('Payment service temporarily unavailable');
  }
}
```

✅ **Circuit breaker benefits:**
- Fails fast instead of hanging
- Auto-recovers after timeout
- Prevents cascading failures

---

## Problem 7: "Can't run two features simultaneously - database locked"

### Symptoms
- Feature A is running a big data import
- Feature B tries to update a record
- Everything waits indefinitely
- Database reports "lock timeout"

### Root Cause
Long-running transaction without proper isolation level.

### Solution

**Step 1**: Use appropriate isolation levels
```typescript
// ✅ GOOD: Read-only queries use READ COMMITTED (default)
async readInvoices() {
  return await pool.query('SELECT * FROM invoices WHERE status = $1', ['PAID']);
}

// ✅ GOOD: Updates use explicit transaction with retry
async updateInvoice(id: string, data: any) {
  const maxRetries = 3;
  for (let i = 0; i < maxRetries; i++) {
    try {
      const client = await pool.connect();
      try {
        await client.query('BEGIN ISOLATION LEVEL READ COMMITTED');
        const result = await client.query(
          'UPDATE invoices SET amount = $1 WHERE id = $2 RETURNING *',
          [data.amount, id]
        );
        await client.query('COMMIT');
        return result.rows[0];
      } finally {
        client.release();
      }
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      // Wait before retry
      await new Promise(r => setTimeout(r, 100 * Math.pow(2, i)));
    }
  }
}
```

**Step 2**: Break long transactions into batches
```typescript
// ❌ WRONG: One big transaction locks table
async importInvoices(invoices: any[]) {
  await pool.query('BEGIN');
  for (const invoice of invoices) {
    await pool.query('INSERT INTO invoices ...', [invoice]);
  }
  await pool.query('COMMIT');
}

// ✅ RIGHT: Batch commits, allows other transactions
async importInvoices(invoices: any[], batchSize = 100) {
  for (let i = 0; i < invoices.length; i += batchSize) {
    const batch = invoices.slice(i, i + batchSize);
    await pool.query('BEGIN');
    try {
      for (const invoice of batch) {
        await pool.query('INSERT INTO invoices ...', [invoice]);
      }
      await pool.query('COMMIT');
    } catch (error) {
      await pool.query('ROLLBACK');
      throw error;
    }
  }
}
```

---

## Problem 8: "Error message is useless - just 'Error: undefined' in logs"

### Symptoms
- Console logs show "Error: undefined"
- No stack trace
- Can't debug where error came from
- Error bubbles up to frontend as "500 Error"

### Root Cause
Errors caught but not properly formatted or logged.

### Solution

**Step 1**: Create standard error types
```typescript
// src/errors/AppError.ts
export class AppError extends Error {
  constructor(
    public code: string,
    public message: string,
    public statusCode: number = 500,
    public details?: any
  ) {
    super(message);
    Error.captureStackTrace(this, this.constructor);
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super('NOT_FOUND', `${resource} not found: ${id}`, 404);
  }
}

export class ValidationError extends AppError {
  constructor(field: string, reason: string) {
    super('VALIDATION_FAILED', `Validation failed: ${field}`, 400, { field, reason });
  }
}
```

**Step 2**: Log with full context
```typescript
// src/middleware/errorHandler.ts
export function errorHandler(err: any, req: Request, res: Response, next: NextFunction) {
  if (err instanceof AppError) {
    // Log structured error
    console.error({
      code: err.code,
      message: err.message,
      statusCode: err.statusCode,
      details: err.details,
      stack: err.stack,
      url: req.url,
      method: req.method,
      timestamp: new Date().toISOString(),
    });

    return res.status(err.statusCode).json({
      error: {
        code: err.code,
        message: err.message,
      },
    });
  }

  // Unknown error
  console.error({
    message: 'Unhandled error',
    error: err?.message || err,
    stack: err?.stack,
    url: req.url,
    timestamp: new Date().toISOString(),
  });

  res.status(500).json({
    error: {
      code: 'INTERNAL_SERVER_ERROR',
      message: 'An unexpected error occurred',
    },
  });
}
```

**Step 3**: Use everywhere
```typescript
// src/routes/invoices.ts
export async function getInvoice(req: Request, res: Response, next: NextFunction) {
  try {
    const invoice = await InvoiceService.get(req.params.id);
    if (!invoice) {
      throw new NotFoundError('Invoice', req.params.id);  // ← Clear error
    }
    res.json(invoice);
  } catch (error) {
    next(error);  // ← Passes to error handler
  }
}
```

✅ **Proper error handling:**
- Error messages are clear
- Stack traces preserved
- Logs are structured and searchable

---

## Quick Reference

| Problem | Check | Solution |
|---------|-------|----------|
| Field missing in response | Adapter layer | Update snakeToCamelCase function |
| Connection pool exhausted | Repositories | Ensure client.release() in finally block |
| Cascade deletes | FK constraints | Change to ON DELETE RESTRICT + use soft delete |
| Inconsistent naming | Transformation | Apply adapter only at API boundary |
| Breaking changes | API routes | Version routes (/api/v1, /api/v2) |
| External service failing | Circuit breaker | Implement and use CircuitBreaker pattern |
| Database locks | Transactions | Use batching + appropriate isolation levels |
| Unhelpful errors | Error handling | Use structured AppError classes + middleware |

---

**Still stuck?** Ask in code review with your error logs and code snippet.
