---
title: 测试代码高亮
date: 2018-11-05 22:03
comments: true
tags:
- leetcode
- 算法
categories:
- 算法
---

> 测试代码高亮
    
```cpp
class Solution {
public:
    string longestPalindrome(string s)
    {
        int size  = s.size();
        if (size == 0 || size == 1)
        {
            return s;
        }
        int table[1000][1000];
        for(int i = 0; i < size; i++)
        {
            table[i][i] = true;
        }
        int maxlen = 1;
        for(int i = 0; i < size-1; i++)
        {
          if(s.at(i)==s.at(i+1))
          {
              table[i][i+1] = true;
              maxlen = 2;
          } else {
              table[i][i + 1] = false;
          }
        }

        for(int len = 3; len <= size; len++)
        {
            int right;
            for(int left = 1; left < size-(len-2); left++){
                right = left+(len-2)-1; // 子串的右界
                if(table[left][right] != 0){
                    if(s.at(left-1) == s.at(right+1))
                    {
                        maxlen = right-left+3;

                        table[left-1][right+1] = true;
                    } else {
                        table[left-1][right+1] = false;
                    }
                } else {
                    table[left-1][right+1] = false;
                }

            }
        }
        for(int i = 0; i < size-maxlen+1; i++){
            if(table[i][i+maxlen-1] != 0) {
                return s.substr(i, maxlen);
            }
        }
        return "";
    }
};
```