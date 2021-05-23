# Effective Java

## 7장. 람다와 스트림

### 아이템42. 익명 클래스보다는 람다를 사용하라

```+java
# 기존 방식
Collections.sort(words, new Comparator<String>(){
    public int compare(String s1, String s2){
        return Integer.compare(s1.length(), s2.length());
    }
})

# 람다식 사용
Collections.sort(words, (s1, s2) -> 
                 Integer.compare(s1.length(), s2.length()));
                 
# 람다 자리에 비교자 생성 메서드를 사용하여 코드를 더 간결하게
Collections.sort(words, comparingInt(String::length));

# java 8 List 인터페이스에 추가된 sort 메서드를 이용하여 더 간결하게
words.sort(comparingInt(String::length));
```

**타입을 명시해야 코드가 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자.**
