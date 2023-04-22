---
layout: post
title: 算法练习一
category: java algorithm
---

使用LeetCode练习算法, 从易到难. 

## 1. Add Binary(二进制求和)
### 题目描述
给定2个二进制字符串a和b, 以二进制字符串的形式返回a,b之和
### 解法
1. 将a,b转化为十进制->求和->再转化为二进制字符串。利用的是java中自带的类型转换方法，但如果a,b长度超出范围,则不适用此方法
2. 逐位计算, 二进制逢二进一, [O(max(m,n))]
```java
public static String addBinary1(String a, String b) {
    StringBuilder sb = new StringBuilder();
    int i = a.length() - 1, j = b.length() - 1, carry = 0;
    while (i >= 0 || j >= 0) {
        int sum = carry;
        if (j >= 0) sum += b.charAt(j--) - '0';
        if (i >= 0) sum += a.charAt(i--) - '0';
        sb.append(sum % 2);
        carry = sum / 2;
    }
    if (carry != 0) sb.append(carry);
    return sb.reverse().toString();
}
```
3. 位运算, 将a,b转换为整数x,y, x保存结果, y保存进位
```java
public static String addBinary2(String a, String b) {
    BigInteger x = new BigInteger(a, 2);
    BigInteger y = new BigInteger(b, 2);
    BigInteger zero = new BigInteger("0", 2);
    BigInteger carry, answer;
    while (y.compareTo(zero) != 0) {
        // 异或, 计算x,y无进位相加结果
        answer = x.xor(y);
        // 按位与后左移1位, 计算进位
        carry = x.and(y).shiftLeft(1);
        x = answer;
        y = carry;
    }
    return x.toString(2);
}
```

## 2. Climbing Stairs(爬楼梯)
### 题目描述
假设需要n阶才能到达楼顶, 每次可以爬1或2个台阶, 有多少种不同方法爬到楼顶
### 解法
1. 递归, 会出现重复计算. T[O(2^n)]
```java
public static int climbStairs1(int n) {
    if (n == 1) {
        return 1;
    }
    if (n == 2) {
        return 2;
    }
    return climbStairs1(n-1) + climb_Stairs1(n-2);
}
```
2. 记忆化递归, 将计算过的保存, 不再重复计算. T[O(n)]
```java
   public static int climbStairs2(int n) {
        int memo[] = new int[n + 1];
        return climb_Stairs2(n, memo);
    }

    public static int climb_Stairs2(int n, int memo[]) {
        if (memo[n] > 0) return memo[n];
        if (n==1) memo[n] = 1;
        else if (n==2) memo[n] = 2;
        else memo[n] = climb_Stairs2(n-1, memo) + climb_Stairs2(n-2, memo);
        return memo[n];
    }
```
3. 动态规划

## 3. CountAndSay(外观数列)
### 题目描述
给定一个正整数N, 输出外观数列的第N项, 即第n项是对第n-1项的描述
* 外观数列：从数字1开始, 序列中每一项都是对前一项的描述。
* 例如: 1, 11(1个1), 21(2个1), 1211(1个2,1个1), 111221(1个1, 1个2, 2个1)

### 解法
Sliding Window(滑动窗口)
* 在函数中，我们使用两个上下文变量扫描序列：
* prevDigit 和 digitCnt 分别指的是我们期望在子序列中出现的数字和该数字在子序列中出现的次数。
* 在每个子序列的末尾，我们将摘要附加到结果中，然后为下一个子序列重置上述两个上下文变量。
* 在序列中使用了一个人为的分隔符，以方便迭代

