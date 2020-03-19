## Manacher算法：

### 原始问题

Manacher算法是由题目“求字符串中最长回文子串的长度”而来。比如`abcdcb`的最长回文子串为`bcdcb`，其长度为5。

我们可以遍历字符串中的每个字符，当遍历到某个字符时就比较一下其左边相邻的字符和其右边相邻的字符是否相同，如果相同则继续比较其右边的右边和其左边的左边是否相同，如果相同则继续比较……，我们暂且称这个过程为向外“扩”。当“扩”不动时，经过的所有字符组成的子串就是以当前遍历字符为中心的最长回文子串。

我们每次遍历都能得到一个最长回文子串的长度，使用一个全局变量保存最大的那个，遍历完后就能得到此题的解。但分析这种方法的时间复杂度：当来到第一个字符时，只能扩其本身即1个；来到第二个字符时，最多扩两个；……；来到字符串中间那个字符时，最多扩`(n-1)/2+1`个；因此时间复杂度为`1+2+……+(n-1)/2+1`即`O(N^2)`。但Manacher算法却能做到`O(N)`。

Manacher算法中定义了如下几个概念：

- 回文半径：串中某个字符最多能向外扩的字符个数称为该字符的回文半径。比如`abcdcb`中字符`d`，能扩一个`c`，还能再扩一个`b`，再扩就到字符串右边界了，再算上字符本身，字符`d`的回文半径是3。
- 回文半径数组`pArr`：长度和字符串长度一样，保存串中每个字符的回文半径。比如`charArr="abcdcb"`，其中`charArr[0]='a'`一个都扩不了，但算上其本身有`pArr[0]=1`；而`charArr[3]='d'`最多扩2个，算上其本身有`pArr[3]=3`。
- 最右回文右边界`R`：遍历过程中，“扩”这一操作扩到的最右的字符的下标。比如`charArr=“abcdcb”`，当遍历到`a`时，只能扩`a`本身，向外扩不动，所以`R=0`；当遍历到`b`时，也只能扩`b`本身，所以更新`R=1`；但当遍历到`d`时，能向外扩两个字符到`charArr[5]=b`，所以`R`更新为5。
- 最右回文右边界对应的回文中心`C`：`C`与`R`是对应的、同时更新的。比如`abcdcb`遍历到`d`时，`R=5`，`C`就是`charArr[3]='d'`的下标`3`。

处理回文子串长度为偶数的问题：上面拿`abcdcb`来举例，其中`bcdcb`属于一个回文子串，但如果回文子串长度为偶数呢？像`cabbac`，按照上面定义的“扩”的逻辑岂不是每个字符的回文半径都是0，但事实上`cabbac`的最长回文子串的长度是6。因为我们上面“扩”的逻辑默认是将回文子串当做奇数长度的串来看的，因此我们在使用Manacher算法之前还需要将字符串处理一下，这里有一个小技巧，那就是将字符串的首尾和每个字符之间加上一个特殊符号，这样就能将输入的串统一转为奇数长度的串了。比如`abba`处理过后为`#a#b#b#a`，这样的话就有`charArr[4]='#'`的回文半径为4，也即原串的最大回文子串长度为4。相应代码如下：

```java
public static char[] manacherString(String str) {  
    char[] charArr = str.toCharArray();  
    char[] res = new char[str.length() * 2 + 1];  
    int index = 0;  
    for (int i = 0; i != res.length; i++) {  
        res[i] = (i & 1) == 0 ? '#' : charArr[index++];  
    }  
    return res;  
}
```

接下来分析，BFPRT算法是如何利用遍历过程中计算的`pArr`、`R`、`C`来为后续字符的回文半径的求解加速的。

首先，情况1是，遍历到的字符下标`cur`在`R`的右边（起初另`R=-1`），这种情况下该字符的最大回文半径`pArr[cur]`的求解无法加速，只能一步步向外扩来求解。



![img](33Manacher%E7%AE%97%E6%B3%95/169045e8308cb40c)



情况2是，遍历到的字符下标`cur`在`R`的左边，这时`pArr[cur]`的求解过程可以利用之前遍历的字符回文半径信息来加速。分别做`cur`、`R`关于`C`的对称点`cur'`和`L`：

- 如果从`cur'`向外扩的最大范围的左边界没有超过`L`，那么`pArr[cur]=pArr[cur']`。

    

  ![img](33Manacher%E7%AE%97%E6%B3%95/169045e83279de43)

  

  证明如下：

  

  ![img](33Manacher%E7%AE%97%E6%B3%95/169045e83231d1ba)

  

  由于之前遍历过`cur'`位置上的字符，所以该位置上能扩的步数我们是有记录的（`pArr[cur']`），也就是说`cur'+pArr[cur']`处的字符`y'`是不等于`cur'-pArr[cur']`处的字符`x'`的。根据`R`和`C`的定义，整个`L`到`R`范围的字符是关于`C`对称的，也就是说`cur`能扩出的最大回文子串和`cur'`能扩出的最大回文子串相同，因此可以直接得出`pArr[cur]=pArr[cur']`。

