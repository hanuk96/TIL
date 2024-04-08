# 반복되는 Switch문
- 반복해서 등장하는 동일한 switch문을 냄새로 여기고 있다. 
- 반복해서 동일한 switch문이 존재할 경우, 새로운 조건을 추가하거나 기존의 조건을 변경할 때모든 switch 문을 찾아서 코드를 고쳐야 할지도 모른다.

before

```java
public int vacationHours(String type) {
    int result;
    switch (type) {
        case "full-time": result = 120; break;
        case "part-time": result = 80; break;
        case "temporal": result = 32; break;
        default: result = 0;
    }
    return result;
}
```

after - switch expression을 통해 깔끔하게 switch문을 변경한 모습

```java
public int vacationHours(String type) {
    int result = switch (type) {
        case "full-time" -> 120;
        case "part-time" ->  80;
        case "temporal" -> 32;
        default -> 0;
    };
    return result;
}
```
