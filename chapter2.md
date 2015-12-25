# tdd数据格式

&emsp;&emsp;tdd是fmpp自己的一种数据格式，语法类似json,但又与json不同，它更像是json的扩充，还支持如下的格式：

* Strings need not be quoted if they doesn't look like a legal boolean or number value, and they don't contain:
    
    *** white-space: space, tab, line-break, etc.
    
    *** Quotation marks or apostrophe-quote: ", '
    
    *** Separator-like chars: comma (,), semicolon (;). Colon (:) is allowed without quoting the string if the string is not a key in a hash.
    
    *** Bracket-like chars: (, ), [, ], {, }, <, >
    
    *** Equals sign (=)
    
    *** Plus sign (+)

* Line-break can be used instead of comma (,). That is, in practice, you can omit commas that would be at the end of the lines.

* If in a hash the value is missing from a key:value pair, then the value defaults to boolean true.

&emsp;&emsp;所以以下两种写法是等同的：
```
{
    "user": "Big Joe",
    "tall": true,
    "animals": [
        {"name": "white mouse", "price": 30},
        {"name": "black mouse", "price": 25},
        {"name": "green mouse", "price": 150}
    ]
}

```

```
{
    user: "Big Joe"
    tall
    animals: [
        {name: "white mouse", price: 30}
        {name: "black mouse", price: 25}
        {name: "green mouse", price: 150}
    ]
}
```


## References:
1. [TDD - Textual Data Definition](http://fmpp.sourceforge.net/tdd.html)