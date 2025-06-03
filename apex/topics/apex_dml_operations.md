# Apex Data Manipulation Language (DML) Operations

## Key Points

- Core operations: insert, update, upsert, delete, undelete, merge
- Crucial: Bulkify by operating on lists to avoid governor limits
- Use Database class methods for partial success
- Handle DmlException

## Quick Reference

```apex
// BULK INSERT
List<Account> accounts = new List<Account>();
for(Integer i = 0; i < 200; i++) {
    accounts.add(new Account(Name = 'Account ' + i));
}
insert accounts;

// BULK UPDATE WITH SOQL
List<Account> accountsToUpdate = [SELECT Id, Name FROM Account WHERE Industry = 'Technology'];
for(Account acc : accountsToUpdate) {
    acc.Description = 'Updated via bulk operation';
}
update accountsToUpdate;

// UPSERT WITH EXTERNAL ID
List<Account> accountsToUpsert = new List<Account>();
accountsToUpsert.add(new Account(Name='Acme Corp', External_Id__c='EXT001'));
upsert accountsToUpsert Account.External_Id__c;

// DATABASE CLASS WITH PARTIAL SUCCESS
Database.SaveResult[] results = Database.insert(accounts, false);
for(Database.SaveResult sr : results) {
    if(!sr.isSuccess()) {
        for(Database.Error err : sr.getErrors()) {
            System.debug('Error: ' + err.getMessage());
        }
    }
}
```

## Core DML Operations

- `insert`: Adds one or more new records.
- `update`: Modifies one or more existing records.
- `upsert`: Creates new records or updates existing ones based on a specified field.
- `delete`: Moves one or more records to the Recycle Bin.
- `undelete`: Restores one or more records from the Recycle Bin.
- `merge`: Merges up to three records into one, deleting others and re-parenting related records.

## Bulk Operations (Bulkification)

- **Best Practice:** Perform DML on lists of sObjects, not single sObjects in loops.
- **Reason:** Salesforce governor limits (e.g., 150 DML statements per transaction). A list counts as one DML statement (up to 10,000 rows).
- **Bad Example (Avoid):**
  ```apex
  for(Account acc : accountsToUpdate) {
      update acc;
  }
  ```
- **Good Example (Recommended):**
  ```apex
  List<Account> accountsToUpdate = new List<Account>();
  for(Account acc : listOfAllAccounts) {
      if(acc.Industry == 'Technology') {
          acc.Description = 'Updated';
          accountsToUpdate.add(acc);
      }
  }
  update accountsToUpdate;
  ```

## Atomic Transactions

- DML statements execute within an atomic transaction.
- **All or Nothing:** An error during processing rolls back the entire operation.
- **Partial Success:** Use `Database` class methods for partial success.

## Execution Context and Security

- DML runs in system context; user permissions and field-level security are bypassed (generally).
- **Sharing Rules:** Record-level sharing rules are respected based on class sharing keywords (`with sharing`, `without sharing`, `inherited sharing`).
- **Enforcing User Permissions:** Use `WITH USER_MODE` or `WITH SECURITY_ENFORCED` in SOQL, or `Security.stripInaccessible()` before DML.

## Exceptions

- Failed DML statements throw a `DmlException`. Use `try-catch` blocks.

## DML Statements vs. Database Class Methods

1.  **DML Statements:** (`insert`, `update`, `delete`)

    - Straightforward syntax.
    - Throw `DmlException` on the first error, rolling back the transaction.
    - Use `try-catch` blocks.

2.  **`Database` Class Methods:** (`Database.insert()`, `Database.update()`)
    - More flexible error handling.
    - Support partial success (set `allOrNone` to `false`).
    ```apex
    Database.SaveResult[] results = Database.insert(recordsToInsert, false);
    ```
    - Return arrays of result objects; check success/failure individually.
    - Include methods not available as DML statements.

**When to Use Which:**

- **DML statements:** All-or-nothing operations, standard `try-catch`.
- **`Database` class methods:** Partial success, inspect individual results.

## Common DML Operations Examples

```apex
// --- Insert ---
List<Account> newAccounts = new List<Account>();
newAccounts.add(new Account(Name='Acme Corp'));
newAccounts.add(new Account(Name='Global Corp'));
insert newAccounts;

// --- Update ---
List<Account> accountsToUpdate = [SELECT Id, Description FROM Account WHERE Name LIKE 'Acme%'];
for(Account acc : accountsToUpdate) {
    acc.Description = 'Updated Description';
}
update accountsToUpdate;

// --- Upsert (using External ID) ---
List<Account> accountsToUpsert = new List<Account>();
accountsToUpsert.add(new Account(Name='Existing Acme', My_External_ID__c='A1'));
accountsToUpsert.add(new Account(Name='New Beta', My_External_ID__c='B2'));
upsert accountsToUpsert Account.My_External_ID__c;

// --- Delete ---
List<Account> accountsToDelete = [SELECT Id FROM Account WHERE Name LIKE 'Test%'];
delete accountsToDelete;
```

## Important Considerations

- **Governor Limits:** Be mindful of DML statement limits (150 per transaction) and row limits (10,000 per transaction). Bulkify your code.
- **Setup vs. Non-Setup Objects:** Cannot mix DML operations on setup and non-setup objects in the same transaction. Use asynchronous operations if necessary.
- **Related Records:** Updating related records requires separate DML statements.

## Common Errors and Solutions

### DML Inside Loops (Governor Limit)

```apex
// ❌ WRONG - DML inside loop
for(Account acc : accounts) {
    update acc; // Each update counts as separate DML statement
}

// ✅ CORRECT - Bulk DML
List<Account> accountsToUpdate = new List<Account>();
for(Account acc : accounts) {
    if(acc.Industry == 'Technology') {
        acc.Description = 'Updated';
        accountsToUpdate.add(acc);
    }
}
if(!accountsToUpdate.isEmpty()) {
    update accountsToUpdate;
}
```

### Mixed DML Operations

```apex
// ❌ WRONG - Mixing setup and non-setup objects
insert new Account(Name='Test');
insert new User(Username='test@example.com'); // DML Exception

// ✅ CORRECT - Separate transactions
insert new Account(Name='Test');
System.runAs(new User(Id=UserInfo.getUserId())) {
    // Or use @future method for User DML
}
```

### Null Pointer Exceptions

```apex
// ❌ WRONG - Not checking for null
Account acc = [SELECT Id FROM Account LIMIT 1];
acc.Name = 'Updated'; // Could be null if no records

// ✅ CORRECT - Null checking
List<Account> accounts = [SELECT Id, Name FROM Account LIMIT 1];
if(!accounts.isEmpty()) {
    accounts[0].Name = 'Updated';
    update accounts[0];
}
```
