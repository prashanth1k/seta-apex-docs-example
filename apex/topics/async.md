# Apex Async Methods

## Future method (@future)

- Asynchronous execution: Queued and executed later.
- Separate thread: Independent of initiating transaction.
- Use cases: Long-running operations, callouts from synchronous contexts, isolating DML operations.

## When to Use Future Methods

1. Callouts to external web services (requires `(callout=true)`).
2. Long-running operations.
3. Isolating DML operations.

## Syntax and Requirements

```apex
global class MyFutureExample {
    @future
    public static void myBasicFutureMethod(List<Id> recordIds) {
        List<Account> accounts = [SELECT Id, Name FROM Account WHERE Id IN :recordIds];
        // ... further processing ...
    }
    @future(callout=true)
    public static void myCalloutFutureMethod(String someParameter) {
        HttpRequest req = new HttpRequest();
        // ... setup request ...
        Http http = new Http();
        HttpResponse res = http.send(req);
        // ... process response ...
    }
}
```

- Annotation: `@future`
- Static: `static` keyword required.
- Return type: `void`.
- Parameters: Primitive types, arrays of primitives, collections of primitives. **Cannot** accept sObjects or complex objects.
- Common pattern: Pass `List<Id>`.

## Callouts

- Use `@future(callout=true)` for HTTP or web service callouts.
- Workaround for synchronous callout restrictions in triggers.

## Limitations and Considerations

- No Job ID.
- Parameter restrictions: Primitive types/collections only.
- No chaining.
- Execution order not guaranteed.
- Potential concurrency/locking issues.
- Cannot be called from other future methods or batch Apex `execute`/`finish` methods.
- Restricted methods: `getContent()`, `getContentAsPDF()`.
- Governor limits: 50 future calls per transaction; 24-hour org limit.

## Testing Future Methods

```apex
@isTest
static void testMyFutureMethod() {
    List<Id> accountIds = new List<Id>{testAccount.Id};

    Test.startTest();
    MyFutureExample.myBasicFutureMethod(accountIds);
    Test.stopTest();

    // 2. Verify results
}
```

- Enclose call in `Test.startTest()` and `Test.stopTest()`.
- Asynchronous calls are run synchronously.

## Alternatives

- **Queueable Apex:** More features, accepts non-primitive types, returns Job ID.
- **Batch Apex:** Large-scale record processing.
