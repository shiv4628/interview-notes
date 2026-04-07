# String Null & Empty Checks in Java

## Core Difference

| Check | Meaning | Safe to Call? |
|-------|---------|---------------|
| `str == null` | Reference doesn't exist | ✅ Yes |
| `str.isEmpty()` | String exists but has 0 length | ❌ Only if not null |

## Quick Examples

```java
String str = null;
str == null        // ✅ true
str.isEmpty()      // ❌ NullPointerException

String str = "";
str == null        // false
str.isEmpty()      // ✅ true

String str = " ";
str.isEmpty()      // ❌ false (contains 1 space character)
```

## Safe Pattern

```java
if (str == null || str.isEmpty()) {
    return "";
}
```
✅ Safe because `||` short-circuits (won't call `isEmpty()` if null)

**Wrong way:** `str.isEmpty() || str == null` → Throws exception if null!

## Better Alternatives (Java 11+)

```java
str.isBlank()           // ✅ Handles null? No, still need null check
str == null || str.isBlank()  // ✅ Handles "", " ", "\t\n"
str == null || str.trim().isEmpty()  // ✅ Handles "", " ", spaces
```

## Key Takeaway

- `isEmpty()` only checks length
- Spaces are NOT considered empty by `isEmpty()`
- Always null-check FIRST to avoid `NullPointerException`
- Use `isBlank()` if you want whitespace-only strings treated as empty
