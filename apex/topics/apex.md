# Apex: Key Features and Code Examples

## How Apex Works

1.  **Development:** Write Apex code (classes, triggers).
2.  **Compilation:** Salesforce compiles code to bytecode.
3.  **Execution:** Server retrieves bytecode; runtime interpreter executes.
4.  **Results:** Results returned to user/process.

## Why Use Apex?

Use Apex for complex logic beyond declarative tools:

- Web services (SOAP, REST)
- Email services
- Complex validation across objects
- Complex business processes
- Custom transactional logic
- Attaching logic to record saves (UI, API, etc.)

## Key Characteristics

- Integrated: DML, SOQL, SOSL support.
- Data-focused: Efficiently queries/manipulates large datasets.
- Rigorous: Strongly typed; compile-time checks.
- Hosted: Interpreted and managed by Salesforce.
- Multitenant-aware: Governor limits prevent resource monopolization.
- Easy to use: Java-like syntax.
- Easy to test: Built-in unit testing support.
- Versioned: Maintained behavior across platform upgrades.

## Core Concepts

- **Variables & Data Types:** Primitives (Integer, Boolean, String), sObjects, Collections, Enums, Custom Classes.
- **Statements:** Assignments, `if-else`, loops (`for`, `while`, `do-while`), DML, transaction control, `try-catch-finally`.
- **Collections:** Lists, Sets, Maps.
- **Classes, Objects, Interfaces:** Classes define blueprints; objects are instances; interfaces define contracts. Supports inheritance and polymorphism.
- **Triggers:** Code invoked before/after DML events (insert, update, delete, undelete) on sObjects.
- **SOQL & SOSL:** Integrated query languages.
- **DML:** Integrated language for record manipulation (insert, update, upsert, delete, undelete).

## Development Environment

- **Salesforce Extensions for Visual Studio Code:** Recommended.
- **Code Builder:** Browser-based VS Code.
- **Developer Console:** Built-in browser-based IDE.
- **Setup Code Editors:** Basic editors in Salesforce Setup.

## Testing

- Unit tests: Minimum 75% code coverage.
- All tests must pass.
- Verify logic, positive/negative cases, bulk processing.
- Test methods use `@isTest` annotation; don't commit data.

## Governor Limits

Salesforce enforces governor limits to ensure fair resource usage. Avoid exceeding limits by writing efficient, bulkified code. Limits include SOQL queries, DML statements, CPU time, and heap size.

```
*For detailed syntax, methods, and advanced features, refer to the official [Apex Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_dev_guide.htm).*
```

## Apex Data Types and Variables

### Primitive Data Types

| Data Type    | Description                                                                                                                      | Example                                                          |
| :----------- | :------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------- |
| **Blob**     | Binary data. Convert to/from `String` using `toString()` and `valueOf()`.                                                        | `Blob myBlob = Blob.valueOf('My data');`                         |
| **Boolean**  | `true`, `false`, or `null`. Initialized to `null` if not assigned.                                                               | `Boolean isWinner = true;` `Boolean flag = null;`                |
| **Date**     | Day without time. Use `Date.today()`. Add/subtract days. Use `String.valueOf()` for string conversion.                           | `Date today = Date.today();` `Date nextWeek = today.addDays(7);` |
| **Datetime** | Day and time. Use `Datetime.now()`. Add/subtract days/fractions of days.                                                         | `Datetime now = Datetime.now();`                                 |
| **Decimal**  | Arbitrary-precision number. Currency fields. Use `setScale()` to set decimal places.                                             | `Decimal myDecimal = 123.456;`                                   |
| **Double**   | 64-bit number. Larger range than `Integer`. +/- 2^63.                                                                            | `Double pi = 3.14159;` `Double d = 5.0 / 3.0; // 1.66...`        |
| **ID**       | 18-character record identifier. Converts 15-char IDs to 18-char.                                                                 | `ID myId = '001D000000Jsm0WIAR';`                                |
| **Integer**  | 32-bit number. -2,147,483,648 to 2,147,483,647.                                                                                  | `Integer i = 10;`                                                |
| **Long**     | 64-bit number. Wider range than `Integer`. -2^63 to 2^63-1. Append 'L' to literals.                                              | `Long l = 2147483648L;`                                          |
| **Object**   | Any Apex data type. Use for generic handling. Requires casting.                                                                  | `Object obj = 10; Integer i = (Integer)obj;`                     |
| **String**   | Set of characters. Supports escape sequences. Case-insensitive comparison. Use `escapeSingleQuotes()` to prevent SOQL injection. | `String name = 'My Company';`                                    |
| **Time**     | Time. Use `Time.newInstance()`.                                                                                                  | `Time t = Time.newInstance(18, 30, 2, 20);`                      |

**Note:** `AnyType` and `Currency` are primarily used in system methods.

### Collections

#### List

- Ordered collection. Duplicates allowed. `List<elementType> listName = new List<elementType>();`
- Example: `List<String> colors = new List<String>{'red', 'green'}; colors.add('blue'); String firstColor = colors[0];`
- Methods: `add()`, `get()`, `set()`, `clear()`, `size()`, `isEmpty()`, `sort()`.

