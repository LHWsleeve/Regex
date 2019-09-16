# Python 正则表达式  
---
## 1.预备知识
Python内置了正则表达式的package，使用之前必须导入 ***re***。
```
import re
````
re.rearsch(pattern， string)函数，如果pattern可以匹配string的某个字串，则返回一个MatchObject对象，否则返回None。  
若在正则表达式两端加上\A,\Z(注意都是大写)，可以判断整个字符串是否能由正则表达式匹配，进行正则验证。
```
re.rearch("\\d{6}", "12345678")!=None  #---→True
re.rearch("\A\\d{6}\Z", "12345678")!=None  #---→Flase
```
**注：**上例子没有 **r'** 原生字符串，需要进行转义，所以\\d{6}，某些代码中正则表达式```\d```没有转义。因为python在处理字符串转义时，如果字符串文字中的某个转义序列无法识别，则会被原封不动地保留下来，\d不是字符串转义序列，所以会被保留当成正则文字。

## 2.正则功能详解
### 2.1 列表
功能|记法|说明
-|-|-
字符组|[...] [^...]|支持unicode
POSIX字符组|[:digit:]|只匹配ascii
Unicode|\p{....}|不支持
字符组简记法|\d, \D, \w, \W, \s, \S|python2默认ascii，python3默认unicode
行起始位置|^,\A|支持
行结束位置|$, \z,\Z|不支持\z
单词边界|\b,\B|python2默认ascii，python3默认unicode
顺序环视|(?=...),(?!...)|支持
逆序环视|(?<=...)，(?<!...)|不支持变长模式
匹配模式| i m s x a u |支持 **注:** 模式对整个表达式生效，不论放在那里
模式作用范围|(?modifier)，失效(?-modifier)|只支持(?modifier)
纯文本模式|\Q..\E|不支持
捕获分组及引用|(...)\num|支持，同时支持命名分组
命名分组|(?P<name>...), (?P=name)|支持
非捕获分组|(?:...)|支持
多选结构|(...\|...)|支持
匹配优先量词|? * -|支持
忽略优先两次|?? *? +? {n,m}?|支持
#### 2.2.2 字符组
在python2中，一旦出现中文必须加u，因为python2默认ASCII编码，中文要用Unicode。Python3不存在问题。
#### 2.2.3 关于编码
对于python2，可以一次性将所有字符串都指定为Unicode
```
from_future_imnport unicode_literals
"不能接受的Unicode字符串".encode("utf-8")
```

#### *2.2.4 条件匹配*(特殊)
属于python特殊用法。语法：(?(id/name)yes-pattern|no-pattern)，其中(id/name)时对应捕获分组的名称或者编号，若捕获成功，则执行yes，否则执行no。no-pattern不是必选项，可以没有。
例如：如果前面没有美元$符号，则价格只能包含整数部分，否则还应包含两位小数
```
regex =ur"^(\$)?[0-9]+(?(\1)\.[0-9]{2}|)$"
re.search(regex, u"34")!=None # Ture
re.search(regex, u"12.00")!=None # Flase
re.search(regex, u"$34")!=None # Flase
re.search(regex, u"$12.00")!=None # Ture
```
<font color=#FF0000>在这个例子中，(\$)捕获分组\1,然后做条件匹配</font>

## 3 正则API简介
### 3.1 RegexObject
正则表达式常用方法是：```e.compile(pattern)```,如果要指定匹配模式或多个模式可以：
```re.compile((?is)pattern)``` 
```re.compile(pattern,re,I|re.S)```

***注：*** re.compile()提供了一个DEBUG功能，如果遇到复杂正则表达式，可以表明意义。其中缩进表示表达式各结构的层级，而字符本身则显示其码值的十进制表示。```re.co,pile("...",DEBUG)

### 3.2 MatchObject
调用re.search(),re.match()匹配成功，会返回一个MatchObject对象。
re.search(),re.match()很相似，唯一的区别在于re.match()如果匹配成功，那么pattern匹配的字符串必须开始于string的最左端，re.search()没有这个限制。即，re.match()只会在字符串开始位置尝试匹配，基本上等于自带 **^**,但是不带 **$**，所以不能直接拿来当验证，需要末尾加上\Z。推荐头加\A,尾加\Z。
### 3.3 re.findall(pattern, string[,flags])
flags:匹配模式
这个函数会一次性找出正则表达式pattern在string中的所有匹配，返回一个**列表**。
如果pattern中存在捕获分组，则返回列表的元素并非整个表达式所匹配的文本，而是 ***各个捕获分组所匹配文本构成的元组***

### 3.4 re.finditer(pattern, string[,flags])
如果文本内容过多，使用re.findall()，一次性找出所有匹配成本很高，所以使用这个函数。可以返回一个迭代器。 ***它可以从左到右的顺序逐个遍历每次皮u配的MatchObject对象***

### 3.5 re.spilt(pattern, string[,flags])
这个函数用正则表达式是pattern能够匹配的文本切分字符串string。返回一个列表，且匹配项不出现
如果真这个表达式在字符串的开头或结尾位置能够匹配，那么返回的列表开头和结尾会出现空字符串。
可以明确指定maxsplit，最大切分次数。
参数|说明
-|-
负数|不做切分
0|没有指定
正数|切分N次

### 3.6 re.sub(pattern, repl, string[,flags])
这个函数进行正则表达式替换，他将字符串string中正则表达式能匹配的部分替换为repl指定文本。
若要在repl字符串中引用之前某个捕获分组匹配的文本，则应当使用\num，并且记住repl前面加上 ***r*** ，否则要转义,<font color=red>\\\num</font>
***注：***
1.repl可以是一个函数，他接受一个MatchObject对象，返回一个string。
2.可以设定count参数，它指定替换操作最多能发生的次数。