- 如果从`cur'`向外扩的最大范围的左边界超过了`L`，那么`pArr[cur]=R-cur+1`。

    

  ![img](33Manacher%E7%AE%97%E6%B3%95/169045e83297b708)

  

  证明如下：

  

  ![img](33Manacher%E7%AE%97%E6%B3%95/169045e8328b74da)

  

  `R`右边一个字符`x`，`x`关于`cur`对称的字符`y`，`x,y`关于`C`对称的字符`x',y'`。根据`C,R`的定义有`x!=x'`；由于`x',y'`在以`cur'`为中心的回文子串内且关于`cur'`对称，所以有`x'=y'`，可推出`x!=y'`；又`y,y'`关于`C`对称，且在`L,R`内，所以有`y=y'`。综上所述，有`x!=y`，因此`cur`的回文半径为`R-cur+1`。

- 以`cur'`为中心向外扩的最大范围的左边界正好是`L`，那么`pArr[cur] >= （R-cur+1）`

    

  ![img](33Manacher%E7%AE%97%E6%B3%95/169045e8329df8aa)

  

  这种情况下，`cur'`能扩的范围是`cur'-L`，因此对应有`cur`能扩的范围是`R-cur`。但`cur`能否扩的更大则取决于`x`和`y`是否相等。而我们所能得到的前提条件只有`x!=x'`、`y=y'`、`x'!=y'`，无法推导出`x,y`的关系，只知道`cur`的回文半径最小为`R-cur+1`（算上其本身），需要继续尝试向外扩以求解`pArr[cur]`。

综上所述，`pArr[cur]`的计算有四种情况：暴力扩、等于`pArr[cur']`、等于`R-cur+1`、从`R-cur+1`继续向外扩。使用此算法求解原始问题的过程就是遍历串中的每个字符，每个字符都尝试向外扩到最大并更新`R`（只增不减），每次`R`增加的量就是此次能扩的字符个数，而`R`到达串尾时问题的解就能确定了，因此时间复杂度就是每次扩操作检查的次数总和，也就是`R`的变化范围（`-1~2N`，因为处理串时向串中添加了`N+1`个`#`字符），即`O(1+2N)=O(N)`。

整体代码如下：

```java
package com.gjxaiou.advanced.day01;

public class Manacher {
    public static char[] manacherString(String str) {
        char[] charArr = str.toCharArray();
        char[] res = new char[str.length() * 2 + 1];
        int index = 0;
        for (int i = 0; i != res.length; i++) {
            res[i] = (i & 1) == 0 ? '#' : charArr[index++];
        }
        return res;
    }

    public static int maxLcpsLength(String str) {
        if (str == null || str.length() == 0) {
            return 0;
        }
        char[] charArr = manacherString(str);
        // 回文半径数组
        int[] pArr = new int[charArr.length];
        // index  为对称中心 C
        int index = -1;
        int pR = -1;
        int max = Integer.MIN_VALUE;
        for (int i = 0; i != charArr.length; i++) {
            // 2 * index - 1 就是对应 i' 位置，R> i,表示 i 在回文右边界里面，则最起码有一个不用验的区域，
            pArr[i] = pR > i ? Math.min(pArr[2 * index - i], pR - i) : 1;
            // 四种情况都让扩一下，其中 1 和 4 会成功，但是 2 ，3 会失败则回文右边界不改变；可以自己写成 if-else 问题。
            while (i + pArr[i] < charArr.length && i - pArr[i] > -1) {
                if (charArr[i + pArr[i]] == charArr[i - pArr[i]]) {
                    pArr[i]++;
                } else {
                    break;
                }
            }
            if (i + pArr[i] > pR) {
                pR = i + pArr[i];
                index = i;
            }
            max = Math.max(max, pArr[i]);
        }
        return max - 1;
    }

    public static void main(String[] args) {
        int length = maxLcpsLength("123abccbadbccba4w2");
        System.out.println(length);
    }
}

```

上述代码将四种情况的分支处理浓缩到了`7~14`行。其中第`7`行是确定加速信息：如果当前遍历字符在`R`右边，先算上其本身有`pArr[i]=1`，后面检查如果能扩再直接`pArr[i]++`即可；否则，当前字符的`pArr[i]`要么是`pArr[i']`（`i`关于`C`对称的下标`i'`的推导公式为`2*C-i`），要么是`R-i+1`，要么是`>=R-i+1`，可以先将`pArr[i]`的值置为这三种情况中最小的那一个，后面再检查如果能扩再直接`pArr[i]++`即可。

最后得到的`max`是处理之后的串（`length=2N+1`）的最长回文子串的半径，`max-1`刚好为原串中最长回文子串的长度。