#### Set

- Unordered collection of unique primitives. `Set<primitiveType> setName = new Set<primitiveType>();`
- Example: `Set<Integer> uniqueNumbers = new Set<Integer>{1, 2, 3}; uniqueNumbers.add(1);`
- Cannot access by index. Iteration order is deterministic but not guaranteed.

#### Map

- Key-value pairs. Unique keys. Keys must be primitives. `Map<keyType, valueType> mapName = new Map<keyType, valueType>();`
- Example: `Map<ID, Account> accountMap = new Map<ID, Account>(); Account acc = new Account(Name='Test'); insert acc; accountMap.put(acc.Id, acc); Account retrievedAcc = accountMap.get(acc.Id);`
- String keys are case-sensitive. `keySet()`, `values()`. Only specific key types are JSON serializable.

#### sObjects

- Represents a Salesforce record.
- Specific: `Account`, `MyCustomObject__c`.
- Generic: `sObject`.
- Instantiation: `Account acc = new Account(Name='Test Inc.');` `sObject genericSObj = new Contact();`
- Field Access: Dot notation for specific sObjects; `get()` and `put()` for generic sObjects.
- Initialization: `Account a = new Account(Name='Acme', BillingCity='San Francisco');`

#### Enums

- Fixed set of named constants. `public enum Season {WINTER, SPRING, SUMMER, FALL}`
- Usage: `Season currentSeason = Season.WINTER;`

### User-Defined Classes

Custom Apex classes.

#### The `Object` Type

Base type for all Apex data types. Requires casting. `Object obj = 10; Integer i = (Integer)obj;`

#### Null

Represents absence of a value. Check for `null` before use to avoid `NullPointerException`.

### Important Considerations

- Variable Naming: Case-insensitive. Max 255 characters.
- Case Sensitivity: Code and SOQL/SOSL are generally case-insensitive. Map keys are case-sensitive.
- Type Conversion: Explicit type conversion (casting) is generally required.
- Numeric Overflow/Underflow: Values wrap around. Use larger data types if necessary.

## Apex Data Manipulation Language (DML) Operations

### Core DML Operations

- `insert`: Adds one or more new records.
- `update`: Modifies one or more existing records.
- `upsert`: Creates new records or updates existing ones based on a specified field.
- `delete`: Moves one or more records to the Recycle Bin.
- `undelete`: Restores one or more records from the Recycle Bin.
- `merge`: Merges up to three records into one, deleting others and re-parenting related records.

### Bulk Operations (Bulkification)

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

### Atomic Transactions

- DML statements execute within an atomic transaction.
- **All or Nothing:** An error during processing rolls back the entire operation.
- **Partial Success:** Use `Database` class methods for partial success.

### Execution Context and Security

- DML runs in system context; user permissions and field-level security are bypassed (generally).
- **Sharing Rules:** Record-level sharing rules are respected based on class sharing keywords (`with sharing`, `without sharing`, `inherited sharing`).
- **Enforcing User Permissions:** Use `WITH USER_MODE` or `WITH SECURITY_ENFORCED` in SOQL, or `Security.stripInaccessible()` before DML.

### Exceptions

- Failed DML statements throw a `DmlException`. Use `try-catch` blocks.

#### DML Statements vs. Database Class Methods

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

### Common DML Operations Examples

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

### Important Considerations

- **Governor Limits:** Be mindful of DML statement limits (150 per transaction) and row limits (10,000 per transaction). Bulkify your code.
- **Setup vs. Non-Setup Objects:** Cannot mix DML operations on setup and non-setup objects in the same transaction. Use asynchronous operations if necessary.
- **Related Records:** Updating related records requires separate DML statements.

## SOQL Basics in Apex

### Core Concepts

- Retrieve records (sObjects) and fields from Salesforce objects.
- Embedded in Apex code using `[...]`.
- Return types: `List<sObject>`, single `sObject`, `Integer` (for `COUNT()`). Single `sObject` assignment requires exactly one record; otherwise, a runtime exception occurs.

### Basic Syntax

```sql
SELECT Field1, Field2, ...
FROM ObjectName
[WHERE Condition]
[ORDER BY FieldToSort]
[LIMIT NumberOfRecords]
```

- `SELECT`: Specifies fields. `Id` is implicit unless excluded.
- `FROM`: Specifies the primary sObject.
- `WHERE`: (Optional) Filters records. Uses comparison (`=`, `!=`, `>`, `<`, `>=`, `<=`, `LIKE`) and logical (`AND`, `OR`, `NOT`) operators.
- `ORDER BY`: (Optional) Sorts records (ascending (`ASC`) or descending (`DESC`)).
- `LIMIT`: (Optional) Limits the number of returned records.

### Using SOQL in Apex