```java
public static String countAndSay1(int n) {
    LinkedList<Integer> prevSeq = new LinkedList<>();
    prevSeq.add(1);
    // 用-1做分隔符
    prevSeq.add(-1);

    List<Integer> finalSeq = nextSequence(n, prevSeq);
    StringBuffer seqStr = new StringBuffer();
    for (Integer digit : finalSeq) {
        seqStr.append(digit);
    }
    return seqStr.toString();
}

private static LinkedList<Integer> nextSequence(int n, LinkedList<Integer> prevSeq) {
    if (n <= 1) {
        // 返回之前删除分隔符
        prevSeq.pollLast();
        return prevSeq;
    }

    LinkedList<Integer> nextSeq = new LinkedList<>();
    Integer prevDigit = null;
    int digitCnt = 0;
    for (Integer digit : prevSeq) {
        if (prevDigit == null) {
            prevDigit = digit;
            digitCnt += 1;
        } else if (digit.equals(prevDigit)) {
            // 子序列中间
            digitCnt += 1;
        } else {
            // 子序列末尾
            nextSeq.add(digitCnt);
            nextSeq.add(prevDigit);
            // 为下一个子序列重置两个变量
            prevDigit = digit;
            digitCnt = 1;
        }
    }

    // 为下一个递归添加分隔符
    nextSeq.add(-1);
    return nextSequence(n - 1, nextSeq);
}
```

iteration(迭代)
* S1=1, S2=11,... Sn.

```java
public static String countAndSay2(int n) {
    StringBuilder result = new StringBuilder("1");
    for (int outer = 1; outer < n; outer++) {
        String previous = result.toString();
        result = new StringBuilder();
        int count = 1;
        char say = previous.charAt(0);

        for (int i = 1; i < previous.length(); i++) {
            if (previous.charAt(i) != say) {
                result.append(count).append(say);
                count = 1;
                say = previous.charAt(i);
            } else count++;
        }
        result.append(count).append(say);
    }
    return result.toString();

}
```

Regular Expression(正则)

```java
public static String countAndSay3(int n) {
    String currentSeq = "1";

    // 匹配重复数字
    String regexPattern = "(.)\\1*";
    Pattern pattern = Pattern.compile(regexPattern);

    for (int i = 1; i < n; i++) {
        Matcher m = pattern.matcher(currentSeq);
        StringBuffer nextSeq = new StringBuffer();

        // each group contains identical and adjacent digits
        while (m.find()) {
            nextSeq.append(m.group().length() + String.valueOf(m.group().charAt(0)));
        }
        // prepare for the next iteration
        currentSeq = nextSeq.toString();
    }
    return currentSeq;
}
```

recursion(递归)

```java
public static String countAndSay4(int n) {
    if (n == 1) return "1";
    String s = countAndSay4(n - 1);

    StringBuilder sb = new StringBuilder();
    int count = 1;
    for (int i = 1; i < s.length(); i++) {
        if (s.charAt(i) == s.charAt(i - 1)) count++;
        else{
            sb.append(count).append(s.charAt(i - 1));
            count = 1;
        }
    }
    sb.append(count).append(s.charAt(s.length() - 1));
    return sb.toString();
}
```

## 4. Find the Index of the First Occurrence in a String(找出字符串中第一个匹配项的下标)
### 题目描述
给定两个字符串haystack和needle,在haystack字符串中找出needle字符串的第一个匹配项的下标(下标从 0 开始)。如果needle不是haystack的一部分，则返回-1
### 解法
1. Sliding Window(滑动窗口), 使用substring逐个匹配
```java
public static int strStr1(String haystack, String needle) {
    for (int i =0; i <= haystack.length() - needle.length(); i++) {
        if (needle.equals(haystack.substring(i, needle.length() + i))) {
            return i;
        }
    }
    return -1;
}
```
2. 两点法
```java
public static int strStr2(String haystack, String needle) {
    int l = needle.length();
    int n = haystack.length();
    if (l == 0) return 0;

    int pn = 0;
    while (pn < n - l + 1) {
        // 找到needle中第一个字符在haystack中匹配的位置
        while (pn < n - l + 1 && haystack.charAt(pn) != needle.charAt(0)) ++pn;

        // 计算最大匹配字符串
        int pl = 0;
        int current = 0;
        while (pl < l && pn < n && haystack.charAt(pn) == needle.charAt(pl)) {
            ++pn;
            ++pl;
            ++current;
        }
        // 整个子串找到就返回起始位置
        if (current == pl) return pn - l;

        // 否则回溯重新开始
        pn = pn - current + 1;
    }
    return -1;
}
```
3. rabin-karp算法, 使用散列函数以在文本中搜寻单个模式串的字符串搜索算法单次匹配。该算法先使用旋转哈希以快速筛出无法与给定串匹配的文本位置，此后对剩余位置能否成功匹配进行检验

