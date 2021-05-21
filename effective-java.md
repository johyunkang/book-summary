# Effective Java

## 7장. 람다와 스트림

### 아이템42. 익명 클래스보다는 람다를 사용하라

```+java
Collections.sort(words, new Comparator<String>(){
    public int compare(String s1, String s2){
        return Integer.compare(s1.length(), s2.length());
    }
})
```
