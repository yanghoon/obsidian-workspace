```java
Stream.of(1, 2)
    .map(Integer::toString)
    .toList(Collectors.toList())
```
