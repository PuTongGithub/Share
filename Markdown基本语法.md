# Markdown基本语法

因为我打算用Markdown格式作为记录我内容的基本格式，所以我的首篇文章就来归纳整理一下Markdown的一些基本语法吧。



## 一、标题

标题设置是通过在标题文字前加上数个井号（#）和一个空格来实现的。

示例：

```
## 二级标签

#### 四级标签

###### 六级标签
```

效果：

## 二级标签

#### 四级标签

###### 六级标签



## 二、字体

字体设置是通过在文字两侧加上星号（*）下划线（_）或是波浪号（~）来实现的。

示例：

```
*斜体*
_斜体_
**加粗**
__加粗__
***斜体加粗***
__*斜体加粗*__
~~删除线~~
```

效果：

*斜体*
_斜体_
**加粗**
__加粗__
***斜体加粗***
__*斜体加粗*__
~~删除线~~



## 三、引用

引用设置是通过在引用文字前加上小于号（>）来实现的。

示例：

```
>content level one
>>content level tow
>
>content level one end
```

效果：
>content level one
>>content level tow
>
>content level one end



## 四、分割线

分割线可以用三个以上的星号（*）、减号（-）、下划线（_）来实现。

示例：

```
***
- - - -
____________
```

效果：

***
- - - -
____________



## 五、代码

代码块采用反单引号（`）来实现。本文中所有示例都采用了代码块。

示例：

```
`单行代码`

​```
代码块
​```
```

效果：

`单行代码`

```
代码块
```



## 六、列表

列表分为有序列表与无序列表。

### 有序列表

使用数字加英文句点（.）实现。

示例：

```
1.  Bird
2.  Cat
3.  Dog
```

效果：

1.  Bird
2.  Cat
3.  Dog

### 无序列表

使用星号（*）、加号（+）、减号（-）均可实现。

示例：

```
*   Red
+   Green
-   Blue
```

效果：

*   Red
+   Green
-   Blue



## 七、表格

表格通过竖线（|）、减号（-）、冒号（:）组合构成，具体语法说明如下：

> ```
> 表头|表头|表头
> ---|:--:|---:
> 内容|内容|内容
> 内容|内容|内容
> 
> 第二行分割表头和内容。
> - 有一个就行，为了对齐，多加了几个
> 文字默认居左
> -两边加：表示文字居中
> -右边加：表示文字居右
> 注：原生的语法两边都要用 | 包起来。此处省略
> ```

示例：

```
|表头1|表头2|表头3|
|-|:-:|-:|
|左对齐|居中|右对齐|
|另一行|另一行|另一行|
```

效果：

| 表头1  | 表头2  |  表头3 |
| ------ | :----: | -----: |
| 左对齐 |  居中  | 右对齐 |
| 另一行 | 另一行 | 另一行 |



## 八、图片

图片通过以下语法引入：

> ```
> ![Alt text](/path/to/img.jpg "Optional title")
> ```

示例：

```
![bing](http://searchengineland.com/figz/wp-content/seloads/2016/01/bing-new-logo-1920.jpg "bing logo")
```

效果：

![bing](http://searchengineland.com/figz/wp-content/seloads/2016/01/bing-new-logo-1920.jpg "bing logo")



## 九、超链接

超链接通过以下语法引入：

> ```
> [an example](http://example.com/ "Optional title")
> ```

示例：

```
点击[这里](https://www.bing.com/)访问bing主页
```

效果：

点击[这里](https://www.bing.com/)访问bing主页



###### 内容参考链接

1. [Markdown 编辑器语法指南](https://segmentfault.com/markdown)
2. [Markdown基本语法-简书](https://www.jianshu.com/p/191d1e21f7ed)
3. [教程-Markdown](http://www.markdown.cn/)
4. 我所使用的Markdown编辑器[typora](https://typora.io/)