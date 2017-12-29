---
title: Python正则表达式的5个小贴士
date: 2017-12-23 17:31:48
tags: python
categories: python
---


{% note info %}

正则表达式是一个非常强大的处理字符工具，但有时可读性很差、晦涩难懂，Jamie Zawinski 说道：

 >Some people, when confronted with a problem, think, “I know, I’ll use regular expressions.” Now they have two problems.
{% endnote %}

本来是一个问题，引入正则表达式之后就成了两个问题。其实并不是任何场景都需要正则表达式。
在简单场景，能用字符串自己提供的方法解决问题就没必要用正则表达式，比如字符替换

<!-- more -->


```python
>>> import re
>>> text = 'java is most popular language'

>>> re.sub(r'java', 'python', text)
'python is most popular language'

# good
>>> text.replace("java", "python")
'python is most popular language'
```

判断字符串是否以某字符开头
```python
>>> re.match(r"^java", text)
<_sre.SRE_Match object at 0x000000000471D578>
>>> text.startwith("java")

# good
>>> text.startswith("java")
True
```
**re.match()** 与 **re.search()**
`re.match` 从字符串的起始位置匹配，如果没匹配成功就不再往后匹配，返回 None。而 search 虽然也是从起始位置开始匹配，但是如果在起始位置没有匹配，就继续往后匹配，直到匹配为止，如果匹配到字符串末尾都没有匹配则返回 None
```python
>>> text = "java is most popular langauge"
>>> re.match("most", text) # 没匹配

# bad
>>> re.match(".*most", text) 
<_sre.SRE_Match object at 0x0000000004CCD578>

# good
>>> re.search("most", text)
<_sre.SRE_Match object at 0x000000000471D578>
```
**不分组的括号**
我们知道正则表达式中括号可以用于分组提取，有时我们并不希望括号用于分组该怎么办，答案是使用 `(?:)`
看一个例子，用正则表达式提取URL中的各个组成部分
{% asset_img python-rex.png %}

```python
rex = r'^(http[s]?)://([^/\s]+)([/\w\-\.]+[^#?\s]*)?(?:\?([^#]*))?(?:#(.*))?$'
print(re.match(rex, url).groups())
>>> ('http',
     'www.example.com', 
     '/path/to/myfile.html', 
     'key1=value1&key2=value2', 
     'SomewhereInTheDocument')
```
 
上面虽然写了7对括号，但其实只有5个分组。下面是不使用`?:`，出现了 7 组数据

```python
rex = r'^(http[s]?)://([^/\s]+)([/\w\-\.]+[^#?\s]*)?(\?([^#]*))?(#(.*))?$'
print(re.match(rex, url).groups())
>>>('http', 
    'www.example.com', 
    '/path/to/myfile.html', 
    '?key1=value1&key2=value2', 
    'key1=value1&key2=value2', 
    '#SomewhereInTheDocument', 
    'SomewhereInTheDocument')
```

**贪婪匹配**
正则表达式默认是贪婪匹配的，也就是说它会在满足匹配条件的情况下尽可能多的匹配字符，例如这里有一段话：
```python
html = """<div><p>Today a quick article on a nic</p><p>Read more ...</p></div>"""
```

里面有两对`<p>`标签，如果你只想匹配第一对，使用
```python
>>> re.search("<p>.*</p>", html)
>>> m = re.search("<p>.*</p>", html)
>>> m.group()
'<p>Today a quick article on a nic</p><p>Read more ...</p>'
>>>
<p>.*</p> 会从第一个<p>开始，匹配到最后一个</p>，如果要想尽可能少匹配则可以在元字符后面加 ?

>>> m = re.search("<p>.*?</p>", html)
>>> m.group()
'<p>Today a quick article on a nic</p>'
```

----

[原文地址：关于正则表达式的5个小贴士](https://mp.weixin.qq.com/s?__biz=MjM5MzgyODQxMQ==&mid=2650367680&idx=1&sn=2e8ef8bcf4dc176c46376508cb5a8fa7&chksm=be9cdd9489eb54822dc5993ff71050ca9011aff07fdf642b3eccdee7e20dc2efad9f21fb1a63&scene=0#rd)