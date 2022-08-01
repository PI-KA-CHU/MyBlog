author = "pikachu"
title = "leetcode hot100"
date = "2022-07-28"
description = " leetcode100"
tags = [
]
categories = [
    "it","算法"
]

---





#### 3. 无重复字符的最长子串

- 算法：滑动窗口法
  - 窗口存放[left, right)中字符的数量

```
public int lengthOfLongestSubstring(String s) {

    int left = 0;
    int right = 0;
    Map<Character, Integer> windows = new HashMap<>();
    int maxVal = 0;

    while(right < s.length()){
        char c = s.charAt(right);
        windows.put(c, windows.getOrDefault(c, 0) + 1);
        right ++;

        while(windows.get(c) > 1){
            char d = s.charAt(left);
            windows.put(d, windows.get(d) - 1);
            left ++;
        }
        maxVal = Integer.max(maxVal, right - left);
    }

    return maxVal;
}
```

- 优化版
  - left不需要一个一个滑动，map存放的是下标，但是容易出错

```
public int lengthOfLongestSubstring(String s) {

    int left = 0;
    int right = 0;
    Map<Character, Integer> windows = new HashMap<>();
    int maxVal = 0;

    while(right < s.length()){
        char c = s.charAt(right);

        if(windows.containsKey(c) && windows.get(c) >= left){
            maxVal = Integer.max(maxVal, right - left);
            left = windows.get(c) + 1;
        }

        windows.put(c, right);
        right ++;
    }
    maxVal = Integer.max(maxVal, right - left);

    return maxVal;
}
```



#### 5. >最长回文子串

- 解法：中心扩列法
- 注意：substring需要的是左右的下标

```
public String longestPalindrome(String s) {

    int left = 0;
    int maxLen = 1;

    for(int i = 0;i < s.length();i ++){
        int len1 = subLongestPalindrome(s, i , i);
        int len2 = subLongestPalindrome(s, i , i + 1);
        int subMax = Integer.max(len1, len2);

        if(subMax > maxLen){
            left = i - (subMax - 1) / 2;
            maxLen = subMax;
        }
    }

    return s.substring(left, left + maxLen);
}

private int subLongestPalindrome(String s, int left, int right){
    
    int maxLen = 0;
    while(left >= 0 && right < s.length()){
        if(s.charAt(left) == s.charAt(right)){
            maxLen = Integer.max(maxLen, right - left + 1);
            left --;
            right ++;
        }else {
            return maxLen;
        }
    }

    return maxLen;
}
```



#### 11. >盛最多水的容器

- 双指针

```
public int maxArea(int[] height) {

    int max = 0;
    int left = 0;
    int right = height.length - 1;

    while(left < right){
        max = Integer.max(max, Integer.min(height[left], height[right]) * (right - left));

        if(height[left] < height[right]){
            left ++;
        }else {
            right --;
        }
    }

    return max;
}
```



#### 15. >三数之和

- 三指针

```
public List<List<Integer>> threeSum(int[] nums) {

    Arrays.sort(nums);

    List<List<Integer>> ans = new ArrayList<>();
    for(int i = 0;i < nums.length;i ++){

        int left = i + 1;
        int right = nums.length - 1;

        // 去重
        if(i > 0 && nums[i] == nums[i - 1]){
            continue;
        }

        while(left < right){
            int sum = nums[left] + nums[right] + nums[i];
            if(sum == 0){
                ans.add(Arrays.asList(nums[left], nums[right], nums[i]));
                left ++;
                right --;

                // 去重
                while(left < right && nums[left] == nums[left - 1]) left ++;
                while(left < right && nums[right] == nums[right + 1]) right --;
            }else if(sum < 0){
                left ++;
            }else {
                right --;
            }
        }
    }

    return ans;
}
```



#### 17. >电话号码的字母组合

- 算法：回溯法

> 组合和排列一般都会用到回溯法，排列和组合的区别：组合不考虑顺序，例如对[A,B]选两个数进行排列组合，[A,B]即组合成一组，而排列考虑顺序，[A,B]、[B,A]为不同排列。

