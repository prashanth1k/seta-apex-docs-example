# Apex Data Types and Variables

## Key Points

- Covers Primitives (Integer, String, Boolean, Date, Datetime, Decimal, etc.), sObjects, Collections (List, Set, Map), and Enums
- Lists are ordered, Sets are unique, Maps are key-value
- sObjects represent Salesforce records
- Handle null carefully to avoid NullPointerException

## Primitive Data Types

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

## Collections

### List

- Ordered collection. Duplicates allowed. `List<elementType> listName = new List<elementType>();`
- Example: `List<String> colors = new List<String>{'red', 'green'}; colors.add('blue'); String firstColor = colors[0];`
- Methods: `add()`, `get()`, `set()`, `clear()`, `size()`, `isEmpty()`, `sort()`.

### Set

- Unordered collection of unique primitives. `Set<primitiveType> setName = new Set<primitiveType>();`
- Example: `Set<Integer> uniqueNumbers = new Set<Integer>{1, 2, 3}; uniqueNumbers.add(1);`
- Cannot access by index. Iteration order is deterministic but not guaranteed.

### Map

- Key-value pairs. Unique keys. Keys must be primitives. `Map<keyType, valueType> mapName = new Map<keyType, valueType>();`
- Example: `Map<ID, Account> accountMap = new Map<ID, Account>(); Account acc = new Account(Name='Test'); insert acc; accountMap.put(acc.Id, acc); Account retrievedAcc = accountMap.get(acc.Id);`
- String keys are case-sensitive. `keySet()`, `values()`. Only specific key types are JSON serializable.

## sObjects

- Represents a Salesforce record.
- Specific: `Account`, `MyCustomObject__c`.
- Generic: `sObject`.
- Instantiation: `Account acc = new Account(Name='Test Inc.');` `sObject genericSObj = new Contact();`
- Field Access: Dot notation for specific sObjects; `get()` and `put()` for generic sObjects.
- Initialization: `Account a = new Account(Name='Acme', BillingCity='San Francisco');`

## Enums

- Fixed set of named constants. `public enum Season {WINTER, SPRING, SUMMER, FALL}`
- Usage: `Season currentSeason = Season.WINTER;`

## User-Defined Classes

Custom Apex classes.

## The `Object` Type

Base type for all Apex data types. Requires casting. `Object obj = 10; Integer i = (Integer)obj;`

## Null

Represents absence of a value. Check for `null` before use to avoid `NullPointerException`.

## Important Considerations

- Variable Naming: Case-insensitive. Max 255 characters.
- Case Sensitivity: Code and SOQL/SOSL are generally case-insensitive. Map keys are case-sensitive.
- Type Conversion: Explicit type conversion (casting) is generally required.
- Numeric Overflow/Underflow: Values wrap around. Use larger data types if necessary.
