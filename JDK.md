```java
Stream.of(1, 2)
    .map(i -> i++)
    .map(Integer::toString)
    .toList(Collectors.toList())
```
## Summary
### Java 8
- Lambda, @FuntionalInterface, Method Reference
- Stream, Optional, Collectors, parallel()
- DateTime
- Interface Default Method
### Java 9
- Collection Factory
- HttpClient
### Java 10
- Lcal Variable `var`
### Java 11
### Java 12
### Java 13
- ZGC
### Java 14
- Switch Expression
### Java 15
### Java 16
- Type Pattern Mattching
### Java 17
- Sealed Class (permits)