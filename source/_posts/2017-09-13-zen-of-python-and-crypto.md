---
title: Python之禅的密码学原理
date: 2017-09-13 09:49:38
keywords: python, zen, cryptography, python之禅, 密码学
tags:
 - python
 - cryptography
---

# Python之禅

在Python命令行运行`import this`，会打印出python之禅

```
In [1]: import this
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

而运行`print(this.s)`，则会打印出一段"乱码":

<!-- more -->

```
In [2]: print(this.s)
Gur Mra bs Clguba, ol Gvz Crgref

Ornhgvshy vf orggre guna htyl.
Rkcyvpvg vf orggre guna vzcyvpvg.
Fvzcyr vf orggre guna pbzcyrk.
Pbzcyrk vf orggre guna pbzcyvpngrq.
Syng vf orggre guna arfgrq.
Fcnefr vf orggre guna qrafr.
Ernqnovyvgl pbhagf.
Fcrpvny pnfrf nera'g fcrpvny rabhtu gb oernx gur ehyrf.
Nygubhtu cenpgvpnyvgl orngf chevgl.
Reebef fubhyq arire cnff fvyragyl.
Hayrff rkcyvpvgyl fvyraprq.
Va gur snpr bs nzovthvgl, ershfr gur grzcgngvba gb thrff.
Gurer fubhyq or bar-- naq cersrenoyl bayl bar --boivbhf jnl gb qb vg.
Nygubhtu gung jnl znl abg or boivbhf ng svefg hayrff lbh'er Qhgpu.
Abj vf orggre guna arire.
Nygubhtu arire vf bsgra orggre guna *evtug* abj.
Vs gur vzcyrzragngvba vf uneq gb rkcynva, vg'f n onq vqrn.
Vs gur vzcyrzragngvba vf rnfl gb rkcynva, vg znl or n tbbq vqrn.
Anzrfcnprf ner bar ubaxvat terng vqrn -- yrg'f qb zber bs gubfr!
```

这段"乱码"的结构看上去和Python之禅一样，在[Python源码中的`Lib/this.py`](https://github.com/python/cpython/blob/master/Lib/this.py)里，有一段解码程序:

```
d = {}
for c in (65, 97):
    for i in range(26):
        d[chr(i+c)] = chr((i+13) % 26 + c)

print("".join([d.get(c, c) for c in s]))
```

# 密码学解释

Python之禅使用了一种简单的对称加密方法，即替换密码。

替换密码的基本思路是用一个符号替换另外一个符号，比如用`h`代替`a`，用`x`代替`b`等。

替换密码有一个特例，叫移位密码，也叫凯撒密码，这种密码是用某字符后面的第n个字符替换该字符。比如当`n=1`时，是用`b`代替`a`，用`c`代替`b`等等，用什么来代替`z`呢，当然是`a`了。

替换密码的加密方法是:
```
y = (x + n) % 26  # x，y是明文和密文在26个字母中的位置
```
解密方法是
```
y = (x - n) % 26
```

Python之禅用的就是移位密码，将大小写字母分别由其后面的第13个字母所替换，即用`n`代替`a`，用`o`代替`b`，以此类推。

上面的解码程序构建了一个解密字典`d`，表示了原文和密文的对应关系，有趣的是，这个字典`d`即可以作为加密字典，也可以作为解密字典。

也就是说，经过这段代码的处理，原文可以变成密文，密文还能变回原文。因为这个字典是这样的(这里只给出小写字母):
```
{
    'a': 'n',
    'b': 'o',
    'c': 'p',
    'd': 'q',
    'e': 'r',
    'f': 's',
    'g': 't',
    'h': 'u',
    'i': 'v',
    'j': 'w',
    'k': 'x',
    'l': 'y',
    'm': 'z',
    'n': 'a',
    'o': 'b',
    'p': 'c',
    'q': 'd',
    'r': 'e',
    's': 'f',
    't': 'g',
    'u': 'h',
    'v': 'i',
    'w': 'j',
    'x': 'k',
    'y': 'l',
    'z': 'm',
}
```

让我们来分析一下原因，首先其加密的方法是:
```
y = (x + 13) % 26
```

根据数论，我们可以说`x + 13`与`y`模26同余，即:
```
y ≡ x + 13 (mod 26)
```

其中符号`≡`表示同余，我们把等式两边减26，得到:
```
y - 26 ≡ x - 13 (mod 26)
```

又因为
```
y - 26 ≡ y (mod 26)
```

得到如下公式,即解密公式
```
y ≡ x - 13 (mod 26)
```

本质上偏移为13的加解密公式为同一个公式，他们生成的加解密字典是一样的，这也就解释了刚才的问题。

# 扩展

如果只给了一段加密信息(这里只讨论替换密码)，不知道加解密算法，这段信息应该怎么破解呢。

有两个方法:
1. 暴力破解，即把所有替换的情况都尝试一遍，这个办法基本不可行。
2. 字母频率分析，根据字母出现的频率猜测

## 字母频率分析

把文章中出现的字母进行统计并排序，和[英语字母频率](https://en.wikipedia.org/wiki/Letter_frequency#Relative_frequencies_of_letters_in_the_English_language)对比，根据排序猜测出对应的字母。

Python之禅的密文字母统计结果是(为了简化，统一按小写字母处理):
```
r        92  0.135894
g        79  0.116691
v        53  0.078287
n        53  0.078287
f        46  0.067947
b        43  0.063516
a        42  0.062038
y        33  0.048744
e        33  0.048744
u        31  0.045790
c        22  0.032496
o        21  0.031019
h        21  0.031019
l        17  0.025111
p        17  0.025111
q        17  0.025111
z        16  0.023634
s        12  0.017725
t        11  0.016248
k         6  0.008863
i         5  0.007386
j         4  0.005908
x         2  0.002954
m         1  0.001477
```

其中第一列是字母，第二列是出现次数，第三列是频率。

英语字母频率表中的字母出现频率前几位分别是
```
e	12.702%
t	9.056%
a	8.167%
o	7.507%
i	6.966%
n	6.749%
s	6.327%
h	6.094%
r	5.987%
d	4.253%
l	4.025%
c	2.782%
u	2.758%
```

通过对比，`r`和`e`、`g`和`t`都可以匹配上，后面的几个字母不能严格安装字母频率顺序匹配，毕竟密文的样本量有限，但是字母频率表极大缩小了密码猜测的范围，效率比暴力破解法高很多。

# 结论

这种加密方法基本上没有什么可靠性，仅供学习和娱乐。
