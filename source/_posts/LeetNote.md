---
title: LeetNote
date: 2022-02-11 17:16:38
tags:
---

### [833. 字符串中的查找与替换](https://leetcode-cn.com/problems/find-and-replace-in-string/)

```c++
class Solution {
public:
    string findReplaceString(string s, vector<int>& indices, vector<string>& sources, vector<string>& targets) {
        int n = indices.size();
        //需要用map 对可能无序的indices进行一波排序
        map<int,pair<string,string>> m;
        for(int i =0;i<n;i++) m[indices[i]] = pair(sources[i],targets[i]);
        //注意这里的trick可以从右边开始替换左边的
        //而且注意没有from m.end() iter-- to m.brgin()的写法！
        for(auto iter = m.rbegin();iter != m.rend();iter++){
            int pos = iter->first;
            string s1 = iter->second.first;
            string s2 = iter->second.second;
            if(s.substr(pos,s1.size()) == s1) s = s.substr(0,pos) + s2 +s.substr(pos+s1.size());
        }
        return s;
    }
};
```

```c++
class Solution {
public:
    string findReplaceString(string s, vector<int>& indices, vector<string>& sources, vector<string>& targets) {
        int n = indices.size();
        unordered_map<int,int> m;
        for(int i=0;i<n;i++){
            m[indices[i]]=i;
        }
        sort(indices.begin(),indices.end());
        //需要用map 对可能无序的indices进行一波排序
        int offset = 0;
         for(int sIdx:indices){
            int new_pos = sIdx+offset;
            string a = sources[m[sIdx]];
            string b = targets[m[sIdx]];
            int asz = a.size();
            if(s.substr(new_pos,asz) == a) {
                s = s.substr(0,new_pos) + b +s.substr(new_pos + asz);
                offset += b.size() - asz;
            }
        }
        return s;
    }
};
```

### [1423. 可获得的最大点数](https://leetcode-cn.com/problems/maximum-points-you-can-obtain-from-cards/)

```c++
class Solution {
public:
    int maxScore(vector<int>& cardPoints, int k) {
        int n = cardPoints.size();
        //求n-k大小的滑动窗口中sum最小的值
        int Winsz = n-k;
        int minSum = accumulate(cardPoints.begin(),cardPoints.begin()+Winsz,0);
        int minALL = minSum;
        for(int i = Winsz;i < n;i++){
            minSum += (cardPoints[i] - cardPoints[i-Winsz]);
            minALL  = min(minSum,minALL);
        }
        return accumulate(cardPoints.begin(),cardPoints.end(),0) - minALL;
    }
};
```

### [410. 分割数组的最大值](https://leetcode-cn.com/problems/split-array-largest-sum/)

遇到最大化最小值或最小化最大值，就是二分查找

1.动态规划

![image-20220210170547790](C:\Users\yeqiuhan\AppData\Roaming\Typora\typora-user-images\image-20220210170547790.png)

```c++
class Solution {
public:
    int splitArray(vector<int>& nums, int m) {
        int n = nums.size();
        //dp[i][j]数组中存的是前i个数分成j段能得到的最大连续子数组和的最小值
        //对k进行枚举，前k个数被分为j-1段，而第k+1到第i个数为第j段
        vector<vector<long long>> dp(n + 1, vector<long long>(m + 1, LLONG_MAX));
        vector<long long> sub(n+1,0);
        dp[0][0] =0;
        for(int i=0;i<n;i++){
            sub[i + 1] = sub[i] +nums[i];
        }
        for(int i = 0;i<= n;i++)
            for(int j = 1; j <= min(i,m);j++)
                for(int k = 0;k < i;k++)
                    dp[i][j] = min(dp[i][j],max(dp[k][j-1],sub[i]-sub[k]));
        return dp[n][m];

    }
};
```

时间复杂度：O(n^2×m)，其中 n 是数组的长度，m 是分成的非空的连续子数组的个数。总状态数为O(n×m)，状态转移时间复杂度O(n)【这是因为最内层枚举k循环了O（N）次】，所以总时间复杂度为O(n^2 ×m)。

空间复杂度：O(n×m)，为动态规划数组的开销。

2.二分法

由于题目的返回要求：返回最小和的值，最小和mid必然落在 [max(nums), sum(nums)] 之间，我们可以使用二分来进行查找。猜mid值并使用贪心算法确定此值的正确性，直到猜到正确最小和的值。

我先猜一个mid值，然后遍历数组划分，使**每个子数组和都最接近mid**（贪心地逼近mid），这样我得到的**子数组数一定最少**;（因为要数组各自和的**最大值最小**，即数组数要尽可能小地划分，因此这里算最少的子数组）
如果即使这样子数组数量仍旧多于m个，那么明显说明我mid猜小了，因此 l = mid + 1;
而如果得到的子数组数量小于等于m个，那么我可能会想，mid是不是有可能更小？值得一试。因此 h = mid;

![image-20220210173444904](C:\Users\yeqiuhan\AppData\Roaming\Typora\typora-user-images\image-20220210173444904.png)

```c++
int splitArray(vector<int>& nums, int m) {
        long l = nums[0], h = 0;//int类型在这里不合适，因为h可能会超过int类型能表示的最大值
        for (auto i : nums)
        {
            h += i;
            l = l > i ? l : i;
        }
        while (l<h)
        {
            long mid = (l + h) / 2;
            long temp = 0;
            int cnt = 1;//初始值必须为1
            for(auto i:nums)
            {
                temp += i;
                if(temp>mid)
                {
                    temp = i;
                    ++cnt;
                }
            }
            if(cnt>m)
                l = mid + 1;
            else
                h = mid;
        }
        return l;
    }
```

