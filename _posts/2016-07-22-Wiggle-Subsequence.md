---
layout: post
title: leetcode 算法题 Wiggle Subsequence
date: 2016-07-22
categories: blog
tags:
description:
---
# leetcode Wiggle Subsequence

## 问题描述
A sequence of numbers is called a <b>wiggle sequence</b> if the differences between successive numbers strictly alternate between positive and negative. The first difference (if one exists) may be either positive or negative. A sequence with fewer than two elements is trivially a wiggle sequence.

For example, [1,7,4,9,2,5] is a wiggle sequence because the differences (6,-3,5,-7,3) are alternately positive and negative. In contrast, [1,4,7,2,5] and [1,7,4,5,5] are not wiggle sequences, the first because its first two differences are positive and the second because its last difference is zero.

Given a sequence of integers, return the length of the longest subsequence that is a wiggle sequence. A subsequence is obtained by deleting some number of elements (eventually, also zero) from the original sequence, leaving the remaining elements in their original order.

## Example

    Input: [1,7,4,9,2,5]  
    Output: 6  
    The entire sequence is a wiggle sequence.  

    Input: [1,17,5,10,13,15,10,5,16,8]  
    Output: 7  
    There are several subsequences that achieve this length. One is [1,17,10,13,10,16,8].  

    Input: [1,2,3,4,5,6,7,8,9]  
    Output: 2  

## 实现
采用贪心，对于连续递增的子窜选择最大的数字，对于连续递减的子窜选择其最小的数组。并将最开始为减小的问题转化为最开始为增大的问题。代码如下：

    class Solution {
        int wiggleMaxLengthInce(vector<int>& nums, int i, int j) {
            int count = 1;
            while (i + 1 < j)
            {
                int index = i;
                while (index + 1 < j && nums[index] <= nums[index + 1])
                    ++index;
                if (nums[i] != nums[index]) ++count;

                i = index;
                while (i + 1 < j && nums[i] >= nums[i + 1])
                    ++i;
                if (nums[i] != nums[index]) ++count;
            }
            return count;
        }
    public:
        int wiggleMaxLength(vector<int>& nums) {
            if (nums.size() <= 1) return nums.size();
            if (nums[0] <= nums[1]) return wiggleMaxLengthInce(nums, 0, nums.size());
            // 将开始为减少的问题转化为增加的问题
            int i = 1;
            while (i + 1 < nums.size() && nums[i] >= nums[i + 1])
            ++i;
            return wiggleMaxLengthInce(nums, i, nums.size()) + 1;
        }
    };