## 5. Length of Last Word(最后一个单词的长度)
### 题目描述
给定一个字符串s, 由若干单词组成, 单词前后使用空格符隔开, 返回字符串中最后一个单词长度
### 解法
1. 逐字符循环
```java
public static int lengthOfLastWord1(String s) {
    int l = s.length(), lastLength = 0;
    while (l > 0 && s.charAt(l - 1) == ' ') {
        l--;
    }
    while ((l > 0 && s.charAt(l - 1) != ' ')) {
        lastLength++;
        l--;
    }
    return lastLength;
}
```
2. 
```java
public static int lengthOfLastWord2(String s) {
    int l = 0;
    for (int i = s.length() - 1; i >= 0; i--) {
        if (s.charAt(i) != ' ') {
            l++;
        }
        if (l > 0) return l;
    }
    return l;
}
```

## 6. Longest Common Prefix(最长公共前缀)
### 题目描述
编写一个函数来查找字符串数组中的最长公共前缀,如果不存在公共前缀，返回空字符串""
### 解法
1. Horizontal scanning(水平扫描)
```java
public static String longestCommonPrefix2(String[] strs) {
    if (strs.length == 0) {
        return "";
    }
    String prefix = strs[0];
    for (int i = 1; i < strs.length; i++) {
        while (strs[i].indexOf(prefix) != 0) {
            prefix = prefix.substring(0, prefix.length() - 1);
            if (prefix.isEmpty()) {
                return "";
            }
        }
    }
    return prefix;
}
```
2. Vertical scanning(垂直扫描)
```java
public static String longestCommonPrefix3(String[] strs) {
    if (strs == null || strs.length == 0) {
        return "";
    }
    for (int i = 0; i < strs[0].length(); i++) {
        char c = strs[0].charAt(i);
        for (int j = 1; j < strs.length; j++) {
            if (strs[j].length() == i || strs[j].charAt(i) != c) {
                return strs[0].substring(0, i);
            }
        }
    }
    return strs[0];
}
```
3. recursive(递归)

```java
public static String longestCommonPrefix4(String[] strs) {
    if (strs == null || strs.length == 0) {
        return "";
    }
    return longestCommonPrefix(strs, 0, strs.length - 1);
}

private static String longestCommonPrefix(String[] strs, int l, int r) {
    if (l == r) {
        return strs[l];
    } else {
        int mid = (l + r) / 2;
        String lcpLeft = longestCommonPrefix(strs, l, mid);
        String lcpRight = longestCommonPrefix(strs, mid + 1, r);
        return commonPrefix(lcpLeft, lcpRight);
    }
}

private static String commonPrefix(String left, String right) {
    int min = Math.min(left.length(), right.length());
    for (int i = 0; i < min; i++) {
        if (left.charAt(i) != right.charAt(i)) {
            return left.substring(0, i);
        }
    }
    return left.substring(0, min);
}
```


## 7. Maximum Subarray(最大子数组和)
### 题目描述
给定一个整数数组nums,找出一个具有最大和的连续子数组（子数组最少包含一个元素）,返回其最大和
### 解法
分而治之
```java
public static int maxSubArray1(int[] nums) {
    return helper(nums, 0, nums.length - 1);
}
private static int helper(int[] nums, int left, int right) {
    if (left == right) return nums[left];
    int mid = (left + right) / 2;
    int leftSum = helper(nums, left, mid);
    int rightSum = helper(nums, mid + 1, right);
    int crossSum = crossSub(nums, left, right, mid);

    return Math.max(Math.max(leftSum, rightSum), crossSum);
}
private static int crossSub (int[] nums, int left, int right, int mid) {
    if (left == right) return nums[left];

    int leftSubSum = Integer.MIN_VALUE;
    int currentSum = 0;
    for (int i = mid; i > left - 1; --i) {
        currentSum += nums[i];
        leftSubSum = Math.max(leftSubSum, currentSum);
    }

    int rightSubSum = Integer.MIN_VALUE;
    currentSum = 0;
    for (int i = mid + 1; i < right + 1; ++i) {
        currentSum += nums[i];
        rightSubSum = Math.max(rightSubSum, currentSum);
    }
    return leftSubSum + rightSubSum;
}
```
Greedy-贪婪算法
```java
public static int maxSubArray2(int[] nums) {
    int n = nums.length;
    int currentSum = nums[0], maxSum = nums[0];
    for (int i = 1; i < n; ++i) {
        currentSum = Math.max(nums[i], currentSum + nums[i]);
        maxSum = Math.max(maxSum, currentSum);
    }
    return maxSum;
}
```
Dynamic Programming-(Kadane算法)

