# Husky
markdown to html 

## 核心思想
利用正则表达式，筛选出我们想要的目标，将该目标替换成Html的语法。

-------------------------------------------
## 语法：标题

 语法形式如下：
```
# 这是 H1

## 这是 H2

### 这是 H3

```
用正则表达式来描述：
``(#{1,6})\s*(.*?)\s*$``
替换成html标签：
```
def process_header(text: str):
    pat = re.compile(r"^(#{1,6})\s*(.*?)\s*$", re.M)

    def rep(match):
        tags = match.group(1)  # type:str
        content = match.group(2)

        tag_start = '<h{}>'.format(str(len(tags)))
        tag_end = '</h{}>'.format(str(len(tags)))
        return tag_start + content + tag_end

    return pat.sub(repl=rep, string=text)
```

## 语法：强调
语法形式如下：
```
*single asterisks*

_single underscores_

**double asterisks**

__double underscores__
```
用正则表达式来描述：
``([*|_]{1,2})(.*)\1$``
替换成html标签：
```
def process_emphasize(text: str):
    pat = re.compile(r"([*|_]{1,2})(.*)\1", re.DOTALL)

    def rep(match):
        tags = match.group(1)  # type:str
        content = match.group(2)

        if len(tags) == 2:
            t = 'strong'
        elif len(tags) == 1:
            t = 'em'

        tag_start = '<{}>'.format(t)
        tag_end = '</{}>'.format(t)
        return tag_start + content + tag_end

    return pat.sub(repl=rep, string=text)

```





## 语法：列表
语法形式如下：
```
*   Red
*   Green
*   Blue
--------------
+   Red
+   Green
+   Blue
-------------
-   Red
-   Green
-   Blue
-------------
1.  Bird
2.  McHale
3.  Parish
```

用正则表达式来描述：
```
(
	([\s\t]*)([*|+|-]|\d+[.])[\s]+      # start
	(?s:.+?)  
	(\Z|       # end  (End of input)
	(?=(\n\s*\n*)+(?!([\s\t]*)([*|+|-]|\d+[.])[\s]))  # end  (下一段 非序列标志)  
	)                         
) 
```
替换成html标签：
```
def process_list(text: str):
    pat = re.compile(r"""
        (
            ([\s\t]*)([*|+|-]|\d+[.])[\s]+      # start
            (?s:.+?)  
            (\Z|       # end  (End of input)
            (?=(\n\s*\n*)+(?!([\s\t]*)([*|+|-]|\d+[.])[\s]))  # end  (下一段 非序列标志)  (?!([\s\t]*)([*|+|-]|\d+[.])[\s])
            )                         
        )   
        """, re.M | re.X)

    def rep(match):
        m_tag = match.group(3)
        if len(m_tag) == 1:
            h_tag = 'ul'
        else:
            h_tag = 'ol'
        result = process_list_one(match.group(1))
        return "<{tag}>{result}</{tag}>".format(tag=h_tag, result=result)

    return pat.sub(repl=rep, string=text)

----------------------------------------------------
def process_list_one(text: str):
    pat = re.compile(r"""
            (
                ([\s\t]*)([*|+|-]|\d+[.])[\s]+      # start
                ((?s:.+?))  
                ((?=([\s\t]*)([*|+|-]|\d+[.])[\s]+)| # end
                 \Z| 
                (?=(\n\s*\n*)+) 
                )                          
            )   
            """, re.M | re.X)

    def rep(match):
        return "<{tag}>{content}</{tag}>".format(tag='li', content=match.group(4))

    return pat.sub(repl=rep, string=text)

```

## 引用