```apex
// Retrieve Accounts
List<Account> accounts = [SELECT Name, Phone FROM Account WHERE Industry = 'Technology'];

// Retrieve a single Account by ID
Account singleAcct = [SELECT Name FROM Account WHERE Id = :someAccountId LIMIT 1];

// Get Contact count
Integer contactCount = [SELECT COUNT() FROM Contact WHERE LastName = 'Smith'];
```

### Field Selection

- Explicitly list fields in `SELECT`.
- Accessing unselected fields (except `Id`) causes a runtime error.

```apex
Account acc = [SELECT Name FROM Account LIMIT 1];
// System.debug(acc.Phone); // Runtime error if Phone wasn't queried
```

### Filtering with `WHERE`

```apex
// Simple filter
List<Contact> smiths = [SELECT Name FROM Contact WHERE LastName = 'Smith'];

// Multiple conditions with AND
List<Opportunity> closedWonOpps = [SELECT Name FROM Opportunity WHERE StageName = 'Closed Won' AND Amount > 10000];

// Using OR
List<Lead> hotLeads = [SELECT Name FROM Lead WHERE Status = 'Working - Contacted' OR Rating = 'Hot'];
```

### Bind Variables

- Use Apex variables prefixed with `:` (bind variables).
- Prevents SOQL injection.
- Apex parser evaluates the variable before SOQL execution.

```apex
String targetIndustry = 'Media';
List<Account> mediaAccounts = [SELECT Id, Name FROM Account WHERE Industry = :targetIndustry];

Set<Id> accountIds = new Map<Id, Account>(Trigger.newMap).keySet();
List<Contact> relatedContacts = [SELECT Name FROM Contact WHERE AccountId IN :accountIds];
```

- Use in `WHERE`, `LIMIT`, `OFFSET`, `FIND` (SOSL).
- Cannot use bind variable fields directly; resolve to simple variables first.

### Relationship Queries

- **Child-to-Parent:** Use dot notation. For custom relationships, append `__r`.

  ```apex
  List<Contact> contacts = [SELECT Name, Account.Name FROM Contact WHERE Account.Industry = 'Technology'];
  ```

- **Parent-to-Child:** Use a nested `SELECT` subquery. Use the plural relationship name. For custom relationships, append `__r`.

  ```apex
  List<Account> accountsWithContacts = [SELECT Name, (SELECT LastName FROM Contacts) FROM Account WHERE Name = 'Acme Inc.'];
  for(Account acc : accountsWithContacts) {
      List<Contact> childContacts = acc.Contacts;
      // ... process childContacts ...
  }
  ```

### Aggregate Functions

SOQL supports `COUNT()`, `COUNT(fieldName)`, `SUM(fieldName)`, `AVG()`, `MIN()`, `MAX()`.

- Returns `List<AggregateResult>`.
- Access values using `get('alias')` or `expr0`, `expr1`, etc.
- `COUNT()` returns an `Integer`.

```apex
// Total Accounts
Integer totalAccounts = [SELECT COUNT() FROM Account];

// Average amount per StageName
List<AggregateResult> results = [SELECT StageName, AVG(Amount) avgAmount FROM Opportunity GROUP BY StageName];
for(AggregateResult ar : results) {
    System.debug('Stage: ' + ar.get('StageName') + ', Avg Amount: ' + ar.get('avgAmount'));
}
```

### Semi-Joins and Anti-Joins (`IN`, `NOT IN`)

- **Semi-Join (`IN`):** Returns records with related records matching subquery criteria.

  ```apex
  List<Account> accountsWithOpps = [SELECT Id, Name FROM Account WHERE Id IN (SELECT AccountId FROM Opportunity)];
  ```

- **Anti-Join (`NOT IN`):** Returns records without related records matching subquery criteria.

  ```apex
  List<Account> accountsWithoutWonOpps = [SELECT Id, Name FROM Account WHERE Id NOT IN (SELECT AccountId FROM Opportunity WHERE StageName = 'Closed Won')];
  ```

### SOQL For Loops

- Processes large results to avoid heap size limits.
- Retrieves records in batches using `query()` and `queryMore()`.

- **Iterate over single sObjects:**

  ```apex
  Integer count = 0;
  for (Account acc : [SELECT Name FROM Account WHERE Industry = 'Energy']) {
      count++;
  }
  ```

- **Iterate over lists of sObjects:**

  ```apex
  List<Account> accountsToUpdate = new List<Account>();
  for (List<Account> accountBatch : [SELECT Name, Description FROM Account WHERE IsActive__c = true]) {
      for (Account acc : accountBatch) {
          acc.Description = 'Processing batch update';
          accountsToUpdate.add(acc);
      }
  }
  // update accountsToUpdate; // Often better outside the loop or using Batch Apex
  ```

### Governor Limits & Bulkification

- Synchronous Apex: 100 SOQL queries per transaction. Asynchronous Apex: 200.
- **Never** place SOQL queries inside loops.
- **Best Practice:** Collect IDs/criteria in a `Set` or `List`, perform one SOQL query outside the loop using `WHERE Id IN :myIdSet`, and use a `Map` for efficient lookup inside the loop.