```java
public static int maxSubArray3(int[] nums) {
    int n = nums.length, maxSum = nums[0];
    for (int i = 1; i < n; ++i) {
        if (nums[i - 1] > 0) nums[i] += nums[i - 1];
        maxSum = Math.max(nums[i], maxSum);
    }
    return maxSum;
}

public static int maxSubArray4(int[] nums) {
    int sum = 0, max = Integer.MIN_VALUE;
    for (int i = 0; i < nums.length; i++) {
        sum = sum < 0 ? nums[i] : (sum + nums[i]);
        max = Math.max(sum, max);
    }
    return max;
}
```

## 8. Palindrome(回文数)
### 题目描述
给定一个整数x ，如果x是一个回文整数，返回 true;否则返回false
* 回文: 从左向右读和从右向左读都是一样的
### 解法
1. 反转一半数字(防止整个反转导致溢出)
```java
public static boolean isPalindrome2(int x) {
    // 临界情况
    if (x < 0 || (x % 10 == 0 && x != 0)) {
        return false;
    }
    int rev = 0;
    while (x > rev) {
        rev = rev * 10 + x % 10;
        x /= 10;
    }
    // 当x为奇数时, 使用rev/10去除中间数字(中间数字不影响回文)后比较
    return x == rev || x == rev/10;
}
```
2. 转成字符串比较
```java
public static boolean isPalindrome3(int x) {
    String str = String.valueOf(x);
    int start = 0;
    int end = str.length() - 1;
    while (start < end) {
        if (str.charAt(start++) != str.charAt(end--)) {
            return false;
        }
    }
    return true;
}
```

## 9. Plus One(加一)
### 题目描述
给定一个由正整数组成的非空数组所表示的整数，在该数的基础上加一。数组中每个元素只存储单个数字, 且这些数字按从左到右的顺序从最高有效位到最低有效位排序。除了整数 0 之外，这个整数不会以零开头。
### 解法
1. 转成1个整数计算后再转成数组
```java
public static int[] plusOne2(int[] nums) {
    int length = nums.length - 1;
    int temp = 0;
    int i = 0;
    for (; length >= 0 ; length--) {
        temp += nums[length] * (int)Math.pow(10, i++);
    }
    System.out.println(temp);

    String s = Integer.toString(temp + 1);
    int[] result;
    if (s.length() > nums.length) {
        result = new int[nums.length + 1];
    } else {
        result = new int[nums.length];
    }
    for (int j = 0; j < s.length(); j++) {
        System.out.println(s.charAt(j));
        result[j] =  s.charAt(j) - '0';
    }

    return result;
}
```
2. 找出所有9
```java
public static int[] plusOne(int[] nums) {
    int n = nums.length;

    // 逆序遍历
    for (int i = n - 1; i >= 0; i-- ) {
        // 将数组末尾所有9设置为0
        if (nums[i] == 9) {
            nums[i] = 0;
        }
        // 最右边第一个非9
        else {
            // 将该非9加1
            nums[i]++;
            return nums;
        }
    }
    // 若所有数都为9
    nums = new int[n + 1];
    nums[0] = 1;
    return nums;
}
```

## 10. Remove Duplicates from Sorted List(删除排序链表中的重复元素)
### 题目描述
给定一个已排序的的头he链表ad ， 删除所有重复的元素，使每个元素只出现一次.返回已排序的链表
### 解法
一次遍历