```
public List<String> letterCombinations(String digits) {
    if(digits == null || digits.length() == 0){
        return new ArrayList<>();
    }

    Map<Character, String> map = new HashMap<>();
    map.put('2', "abc");
    map.put('3', "def");
    map.put('4', "ghi");
    map.put('5', "jkl");
    map.put('6', "mno");
    map.put('7', "pqrs");
    map.put('8', "tuv");
    map.put('9', "wxyz");

    List<String> ans = new ArrayList<>();
    backtract(digits, new StringBuilder(), ans, map, 0);

    return ans;
}

public void backtract(String digits, StringBuilder subAns, List<String> ans, Map<Character, String> map, int index){
    if(subAns.length() == digits.length()){
        ans.add(new String(subAns));
        return;
    }

    char c = digits.charAt(index);
    String nums = map.get(c);

    for(int i = 0;i < nums.length();i ++){
        char c2 = nums.charAt(i);
        subAns.append(c2);
        backtract(digits, subAns, ans, map, index + 1);
        subAns.deleteCharAt(subAns.length() - 1);
    }

}
```



#### 19. 删除链表的倒数第 N 个结点

- 算法：快慢指针
- 注意：添加dummyHead

```
public ListNode removeNthFromEnd(ListNode head, int n) {
    if(head == null || n <= 0){
        return head;
    }
    
    ListNode dummyHead = new ListNode();
    dummyHead.next = head;
    ListNode p1 = dummyHead;
    ListNode p2 = dummyHead;

    for(int i = 1;i <= n;i ++){
        p1 = p1.next;
    }

    while(p1.next != null){
        p1 = p1.next;
        p2 = p2.next;
    }

    p2.next = p2.next.next;

    return dummyHead.next;
}
```



#### 20. >有效的括号

- 算法：利用栈
  - 遇到左符合则入栈
  - 遇到右符合则检查栈不为空且栈顶元素需为其左符号

```
public boolean isValid(String s) {

    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> map = new HashMap<>();
    map.put('(', ')');
    map.put('[', ']');
    map.put('{', '}');

    for(char c : s.toCharArray()){
        if(map.containsKey(c)){
            stack.push(c);
        }else {
            if(stack.isEmpty() || map.get(stack.peek()) != c){
                return false;
            }
            stack.pop();
        }
    }

    return stack.isEmpty();
}
```



#### [21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)

- 思路：归并排序

```
public ListNode mergeTwoLists(ListNode list1, ListNode list2) {

    ListNode dummyHead = new ListNode();
    ListNode p3 = dummyHead;
    ListNode p1 = list1;
    ListNode p2 = list2;

    while(p1 != null && p2 != null){
        if(p1.val < p2.val){
            p3.next = p1;
            p1 = p1.next;;
        }else {
            p3.next = p2;
            p2 = p2.next;;
        }

        p3 = p3.next;
    }

    while(p1 != null){
        p3.next = p1;
        p1 = p1.next;
        p3 = p3.next;
    }

    while(p2 != null){
        p3.next = p2;
        p2 = p2.next;
        p3 = p3.next;
    }

    return dummyHead.next;
}
```



#### [22. 括号生成](https://leetcode.cn/problems/generate-parentheses/)

- 算法：回溯法（全排列） - 暴力解法

```
public List<String> generateParenthesis(int n) {

    List<String> ans = new ArrayList<>();
    backtract(ans, new StringBuilder(), n, n);

    return ans;
}

public void backtract(List<String> ans, StringBuilder subAns, int left, int right){
    if(left == 0 && right == 0){
        String str = new String(subAns);
        if(valid(str)) ans.add(str);
        return;
    }

    if(left > 0){
        subAns.append("(");
        backtract(ans, subAns, left - 1, right);
        subAns.deleteCharAt(subAns.length() - 1);
    }

    if(right > 0){
        subAns.append(")");
        backtract(ans, subAns, left, right - 1);
        subAns.deleteCharAt(subAns.length() - 1);
    }
}

private boolean valid(String subAns){
    int cnt = 0;
    for(char c : subAns.toCharArray()){
        if(c == '('){
            cnt ++;
        }else {
            if(cnt > 0){
                cnt --;
            }else {
                return false;
            }
        }
    }

    return cnt == 0;
}
```

- **最优**：优化后

```
public List<String> generateParenthesis(int n) {

    List<String> ans = new ArrayList<>();
    backtract(ans, new StringBuilder(), n, n);

    return ans;
}

public void backtract(List<String> ans, StringBuilder subAns, int left, int right){
    if(left == 0 && right == 0){
        ans.add(new String(subAns));
        return;
    }

    if(left > 0){
        subAns.append("(");
        backtract(ans, subAns, left - 1, right);
        subAns.deleteCharAt(subAns.length() - 1);
    }
	
	// 关键点
    if(right > left){
        subAns.append(")");
        backtract(ans, subAns, left, right - 1);
        subAns.deleteCharAt(subAns.length() - 1);
    }
}
```

