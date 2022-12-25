---
title: Kettle
date: 2022-03-23 15:13:47
categories:
- 后端
tags:
- ETL
- Kettle
- JSON Path
---

# 一、Kettle安装

安装环境：Windows

下载地址：https://sourceforge.net/projects/pentaho/files/

![image-20221031160556335](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221031160556335.png)

![image-20221031160839419](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221031160839419.png)

![image-20221031160907682](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221031160907682.png)

# 二、执行SQL脚本

## 1、sql脚本只执行一次，且第一个执行

执行SQL脚本组件，默认会第一个执行,且只执行一次。

![image-20221031170103337](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221031170103337.png)



## 2、sql脚本只执行一次，且最后一个执行

需要使用 **阻塞**步骤，等待前面所有的步骤完成后，再执行。

需要注意的是，在执行SQL脚本中，需要勾选 **执行每一行**，否则还会第一个执行！

虽然达到了最后执行的要求，但是前面有多少条数据，就会执行多少次。

![image-20221031171529417](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221031171529417.png)

![image-20221031171655579](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221031171655579.png)

![image-20221031171756264](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221031171756264.png)

```
不让【执行SQL脚本】第一个执行而且不重复执行
问题描述： 在kettle的转换里面，除了正常的表输入表输出外还有一个sql脚本，要控制sql脚本的执行顺序，以及sql脚本的执行次数
```

https://blog.csdn.net/u012848709/article/details/67679366

![image-20221031161701097](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221031161701097.png)

# 三、Kettle-案例【发送Http请求】

案例：利用发送Http请求(1)后的结果集中的 url数组，对url进行修改后，再次遍历请求。

最后的结果是这样

![image-20220919204450863](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20220919204450863.png)



## 1、获取请求路径

可以从数据库中读取，也可以手动生成。这里用 【生成记录】组件

![image-20220919204722117](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20220919204722117.png)

## 2、发送Http请求

这里用的Get请求，利用【HTTP client】组件

![image-20220919205010849](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20220919205010849.png)

## 3、接收Http返回结果

因为返回结果为json字符串，所以用到【JSON 输入】组件

![image-20220919205117113](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20220919205117113.png)

因为json字符串一般会返回多个不同类型的字段，我们可以自定义返回字段，拿到我们想要的数据。

其中**路径** 中用到了 **JSON Path** 来筛选数据。 

例子中的 $.childUriList[*]  为拿到 childUriList数组中的每一个字符串

```json
{
"childUriList":
[
"XXXX",
"XXXX",
"XXXX"
]
}
```

![image-20220919205300096](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20220919205300096.png)

## 4、JSON Path 语法

1、什么市JSON Path

SONPath 之于 JSON，就如 XPath之于 XML。JSONPath 可以方便对 JSON 数据结构进行内容提取。

2、语法介绍

| **JsonPath**     | **说明**                                                     |
| ---------------- | ------------------------------------------------------------ |
| $                | 文档根元素                                                   |
| @                | 当前元素                                                     |
| .`或`[]          | 匹配下级元素                                                 |
| ..               | 递归匹配所有子元素                                           |
| *                | 通配符，匹配下级元素                                         |
| []               | 下标运算符，根据索引获取元素，**XPath索引从1开始，JsonPath索引从0开始** |
| [,]              | 连接操作符，将多个结果拼接成数组返回，可以使用索引或别名     |
| [start:end:step] | 数据切片操作，XPath不支持                                    |
| ?()              | 过滤表达式                                                   |
| ()               | 脚本表达式，使用底层脚本引擎，XPath不支持                    |

这是一个非常经典的案例，你几乎可以在任何介绍JSON Path的地方看到它，但是很实用。

```JSON
{ "store": {
    "book": [ 
      { "category": "reference",
        "author": "Nigel Rees",
        "title": "Sayings of the Century",
        "price": 8.95
      },
      { "category": "fiction",
        "author": "Evelyn Waugh",
        "title": "Sword of Honour",
        "price": 12.99
      },
      { "category": "fiction",
        "author": "Herman Melville",
        "title": "Moby Dick",
        "isbn": "0-553-21311-3",
        "price": 8.99
      },
      { "category": "fiction",
        "author": "J. R. R. Tolkien",
        "title": "The Lord of the Rings",
        "isbn": "0-395-19395-8",
        "price": 22.99
      }
    ],
    "bicycle": {
      "color": "red",
      "price": 19.95
    }
  }
}
```

配合示例，对应上面表格的语法，下面的举例更好理解

| **JsonPath**           | **Result**                               |
| ---------------------- | ---------------------------------------- |
| $.store.book[*].author | 所有book的author节点                     |
| $..author              | 所有author节点                           |
| $.store.*              | store下的所有节点，book数组和bicycle节点 |
| $.store..price         | store下的所有price节点                   |
| $..book[2]             | 匹配第3个book节点                        |
| $..book[(@.length-1)]  | 匹配倒数第1个book节点                    |
| $..book[(-1:)]         | 匹配倒数第1个book节点                    |
| $..book[0,1]           | 匹配前两个book节点                       |
| $..book[:2]            | 匹配前两个book节点                       |
| $..book[?(@.isbn)]     | 过滤含isbn字段的节点                     |
| $..book[?(@.price<10)] | 过滤`price<10`的节点                     |
| $..*                   | 递归匹配所有子节点                       |

这个地址可以在线运行 http://jsonpath.com/

## 5、拼接url字符串

拿到结果集后，在每个url字符串后进行拼接处理。这里为了方便，用到了【Java Script代码】组件

因为路径中有**中文**，为了方便再次请求，需要对url进行编码操作 encodeURI，否则http请求失败。

![image-20220919211027415](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20220919211027415.png)

这里也可以使用 【字符串操作】组件

![image-20220919211359727](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20220919211359727.png)

**Lower/Upper**：就是简单的大小写转换，当然只是针对于英文字符的，汉字和数字是无效的。

**Padding**：追加字符串，可以选择头部追加(left),或者尾部追加(right)，不过这个是要配合“Pad char”和“Pad length”使用的。

**Pad char**：只需要再输入框输入想要添加的字符串即可。

**Pad length**：输入长度，这个可以这么理解，假如说未追加内容的字符串是abc长度为3，想要追加的字符串是wq，是以right的方式追加的，如果输入的长度是3，那么结果是没有任何改变的；如果输入的长度是4，那么结果就是abcw；输入的是5，那结果就是abcwq；输入的长度是6，那结果就是abcwqw，相信说到这里应该就很清楚了，不管“Padding”选择的是right还是left，规则都是一样的。

InitCap：这个的作用就是保证字符串的首字母大写其余的字母都是小写，比如有一个字符串是aBC，如果InitCap的参数选择了“是”，那么结果就是“Abc”。

**Escape**：这个一般应用的较少，比如说选择了其中的“Use CDATA”，那么最后出来的字符串就是CDATA格式的字符串“<![CDATA[test_str]]>”，标红的部分就是原本的字符串，其它的就不一一介绍了，感兴趣的实验一下就明白了。

Digits：digits本身就是数字的意思，而选择的参数只有三两个“none、”“only”和“remove”，none就是不操作；only就是只保留数字，其他的字符不要；remove就是其他的字符都留下，只将数字字符去除掉。

**Remove Special character**：这个就很简单了，就是去除特殊字符，可以根据需求选择需要删除的字符，如空格(space)、换行符(line feed)等，这里就不一一介绍了，如有疑惑可以百度翻译。

## 6、结束

后面的再次请求，就和前面的操作重复了，就不再描述了。到这里一个简单的http请求就完成了。