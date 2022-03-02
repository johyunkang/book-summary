## 가장 큰 수

###### 문제 설명

0 또는 양의 정수가 주어졌을 때, 정수를 이어 붙여 만들 수 있는 가장 큰 수를 알아내 주세요.

예를 들어, 주어진 정수가 [6, 10, 2]라면 [6102, 6210, 1062, 1026, 2610, 2106]를 만들 수 있고, 이중 가장 큰 수는 6210입니다.

0 또는 양의 정수가 담긴 배열 numbers가 매개변수로 주어질 때, 순서를 재배치하여 만들 수 있는 가장 큰 수를 문자열로 바꾸어 return 하도록 solution 함수를 작성해주세요.

##### 제한 사항

- numbers의 길이는 1 이상 100,000 이하입니다.
- numbers의 원소는 0 이상 1,000 이하입니다.
- 정답이 너무 클 수 있으니 문자열로 바꾸어 return 합니다.

##### 입출력 예

| numbers           | return    |
| ----------------- | --------- |
| [6, 10, 2]        | "6210"    |
| [3, 30, 34, 5, 9] | "9534330" |



## 제출 코드

- comparator 부분 추가 공부

```java
import java.util.Arrays;
import java.util.Comparator;
class Solution {
	public String biggestNum(int[] numbers) {
		String answer = "";
		
		if(numbers == null || numbers.length == 0)
			return answer;
		
		int loopSize = numbers.length;
		String[] strArr = new String[loopSize];
		
		for(int iLoop=0; iLoop<loopSize; iLoop++) {
			strArr[iLoop] = String.valueOf(numbers[iLoop]);
		}
		
		// 단순 Arrays.sort(strArr, Collections.reverseOrder()) 하면 
		// 샘플 {30, 3} 에서 303 으로 리턴 됨. 의도는 330
		// comparator 이용하여 원하는 내림차순 방법으로 정렬되게 오버라이딩
		Arrays.sort(strArr, new Comparator<String>() {
			public int compare(String a, String b) {
				return (b+a).compareTo(a+b);
			}
		});
		
		// 숫자 제일 처음이 0 이면 000과 같은 경우 0만 리턴하게 설정
		if("0".equals(strArr[0])) {
			answer = "0";
			return answer;
		}
		
		for(String s : strArr)
			answer +=s;
		
		return answer;
	}
}
```

