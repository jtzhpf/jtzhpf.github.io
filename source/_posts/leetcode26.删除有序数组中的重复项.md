---
title: leetcode 26. 删除有序数组中的重复项
category: CS&Maths
# id: 57
date: 2023-8-31 18:42:32
tags: 
  - leetcode
  - C
  - C++
  - Rust
#toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
#timeline: code  # 展示在时间线列表中
# gitalk: true # 启用 Gitalk 评论
---
# 题目描述

给你一个 **升序排列** 的数组 `nums` ，请你 **原地** 删除重复出现的元素，使每个元素 **只出现一次** ，返回删除后数组的新长度。元素的 **相对顺序** 应该保持 **一致** 。然后返回 `nums` 中唯一元素的个数。


考虑 `nums` 的唯一元素的数量为 `k` ，你需要做以下事情确保你的题解可以被通过：

更改数组 `nums` ，使 `nums` 的前 `k` 个元素包含唯一元素，并按照它们最初在 nums 中出现的顺序排列。`nums` 的其余元素与 `nums` 的大小不重要。
返回 `k` 。
判题标准:

系统会用下面的代码来测试你的题解:
```
int[] nums = [...]; // 输入数组
int[] expectedNums = [...]; // 长度正确的期望答案

int k = removeDuplicates(nums); // 调用

assert k == expectedNums.length;
for (int i = 0; i < k; i++) {
    assert nums[i] == expectedNums[i];
}
```
如果所有断言都通过，那么您的题解将被 **通过**。

 

示例 1：
```
输入：nums = [1,1,2]
输出：2, nums = [1,2,_]
解释：函数应该返回新的长度 2 ，并且原数组 nums 的前两个元素被修改为 1, 2 。不需要考虑数组中超出新长度后面的元素。
```
示例 2：
```
输入：nums = [0,0,1,1,1,2,2,3,3,4]
输出：5, nums = [0,1,2,3,4]
解释：函数应该返回新的长度 5 ， 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4 。不需要考虑数组中超出新长度后面的元素。
```

提示：

* `1 <= nums.length <= 3 * 10^4`
* `-10^4 <= nums[i] <= 10^4`
* `nums` 已按 **升序** 排列


# 解答
## Rust

```Rust
impl Solution {
    pub fn remove_duplicates(nums: &mut Vec<i32>) -> i32 {
        let mut slow=0;
        let mut fast=1;
        while fast < nums.len(){
            if nums[fast]!=nums[slow]{
                slow+=1;
                nums[slow]=nums[fast];
            }fast+=1;
        }
        (slow+1) as i32
    }
}
```

## C

```C
int removeDuplicates(int* nums, int numsSize){
    int *slow=nums,*fast=nums;
    while(fast<nums+numsSize){
        if(*fast!=*slow){
            slow++;
            *slow=*fast;
        }            
        fast++;
    }
    return slow-nums+1;
}
```

## C++

```C++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int slow=0,fast=0;
        while(fast<nums.size()){
            if(nums[fast]!=nums[slow])
                nums[++slow]=nums[fast];
            fast++;
        }
        return slow+1;
    }
};
```