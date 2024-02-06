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
- Interface Private Method
### Java 10
- Lcal Variable `var`
### Java 11
### Java 12
### Java 13
- ZGC
### Java 14
- Switch Expression, `ye
### Java 15
- Multi Line String
### Java 16
- Type Pattern Matching
- Record
### Java 17
- Sealed Class (permits)
### Java 18
- HttpServer : create(port), createContext(path, sevlet), start()
### Java 19
### Java 20
### Java 21
- Virtual Thread
- Record Pattern Matching
- SequencedCollection
- String Template