```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { val = x;}
}

public class RemoveDuplicatesFromSortedList {
    public static ListNode deleteDuplicates1(ListNode head) {
        ListNode current = head;
        while (current != null && current.next != null) {
            if (current.next.val == current.val) {
                current.next = current.next.next;
            } else {
                current = current.next;
            }
        }
        return head;
    }
}
```

递归
```java
public ListNode deleteDuplicates2(ListNode head) {
    if(head == null || head.next == null)return head;
    head.next = deleteDuplicates2(head.next);
    return head.val == head.next.val ? head.next : head;
}
```


## 11. Remove Duplicates from Sorted Array(删除有序数组中的重复项)
### 题目描述
给你一个升序排列的数组nums ，请你原地(不使用额外空间)删除重复出现的元素，使每个元素只出现一次,返回删除后数组的新长度
### 解法
1. 双指针
```java
public static int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    int i = 0;
    for (int j = 1; j < nums.length; j++) {
        if (nums[j] != nums[i]) {
            i++;
            nums[i] = nums[j];
        }
    }
    return i + 1;
}
```

## 12. Remove Element(移除元素)
### 题目描述
给定一个数组nums和一个值val，你需要原地(不使用额外空间)移除所有数值等于val的元素，并返回移除后数组的新长度
### 解法
1. 双指针
```java
public static int removeElement1(int[] nums, int val) {
    int i = 0;
    for (int j = 0; j < nums.length; j++) {
        if (nums[j] != val) {
            nums[i] = nums[j];
            i++;
        }
    }
    return i;
}
```
2. 双指针优化,
```java
public static int removeElement3(int[] nums, int val) {
    int i = 0;
    int n = nums.length;
    while (i < n) {
        if (nums[i] == val) {
            nums[i] = nums[n - 1];
            n--;
        }
        i++;
    }
    return n;
}
```

## 13. Reverse Integer(整数反转)
### 题目描述
给定一个32位的有符号整数 x ，返回将x中的数字部分反转后的结果,如果反转后整数超过 32 位的有符号整数的范围 [−2^31, 2^31 − 1] ，就返回0
### 解法
1. pop and push & check overflow 弹出和推入
```java
    public static int reverse1(int x) {
        // 假设rev为反转后数字
        int rev = 0;
        while (x != 0) {
            // 弹出x的末尾
            int pop = x % 10;
            x /= 10;
            // 判断rev是否大于2^31 − 1
            if (rev > Integer.MAX_VALUE / 10 || (rev == Integer.MAX_VALUE / 10 && pop > 7)) {
                return 0;
            }
            // 判断rev是否小于-2^31
            if (rev < Integer.MIN_VALUE / 10 || (rev == Integer.MIN_VALUE / 10 && pop < -8)) {
                return 0;
            }
            // 推入rev末尾
            rev = rev * 10 + pop;
        }
        return rev;
    }
```
2. string reverse
```java
public static int reverse2(int x) {
    String ans = x < 0 ? new StringBuilder(String.valueOf(-x)).append("-").reverse().toString()
            : new StringBuilder(String.valueOf(x)).reverse().toString();
    try {
        return Integer.parseInt(ans);
    } catch (Exception e) {
        return 0;
    }
}
```

## 14. Roman to Integer(罗马数字转整数)
### 题目描述
给定一个罗马数字，将其转换成整数, 罗马数字包含以下七种字符: I(1), V(5), X(10), L(50), C(100), D(500),  M(1000)
通常情况下，罗马数字中小的数字在大的数字的右边。但也存在特例:
* I 可以放在 V (5) 和 X (10) 的左边，来表示 4 和 9。
* X 可以放在 L (50) 和 C (100) 的左边，来表示 40 和 90。 
* C 可以放在 D (500) 和 M (1000) 的左边，来表示 400 和 900

