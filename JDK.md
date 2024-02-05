```java
Stream.of(1, 2)
    .map(i -> i++)
    .map(Integer::toString)
    .toList(Collectors.toList())
```
