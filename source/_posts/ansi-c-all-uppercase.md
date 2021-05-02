---
title: ANSI C：判斷字串是否全大寫
tags:
  - ansi c
  - c
  - uppercase
categories:
  - Developments
date: 2020-10-23 11:10:29
---

程式碼：
<!-- more -->

```cpp
/* 感謝一位不具名朋友提供效能更佳的版本 (all_uppercase)! */
/* 舊版在 all_uppercase_v1() */

#include <string.h>
// 只有 all_uppercase_v1 需要 ctype.h
#include <ctype.h>

int all_uppercase(char* str) {
    int len = strlen(str);
    for (int i = 0; i < len; ++i)
        if (!(str[i] >= 'A' && str[i] <= 'Z'))
            return 0; 
    return 1;
}

int all_uppercase_v1(char* str) {
    int upper = 1;
    for (int i = 0; i<strlen(str); i++)
        if (!isupper(str[i])) upper = 0; 
    return upper;
}
```