### 解法
暴力解法
```java
public static int romanToInt1(String x) {
    String number = "";
    int result = 0;
    for (int i = 0; i < x.length(); i++) {
        char temp = x.charAt(i);
        if ('I' == temp) {
            if (i + 1 < x.length()) {
                if ('V' == x.charAt(i + 1)) {
                    number = "4";
                    //++i;
                }
                if ('X' == x.charAt(i + 1)) {
                    number = "9";
                    //++i;
                }
            }

            number = "1";
        }
        if ('X' == temp) {
            if (i + 1 < x.length()) {
                if ('L' == x.charAt(i + 1)) {
                    number = "40";
                    ++i;
                }
                if ('C' == x.charAt(i + 1)) {
                    number = "90";
                    ++i;
                }
            }
            number = "10";
        }
        if ('C' == temp) {
            if (i + 1 < x.length()) {
                if ('D' == x.charAt(i + 1)) {
                    number = "400";
                    ++i;
                }
                if ('M' == x.charAt(i + 1)) {
                    number = "900";
                    ++i;
                }
            }
            number = "100";
        }
        if ('V' == temp) {
            number = "5";
        }
        if ('L' == temp) {
            number = "50";
        }
        if ('D' == temp) {
            number = "500";
        }
        if ('M' == temp) {
            number = "1000";
        }
        result += Integer.parseInt(number.trim());
    }
    return result;
}
```
逐字计算

```java
static Map<String, Integer> values = new HashMap<>();
static {
    values.put("M", 1000);
    values.put("D", 500);
    values.put("C", 100);
    values.put("L", 50);
    values.put("X", 10);
    values.put("V", 5);
    values.put("I", 1);

}
public static int romanToInt2(String x) {
    int sum  = 0;
    int i = 0;
    while (i <  x.length()) {
        String currentSymbol = x.substring(i, i + 1);
        // 左边数
        int currentValue = values.get(currentSymbol);
        //右边数
        int nextValue = 0;
        if (i + 1 < x.length()) {
            String nextSymbol = x.substring(i + 1, i + 2);
            nextValue = values.get(nextSymbol);
        }
        // 左小于右则属于特殊规则, 左右数为1个数
        if (currentValue < nextValue) {
            sum += (nextValue - currentValue);
            i += 2;
        } else {
            sum += currentValue;
            i += 1;
        }
    }
    return sum;
}
```

两两一组计算

```java
static Map<String, Integer> map = new HashMap<>();
static {
    map.put("IV", 4);
    map.put("IX", 9);
    map.put("XL", 40);
    map.put("XC", 90);
    map.put("CD", 400);
    map.put("CM", 900);
    map.put("M", 1000);
    map.put("D", 500);
    map.put("C", 100);
    map.put("L", 50);
    map.put("X", 10);
    map.put("V", 5);
    map.put("I", 1);
}
public static int romanToInt3(String x) {
    int sum = 0;
    int i = 0;
    while (i < x.length()) {
        if (i < x.length() - 1) {
            String doubleSymbol = x.substring(i, i + 2);
            if(map.containsKey(doubleSymbol)) {
                sum += map.get(doubleSymbol);
                i += 2;
                continue;
            }
        }
        String singleSymbol = x.substring(i, i + 1);
        sum += map.get(singleSymbol);
        i += 1;
    }
    return sum;
}
```

特殊规则与正常计算之间分别差2, 20, 200

```java
public static int romanToInt5(String x) {
    int sum = 0;
    if (x.contains("IV")) { sum -= 2; }
    if (x.contains("IX")) { sum -= 2; }
    if (x.contains("XL")) { sum -= 20; }
    if (x.contains("XC")) { sum -= 20; }
    if (x.contains("CD")) { sum -= 200; }
    if (x.contains("CM")) { sum -= 200; }

    char[] s = x.toCharArray();
    int index = 0;

    for (; index < x.length(); index ++) {
        if (s[index] == 'M') { sum += 1000; }
        if (s[index] == 'D') { sum += 500; }
        if (s[index] == 'C') { sum += 100; }
        if (s[index] == 'L') { sum += 50; }
        if (s[index] == 'X') { sum += 10; }
        if (s[index] == 'V') { sum += 5; }
        if (s[index] == 'I') { sum += 1; }
    }
    return sum;
}
```

