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
### J