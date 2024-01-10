---
title: leetcode 4. 寻找两个正序数组的中位数
category: CS&Maths
# id: 57
date: 2023-12-31 18:42:32
tags: 
  - leetcode
  - C++
  - C
#toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
#timeline: code  # 展示在时间线列表中
# gitalk: true # 启用 Gitalk 评论
---
# 题目描述

给定两个大小分别为 `m` 和 `n` 的正序（从小到大）数组 `nums1` 和 `nums2`。请你找出并返回这两个正序数组的 **中位数**。

算法的时间复杂度应该为 `O(log(m+n))`。


示例 1：
```
输入：nums1 = [1,3], nums2 = [2]
输出：2.00000
解释：合并数组 = [1,2,3] ，中位数 2
```

示例 2：
```
输入：nums1 = [1,2], nums2 = [3,4]
输出：2.50000
解释：合并数组 = [1,2,3,4] ，中位数 (2 + 3) / 2 = 2.5
```

提示：

* `nums1.length == m`
* `nums2.length == n`
* `0 <= m <= 1000`
* `0 <= n <= 1000`
* `1 <= m + n <= 2000`
* `-10^6 <= nums1[i], nums2[i] <= 10^6`





# 解答

## C

```C
#define max(a,b) (a>b?a:b)
double findMedianSortedArrays(int* nums1, int nums1Size, int* nums2, int nums2Size) {
    int total=nums1Size+nums2Size;
    int loc=1+(total>>1);// 左侧数字数量
    int flag=(1^total)&1;//if flag=0, only one number
    int p1=0,p2=0;
    // 先找到中位数的位置并切片，确保中位数在两个数组的左侧
    while(loc>0){
        int cnt1=min(nums1Size,(1+loc)/2);
        int cnt2=min(nums2Size,max((1+loc)/2,loc-cnt1));
        if(nums1Size&&nums2Size){
            if(nums1[p1+cnt1-1]<nums2[p2+cnt2-1]){
                p1=p1+cnt1;
                loc-=cnt1;
                nums1Size-=cnt1;
            }else{
                p2=p2+cnt2;
                loc-=cnt2;
                nums2Size-=cnt2;
            }
        }else if(!nums1Size){
            p2=p2+loc;
            nums2Size-=loc;
            loc=0;
        }else if(!nums2Size){
            p1=p1+loc;
            nums1Size-=loc;
            loc=0;
        }
    }

    // 从两个数组的左侧提取中位数
    if(p1-1>=0&&p2-1>=0){
        if(flag){
            if(nums1[p1-1]>nums2[p2-1]){
                if(p1-2>=0)
                    return 0.5*(nums1[p1-1]+(nums1[p1-2]>nums2[p2-1]?nums1[p1-2]:nums2[p2-1]));
                else 
                    return 0.5*(nums1[p1-1]+nums2[p2-1]);
            }else if(nums1[p1-1]<nums2[p2-1]){
                if(p2-2>=0)
                    return 0.5*(nums2[p2-1]+(nums2[p2-2]>nums1[p1-1]?nums2[p2-2]:nums1[p1-1]));
                else 
                    return 0.5*(nums2[p2-1]+nums1[p1-1]);
            }else{
                return nums1[p1-1];
            }
        }else{
            return max(nums1[p1-1],nums2[p2-1]);
        }
    }else if(p1-1>=0){
        if(flag && p1-2>=0)
            return 0.5*(nums1[p1-1]+nums1[p1-2]);
        else 
            return nums1[p1-1];
    }else if(p2-1>=0){
        if(flag && p2-2>=0)
            return 0.5*(nums2[p2-1]+nums2[p2-2]);
        else 
            return nums2[p2-1];
    }
    return 0;
}
```