## 15. Search Insert Position(搜索插入位置)
### 题目描述
给定一个排序数组和一个目标值，在数组中找到目标值,并返回其索引. 如果目标值不存在于数组中,返回它将会被按顺序插入的位置.必须使用时间复杂度为 O(log n) 的算法
### 解法
1. 二分查找
```java
public static int searchInsert1(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    int mid = 0;
    while (left <= right) {
        mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        if (target < nums[mid]) right = mid - 1;
        else left = mid + 1;
    }
    return left;
}
```
2. 递归+二分
```java
public static int searchInsert2(int[] nums, int target) {
    return searchInsert(nums, target, 0, nums.length - 1);
}
private static int searchInsert(int[] nums, int target, int low, int hight) {
    int mid = (low + hight) / 2;
    if (target < nums[mid]) {
        if (mid == 0 || target > nums[mid - 1]) {
            return mid;
        }
        return searchInsert(nums, target, low, mid - 1);
    }
    if (target > nums[mid]) {
        if (mid == nums.length - 1 || target < nums[mid + 1]) {
            return mid + 1;
        }
        return searchInsert(nums, target, mid + 1, hight);
    }
    return mid;
}
```

## 16. Sqrt(x)(x的平方根)
### 题目描述
给定一个非负整数x ，计算并返回x的算术平方根。返回结果只保留整数部分。不允许使用任何内置指数函数和算符，例如pow(x, 0.5) 或者 x**0.5
### 解法
1. Pocket Calculator Algorithm(袖珍计算器算法):用指数函数exp和对数函数ln代替平方根函数的方法
```java
public static int mySqrt1(int x) {
    if (x == 0) {
        return 0;
    }
    int result = (int) Math.exp(0.5 * Math.log(x));
    return (long) (result + 1) * (result + 1) <= x ? result + 1 : result;
}
```
2. 二分查找
```java
public static int mySqrt2(int x) {
    if (x < 2) return x;
    int left = 2, right = x / 2;
    while (left <= right) {
        int num = (left + right) / 2;
        if (num * num > x) right = num - 1;
        if (num * num < x) left = num + 1;
        if (num * num == x) return num;
    }
    return right;
}
```

## 17. Two Sum(两数之和)
### 题目描述
给定一个整数数组nums和一个整数目标值target，在该数组中找出和为目标值target 的那两个整数，并返回它们的数组下标。
你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。你可以按任意顺序返回答案。
### 解法
1. 暴力解法
```java
public static int[] twoSum1(int[] nums, int target) {
    for (int i = 0; i < nums.length; i ++) {
        for (int j = i + 1; j < nums.length; j++) {
            if (nums[i] != nums[j] && nums[i] + nums[j] == target) {
                return new int[] {i, j};
            }
        }
    }
    throw new IllegalArgumentException();
}
```
2. 哈希表
```java
public static int[] twoSum2(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        map.put(nums[i], i);
    }

    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement) && map.get(complement) != i) {
            return new int[]{i, map.get(complement)};
        }
    }
    throw new IllegalArgumentException();
}
```
3. 哈希表改进
```java
public static int[] twoSum3(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[]{map.get(complement), i};
        }
        map.put(nums[i], i);
    }
    throw new IllegalArgumentException();
}
```

##  18. Valid Parentheses(有效的括号)
### 题目描述
给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串s ，判断字符串是否有效。
有效字符串需满足：
* 左括号必须用相同类型的右括号闭合。
* 左括号必须以正确的顺序闭合。
* 每个右括号都有一个对应的相同类型的左括号。

### 解法
栈

```java
static HashMap<Character, Character> map = new HashMap<>();
static {
    map.put('}', '{');
    map.put(']', '[');
    map.put(')', '(');
}

public static boolean validParentheses2(String str) {
    Stack<Character> stack = new Stack<>();
    for (int i = 0; i < str.length(); i++){
        char c = str.charAt(i);

        //如果当前字符是右括号
        if (map.containsKey(c)) {
            //获取栈顶元素
            char topElement = stack.empty() ? '#' : stack.pop();
            // 如果此括号与栈顶元素不匹配则返回false
            if (topElement != map.get(c)) {
                return false;
            }
        } else {
            //如果当前字符为左括号则入栈
            stack.push(c);
        }
    }
    //如果栈中仍有元素, 则输入无效
    return stack.isEmpty();
}
```


## to be continued