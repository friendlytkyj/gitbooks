# 常用语法

## 换行

1. 行尾加两个`空格` 
2. 使用`<br/>`标签

## 加粗

使用`**加粗**`

## 斜体

使用`*斜体*`

## Markdown支持的代码块类型

AppleScript, ActionScript 3.0, Shell, ColdFusion, C, C#, CSS, Delphi, diff&patch, Erlang, Groovy, Java, JavaFX, JavaScript, Perl, PHP, text, Python, Ruby, SASS&SCSS, Scala, SQL, Visual Basic, XML, Objective C, F#, R, matlab, swift, GO, 

## 导入文件内容

### 导入md文件
使用`{% include "./complex_field_desc.md" %}`
### 导入其它文件
使用`[import]``(../code/example.json)`

## 创建表格

一个简单的表格是这么创建的：

项目     | Value
:--------: | :-----:
电脑  | $1600
手机  | $12
导管  | $1

### 设定内容居中、居左、居右

使用`:---------:`居中  
使用`:----------`居左  
使用`----------:`居右  

| 第一列       | 第二列         | 第三列        |
|:-----------:| -------------:|:-------------|
| 第一列文本居中 | 第二列文本居右  | 第三列文本居左 |

### SmartyPants

SmartyPants将ASCII标点字符转换为“智能”印刷标点HTML实体。例如：  

|    TYPE   |ASCII                          |HTML
|:----------------:|:-------------------------------:|:-----------------------------:|
|Single backticks|`'Isn't this fun?'`            |'Isn't this fun?'            |
|Quotes          |`"Isn't this fun?"`            |"Isn't this fun?"            |
|Dashes          |`-- is en-dash, --- is em-dash`|-- is en-dash, --- is em-dash|