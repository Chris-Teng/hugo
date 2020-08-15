---
title: '【moectf】mess writeup'
date: 2020-8-5
tags: [ctf]
---

<a name="K2Ujw"></a>
# 0x 00 题
puzzle.py:
```python
import random
flag = 'moectf{xxxxxxxxxxx}'
digit = ''

for i in flag:
    digit += str(ord(i))
i = 0
while i < len(digit):
    n = random.randint(0, 128)
    if ord('a') <= n <= ord('z') or ord('A') <= n <= ord('Z'):
        digit = digit[0:i] + chr(n) + digit[i:]
    i += 1

with open('puzzle1.txt', 'w') as out:
    out.write(digit)

```
puzzle.txt:
```
1091111A01ruVJl99hw11Qv6i102xCYC1c2B31DIsz1tm212l11A1l610448re11BQ09549115951n154V895F115d49109h1m1210810j11w2A5
```
<a name="BOxTi"></a>
# 0x 01 构思
原题的加密方案为：将一段明文转换为ASCII码数字，并组合为一串数字字符串。<br />//此步解题难点在于有意义的分组数字混合为无意义无分组的数字字符串，解密时需要分割为各字符的ASCII码<br />接下来虽然用到随机数模块，但其实是循环生成字母并插入到上述数字字符串中；抓住要点：字符 插入 数字字符串<br />也就是解密时把数字字符串中的所有字母过滤即可得到原先的数字字符串<br />这里我偷懒了，看了一眼ASCII码表，'!' 及以下的字符一般都不会涉及。所以直接 stri >= '!' ，就算有偏差，也可以手工修正
<a name="ZEbGx"></a>
# 0x 02 撰写解题脚本
```python
codec = '1091111A01ruVJl99hw11Qv6i102xCYC1c2B31DIsz1tm212l11A1l610448re11BQ09549115951n154V895F115d49109h1m1210810j11w2A5'

encoded = ''
for i in codec:
   if not('a' <= i <= 'z' or 'A' <= i <= 'Z'):
        encoded += i

i=0
j=2
decoded=''
while j<=len(encoded):
    stri = chr(int(encoded[i:j]))
    #'a' <= stri <= 'z' or 'A' <= stri <= 'Z' or stri=='{'or stri=='}' or stri=='_' or '0'<=stri or stri <='9'
    #原来的判断条件，很蠢
    if stri>='!': 
        decoded+=stri
        i=j
        j=j+2
    elif j-i<=3:
        j+=1
    else:
        continue

print(decoded)
```


