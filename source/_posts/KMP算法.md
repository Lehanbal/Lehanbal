---
title: KMP算法
date: 2020-10-29 14:56:09
tags:
- KMP
- JAVA
categories:
- JAVA
---



KMP 算法是 D.E.Knuth、J,H,Morris 和 V.R.Pratt 三位神人共同提出的，称之为 Knuth-Morria-Pratt 算法，简称 KMP 算法。该算法相对于 Brute-Force（暴力）算法有比较大的改进，主要是消除了主串指针的回溯，从而使算法效率有了某种程度的提高。

KMP适用于重复字串多的短串，在这种模式下会有很好的效果，当重复的字串不太多的时候，并且串还有点长，那么KMP的表现和暴力匹配会差不多。KMP会额外的开辟一个匹配串长度大小的空间，相当于用空间换时间。但是，既然都会了KMP，并且题目并没有明确的要求空间复杂度的时候，能用上KMP就用上。

推荐观看的KMP教学视频：[KMP算法实例详解(易懂)](https://www.bilibili.com/video/BV1S64y1u74P?from=search&seid=13411543487636624526)

## 暴力匹配

在说KMP之前，有必要了解暴力匹配，KMP就是在暴力匹配的基础上进行的算法优化。

现在有个题目：在字符串“111011101”找到第一个“10”的位置。

```java
    public int violentMatch(String source, String text){
        char[] str = source.toCharArray();
        char[] match = text.toCharArray();
        int i = 0, j = 0;
        while(i < str.length && j < match.length){
            if(str[i] == match[j]){
                i++;
                j++;
            }else{
                i = i - j + 1;
                j = 0;
            }
        }
        if(j == match.length) return i - j + 1;
        return -1;
    }

    public static void main(String[] args) {
        ViolentMatch vm = new ViolentMatch();
        System.out.println(vm.violentMatch("111011101", "10"));
    }
```

从上述的代码中，暴力匹配就是将匹配的字符串挨个去匹配源字符串的每个字符，如果匹配成功，匹配的字符就各自往前移动一位，否则源字符串就退回到最开始匹配的地方并且往前跳一个位置，再开始继续匹配。

这种匹配模式，如果源字符串和匹配字符串中有很多重复的地方，这个匹配方式依旧会挨个进行全匹配，这会很耗时间，下面说的KMP就是比较智能的方式，这个算法会跳过字符串中出现重复的地方。

为了实现这个操作，我们需要构造一个Next表，这个表记录着匹配的字符串匹配失败后应该跳转的位置。

## 制作Next表

next表所记录的就是匹配字符串当中相同的前后缀出现的位数，进行匹配的时候，如果前面的字符串出现过，那么就跳过就好了，不需要再进行匹配。

```java
    public int[] getNext(String text){
        int[] next = new int[text.length()];
        next[0] = -1;//相当于next表的第一个是万能符，全都能匹配，不然下面会进行死循环
        int i = -1, j = 0;
        while(j < text.length() - 1) {
            if (i == -1 || text.charAt(j) == text.charAt(i)) {
                j++;
                i++;
                next[j] = i;
            } else {
                i = next[i];
            }
        }
        return next;
    }
```

## KMP

KMP和暴力匹配挺像的，但是KMP会用到next作为辅助判断，来计算匹配失败后匹配字符串所需要跳转的位置，相当于经过了一部预处理。

```java
    public int kmp(String source, String text){
        int[] next = getNext(text);
        char[] str = source.toCharArray();
        char[] match = text.toCharArray();
        int i = 0, j = 0;
        while(i < str.length && j < match.length){
            if(j == -1 || str[i] == match[j]){
                i++;
                j++;
            }else{
                j = next[j];
            }
        }
        if(j == match.length) return i - j + 1;
        return -1;
    }
```

