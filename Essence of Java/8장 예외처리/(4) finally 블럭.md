
`finally` 블럭은 예외 발생여부와 상관없이 실행되어야할 코드를 포함시킬 목적으로 사용하게 된다. `try-catch-finally` 순서로 구성된다.

```
try {  
    System.out.printf("hello %s \n", name);  
} catch (Exception e) {  
    throw new RuntimeException(e);  
} finally {  
    System.out.println("welcome jx world");  
}
```


예외가 발생하는 경우에는 `try -> catch -> finally` 순으로 실행되고 예외가 발생하지 않은 경우에는 `try -> finally` 순으로 실행된다.

*`try` 블록에 `return` 이 있을 때는 어떻게 동작할까?*

```
try {  
    System.out.printf("hello %s \n", name);  
    return // 리턴 추가
} catch (Exception e) {  
    throw new RuntimeException(e);  
} finally {  
    System.out.println("welcome jx world");  
}
```

위와 같이 `try` 블럭에 `return` 문이 실행되는 경우에도 `finally` 블럭의 문장이 먼저 실행되고 이후에 현재 실행 중인 메서드를 종료한다.

