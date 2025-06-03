# Apex: Core Concepts and Overview

## Key Points

- Apex is strongly-typed, data-focused, and runs on Salesforce servers
- Used for complex business logic, web services, and custom transaction control
- Features include DML, SOQL, Triggers, Classes, and integrated testing
- Aware of multitenancy via governor limits
- Java-like syntax with built-in Salesforce integration

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

## Governor Limits Overview

Salesforce enforces governor limits to ensure fair resource usage. Avoid exceeding limits by writing efficient, bulkified code. Limits include SOQL queries, DML statements, CPU time, and heap size.

_For detailed syntax, methods, and advanced features, refer to the official [Apex Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_dev_guide.htm)._

## Common Patterns Index

**Data Operations:**

- Bulk insert/update → See `apex dml` topic
- SOQL queries → See `apex soql` topic
- Record relationships → See `apex soql` topic

**Automation:**

- Trigger patterns → See `trigger` topic
- Scheduled jobs → See `async` topic
- Batch processing → See `batch` topic

**Error Handling:**

- Try-catch blocks → See `apex dml` topic
- Governor limit avoidance → See `apex dml`, `apex soql` topics

**Integration:**

- REST/SOAP callouts → See `async` topic
- Platform events → See `async` topic
