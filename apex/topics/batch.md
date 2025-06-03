# Batch Apex

## When to Use Batch Apex

- Processing large data volumes (thousands or millions of records).
- Operations requiring higher governor limits (e.g., SOQL, heap size, CPU time).
- Scheduled maintenance tasks (often initiated by Scheduled Apex).

## How Batch Apex Works

Implement the `Database.Batchable<sObject>` interface.

### The `Database.Batchable` Interface

- `start()`
- `execute()`
- `finish()`

### The `start` Method

```apex
global (Database.QueryLocator | Iterable<sObject>) start(Database.BatchableContext bc) {
    // Collect records/objects for processing
}
```

- Called once.
- Collects records for processing.
- Returns `Database.QueryLocator` or `Iterable<sObject>`.

### The `execute` Method

```apex
global void execute(Database.BatchableContext bc, List<sObject> scope) {
    // Process one batch of records
}
```

- Called for each batch.
- Processes a subset of records.
- Parameters: `Database.BatchableContext bc`, `List<sObject> scope`.
- Batch order is not guaranteed.

### The `finish` Method

```apex
global void finish(Database.BatchableContext bc) {
    // Execute post-processing operations
}
```

- Called once after all batches.
- Performs final tasks.

### Transaction Boundaries

- Each `execute` call is a discrete transaction.
- Governor limits reset for each `execute` transaction.
- One batch failure doesn't rollback others.

## Defining the Scope: `Database.QueryLocator` vs. `Iterable`

### `Database.QueryLocator`

- Use for simple SOQL queries.
- Bypasses 50,000 SOQL record limit (up to 50 million records).
- Use Case: Straightforward SOQL query on a single sObject.

### `Iterable<sObject>`

- Use for complex scopes (pre-processing, API calls, complex logic).
- Standard SOQL record limit applies.
- Requires implementing `Iterator` or returning `List<sObject>`.

## Maintaining State with `Database.Stateful`

- Batch Apex classes are stateless by default.
- Implement `Database.Stateful` to maintain state across transactions.

```apex
global class MyStatefulBatch implements Database.Batchable<sObject>, Database.Stateful {
    global Integer recordsProcessed = 0;

    // ... start, execute, finish methods ...
}
```

- Only instance variables retain values.
- Use Case: Counting records, summarizing data, collecting errors.

## Invoking Batch Apex

1. Instantiate your Batch Apex class: `MyBatchableClass myBatchObject = new MyBatchableClass();`
2. Call `Database.executeBatch`: `Id batchId = Database.executeBatch(myBatchObject);`
3. Optional `scope` parameter (specifies batch size): `Id batchId = Database.executeBatch(myBatchObject, 100);`

## Monitoring Batch Apex

- `Database.executeBatch` returns the Job ID.
- Monitor job status via SOQL:

```apex
AsyncApexJob job = [SELECT Id, Status, JobItemsProcessed, TotalJobItems, NumberOfErrors
                    FROM AsyncApexJob WHERE Id = :batchId];
```

- Monitor via Salesforce UI: **Setup** > **Environments** > **Jobs** > **Apex Jobs**.

## Testing Batch Apex

- Use `Test.startTest()` and `Test.stopTest()` to run synchronously.

```apex
@isTest static void testMyBatch() {
    Test.startTest();
    Id batchId = Database.executeBatch(myBatch, 10);
    Test.stopTest();
    // Assertions
}
```

- `execute` method runs once.
- Use `scope` parameter to control records in testing.

## Governor Limits

**Asynchronous Limits:**

- Total Heap Size: 12 MB
- Maximum CPU Time: 60,000 ms
- SOQL Queries: 200 (per `execute`)
- DML Statements: 150 (per `execute`)
- DML Rows: 10,000 (per `execute`)
- Callouts: 100 (per `execute`)

**Batch Job Specific Limits:**

- Concurrent Jobs: 5
- Apex Flex Queue: 100 (Holding status)
- QueryLocator Record Limit: 50 million
- Test Limits: 5 batch jobs per test run

## Best Practices

- Use Batch Apex only when necessary.
- Optimize SOQL queries in `start`.
- Minimize callouts and optimize queries in `execute`.
- Use `Database.Stateful` sparingly.
- Avoid invoking from triggers if possible.
- Test thoroughly.

## Examples

- `MyBatchableClass.cls`
- `AccountContactDifferentOwnerBatch.cls`
- `Using_DatabaseQueryLocator_to_define_scope.cls` (contains `SearchAndReplace` class)
- `UpdateContactAddresses.cls`
