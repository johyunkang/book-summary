## Two Sum

### 문제

Given an array of integers `nums` and an integer `target`, return *indices of the two numbers such that they add up to `target`*.

You may assume that each input would have ***exactly\* one solution**, and you may not use the *same* element twice.

You can return the answer in any order.

 

**Example 1:**

```
Input: nums = [2,7,11,15], target = 9
Output: [0,1]
Output: Because nums[0] + nums[1] == 9, we return [0, 1].
```

**Example 2:**

```
Input: nums = [3,2,4], target = 6
Output: [1,2]
```

**Example 3:**

```
Input: nums = [3,3], target = 6
Output: [0,1]
```

 

**Constraints:**

- `2 <= nums.length <= 104`
- `-109 <= nums[i] <= 109`
- `-109 <= target <= 109`
- **Only one valid answer exists.**



------

### Solution

1. 풀었지만 느리다

   ```java
   public int[] twoSum_slow(int[] nums, int target) {
       int len = nums.length;
       for(int i=0; i<len-1; i++) {
           for(int j=i+1; j<len; j++) {
               if((nums[i]+nums[j])==target) {
                   return new int[] {i,j};
               }
           }
       }
       return new int[2];
   }
   ```

2. Hash 써서 빠르다

   ```java
   public int[] twoSum(int[] nums, int target) {
       HashMap<Integer, Integer> map = new HashMap<>();
       Integer rslt;
       int len = nums.length;
   
       for(int i=0; i<len; i++) {
           rslt = map.get(nums[i]);
           if(rslt != null)
               return new int[] {rslt, i};
           else
               map.put(target - nums[i], i);
       }
       return new int[2];
   }
   ```

   

