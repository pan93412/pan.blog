---
title: ANSI C：判斷字串是否全大寫
tags:
  - ansi c
  - c
  - uppercase
id: '335'
categories:
  - - Developments
date: 2020-10-23 11:10:29
---

/\* 感謝一位不具名朋友提供效能更佳的版本 (all\_uppercase)! \*/
/\* 舊版在 all\_uppercase\_v1() \*/

#include <string.h>
// 只有 all\_uppercase\_v1 需要 ctype.h
#include <ctype.h>

int all\_uppercase(char\* str) {
    int len = strlen(str);
    for (int i = 0; i < len; ++i)
        if (!(str\[i\] >= 'A' && str\[i\] <= 'Z'))
            return 0; 
    return 1;
}

int all\_uppercase\_v1(char\* str) {
    int upper = 1;
    for (int i = 0; i<strlen(str); i++)
        if (!isupper(str\[i\])) upper = 0; 
    return upper;
}