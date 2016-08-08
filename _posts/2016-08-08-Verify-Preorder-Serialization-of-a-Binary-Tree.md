---
layout: post
title: Verify Preorder Serialization of a Binary Tree[leetcode 算法题]
date: 2016-08-08
categories: blog
tags:
description:
---
## 问题描述
One way to serialize a binary tree is to use pre-order traversal. When we encounter a non-null node, we record the node's value. If it is a null node, we record using a sentinel value such as <b>#</b>.

           _9_
          /   \
         3     2
        / \   / \
       4   1  #  6
      / \ / \   / \
      # # # #   # #

For example, the above binary tree can be serialized to the string <code>"9,3,4,#,#,1,#,#,2,#,6,#,#"</code>, where # represents a null node.

Given a string of comma separated values, verify whether it is a correct preorder traversal serialization of a binary tree. Find an algorithm without reconstructing the tree.

Each comma separated value in the string must be either an integer or a character '#' representing null pointer.

You may assume that the input format is always valid, for example it could never contain two consecutive commas such as <code>"1,,3"</code>.

## Example

    Example 1:
    "9,3,4,#,#,1,#,#,2,#,6,#,#"
    Return true

    Example 2:
    "1,#"
    Return false

    Example 3:
    "9,#,#,1"
    Return false

## 思路
这里首先对字符串进行处理，将字符串中的数字全部替换为1（这里是为了了处理方便，当前的字符为current - 2，这里也完全可以去掉','，不过这里没有进行处理）。
进行与处理完成之后，对得到的新字符串进行从右往左进行处理，根据当前的字符以及下一个字符来进行处理。其中的number则是当前状态下需要的数字的个数，也就是
非叶子节点的个数。状态转换图如下：(突然就想起了以前学习的有限自动机，哈哈)
![状态转换图为：](http://obkkz880t.bkt.clouddn.com/state.png)

根据LeetCode的测试用例，添加了如下的一条状态转换（其中划掉的路径请忽略。。。）,之前一直认为最后出现的'#'最多为两个，不过在如下情况是可能出现更多的'#'
，所以要增加除了该部分的状态。

         3
        / \
       4   #
      / \
      # #

![状态转换图2为：](http://obkkz880t.bkt.clouddn.com/state2.png)

## 实现
根据状态转换图写出代码如下：

    class Solution {
        enum st {s, s1, s2};
        void del(string& s)
        {
            int i = 0, start = 1;
            for (; start < s.size(); ++start)
            {
                if (s[start] >= '0' && s[start] <= '9' && s[start - 1] >= '0' && s[start - 1] <= '9')
                {
                    s[start - 1] = '1';
                    break;
                }
            }
            for (i = start + 1; i < s.size(); ++i)
            {
                if (s[i] == '#' || s[i] == ',')
                    s[start++] = s[i];
                else if (s[i - 1] == '#' || s[i - 1] == ',')
                {
                    s[start++] = '1';
                }
            }
            s.resize(start);
            for (int j = 0; j < s.size(); ++j)
            {
                if (s[j] >= '0' && s[j] <= '9')
                    s[j] = '1';
            }
        }
    public:
            bool isValidSerialization(string preorder) {
            if (preorder == "#") return true;
            del(preorder);
            cout << preorder << endl;
            st state = s;
            int needNum = 0;
            for (int i = preorder.size() - 1; i >= 0; i -= 2)
            {
                char current = preorder[i], pre = i - 2 < 0 ? ' ' : preorder[i - 2];
                switch (state)
                {
                    case s:
                    if (current != '#' || pre != '#')
                        return false;
                    state = s1;
                    ++needNum;
                    i -= 2;
                    break;
                    case s1:
                    if (current == '#')
                        ++needNum;
                    else
                    {
                        state = s2;
                        --needNum;
                    }
                    break;
                    case s2:
                    if (current == '#' && pre == '1')
                    {
                        i -= 2;
                    } //is num
                    else if (current == '1')
                    {
                        if (needNum < 1)
                            return false;
                        --needNum;
                    } else if (current == '#' && pre == '#')
                    {
                        needNum += 2;
                        state = s1;
                        i -= 2;
                    }
                }
            }
            cout << needNum << endl;
            return needNum == 0;
        }

成功AC！时间复杂度O(n)，空间复杂度O(1).
