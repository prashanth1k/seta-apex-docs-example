# SOQL Basics in Apex

## Key Points

- SOQL retrieves Salesforce records; use [...] syntax in Apex
- Key clauses: SELECT, FROM, WHERE, ORDER BY, LIMIT
- Use bind variables (:var) to prevent SOQL injection
- Query parent-to-child and child-to-parent relationships
- Crucial: Bulkify; avoid SOQL inside loops

## Quick Reference

```apex
// BASIC QUERY
List<Account> accounts = [SELECT Id, Name, Industry FROM Account WHERE Industry = 'Technology'];

// WITH BIND VARIABLES
String industry = 'Technology';
List<Account> accounts = [SELECT Id, Name FROM Account WHERE Industry = :industry];

// CHILD-TO-PARENT RELATIONSHIP
List<Contact> contacts = [SELECT Id, Name, Account.Name, Account.Industry FROM Contact];

// PARENT-TO-CHILD RELATIONSHIP
List<Account> accountsWithContacts = [
    SELECT Id, Name, (SELECT Id, Name, Email FROM Contacts)
    FROM Account
    WHERE Id IN :accountIds
];

// AGGREGATE QUERY
List<AggregateResult> results = [
    SELECT Industry, COUNT(Id) accountCount, AVG(AnnualRevenue) avgRevenue
    FROM Account
    GROUP BY Industry
];

// SOQL FOR LOOP (BULK PROCESSING)
for(List<Account> accountBatch : [SELECT Id, Name FROM Account WHERE CreatedDate = TODAY]) {
    // Process batch of up to 200 records
    for(Account acc : accountBatch) {
        // Process individual account
    }
}

// DYNAMIC SOQL
String query = 'SELECT Id, Name FROM Account WHERE Industry = :industry';
List<Account> accounts = Database.query(query);
```

## Core Concepts

- Retrieve records (sObjects) and fields from Salesforce objects.
- Embedded in Apex code using `[...]`.
- Return types: `List<sObject>`, single `sObject`, `Integer` (for `COUNT()`). Single `sObject` assignment requires exactly one record; otherwise, a runtime exception occurs.

## Basic Syntax

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

## Using SOQL in Apex

```apex
// Retrieve Accounts
List<Account> accounts = [SELECT Name, Phone FROM Account WHERE Industry = 'Technology'];

// Retrieve a single Account by ID
Account singleAcct = [SELECT Name FROM Account WHERE Id = :someAccountId LIMIT 1];

// Get Contact count
Integer contactCount = [SELECT COUNT() FROM Contact WHERE LastName = 'Smith'];
```

## Field Selection

- Explicitly list fields in `SELECT`.
- Accessing unselected fields (except `Id`) causes a runtime error.

```apex
Account acc = [SELECT Name FROM Account LIMIT 1];
// System.debug(acc.Phone); // Runtime error if Phone wasn't queried
```

## Filtering with `WHERE`

```apex
// Simple filter
List<Contact> smiths = [SELECT Name FROM Contact WHERE LastName = 'Smith'];

// Multiple conditions with AND
List<Opportunity> closedWonOpps = [SELECT Name FROM Opportunity WHERE StageName = 'Closed Won' AND Amount > 10000];

// Using OR
List<Lead> hotLeads = [SELECT Name FROM Lead WHERE Status = 'Working - Contacted' OR Rating = 'Hot'];
```

## Bind Variables

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

## Relationship Queries

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

## Aggregate Functions

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

## Semi-Joins and Anti-Joins (`IN`, `NOT IN`)

- **Semi-Join (`IN`):** Returns records with related records matching subquery criteria.

  ```apex
  List<Account> accountsWithOpps = [SELECT Id, Name FROM Account WHERE Id IN (SELECT AccountId FROM Opportunity)];
  ```

- **Anti-Join (`NOT IN`):** Returns records without related records matching subquery criteria.

  ```apex
  List<Account> accountsWithoutWonOpps = [SELECT Id, Name FROM Account WHERE Id NOT IN (SELECT AccountId FROM Opportunity WHERE StageName = 'Closed Won')];
  ```

## SOQL For Loops

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

## Governor Limits & Bulkification

- Synchronous Apex: 100 SOQL queries per transaction. Asynchronous Apex: 200.
- **Never** place SOQL queries inside loops.
- **Best Practice:** Collect IDs/criteria in a `Set` or `List`, perform one SOQL query outside the loop using `WHERE Id IN :myIdSet`, and use a `Map` for efficient lookup inside the loop.
