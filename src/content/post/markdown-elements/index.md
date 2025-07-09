---
title: "Markdown元素"
description: "展示多种不同的Markdown元素"
publishDate: "2023-02-22"
seriesId: "markdown-elements"
orderInSeries: 1
tags: ["markdown"]
draft: true
---
```md
# 一级标题
```
# 一级标题
```md
## 二级标题
```
## 二级标题
```md
### 三级标题
```
### 三级标题
```md
#### 四级标题
```
#### 四级标题
```md
##### 五级标题
```
##### 五级标题
```md
###### 六级标题
```
###### 六级标题

## 水平分割线
```md
---

---
```
---

---

## 文字强调
```md
**加粗文本**
```
**加粗文本**
```md
_斜体文本_
```
_斜体文本_
```md
~~删除线~~
```
~~删除线~~

## 引号
```md
"双引号" 和 '单引号'
```
"双引号" 和 '单引号'

## 块引用
```md
> 块引用可以嵌套...
>
> > ...通过连续使用大于号实现...
```
> 块引用可以嵌套...
>
> > ...通过连续使用大于号实现...

## 参考文献
```md
包含可点击参考文献[^1]的示例，链接到来源。

第二个包含参考文献[^2]的示例，链接到来源。

[^1]: 第一个脚注，包含返回内容链接。
[^2]: 第二个脚注，包含链接。

如果你查看`src/content/post/markdown-elements/index.md`中的示例，会发现通过[remark-rehype](https://github.com/remarkjs/remark-rehype#options)插件，"脚注"标题和参考文献会自动添加到页面底部。
```
包含可点击参考文献[^1]的示例，链接到来源。

第二个包含参考文献[^2]的示例，链接到来源。

[^1]: 第一个脚注，包含返回内容链接。
[^2]: 第二个脚注，包含链接。

如果你查看`src/content/post/markdown-elements/index.md`中的示例，会发现通过[remark-rehype](https://github.com/remarkjs/remark-rehype#options)插件，"脚注"标题和参考文献会自动添加到页面底部。

## 列表

### 无序列表
```md
- 使用`+`、`-`或`*`开始一行创建列表
- 通过缩进2个空格创建子列表：
  - 标记字符变化会强制开始新列表：
    - Ac tristique libero volutpat at
    - Facilisis in pretium nisl aliquet
    - Nulla volutpat aliquam velit
- 非常简单！
```
- 使用`+`、`-`或`*`开始一行创建列表
- 通过缩进2个空格创建子列表：
  - 标记字符变化会强制开始新列表：
    - Ac tristique libero volutpat at
    - Facilisis in pretium nisl aliquet
    - Nulla volutpat aliquam velit
- 非常简单！

### 有序列表
```md
可以使用连续数字...

1. 可以使用连续数字...
2. Lorem ipsum dolor sit amet
3. Consectetur adipiscing elit
4. Integer molestie lorem at massa

或者全部使用`1.`

1. 或者全部使用`1.`
1. 或者全部使用`1.`

从指定数字开始：

57. foo
1. bar
```
可以使用连续数字...

1. 可以使用连续数字...
2. Lorem ipsum dolor sit amet
3. Consectetur adipiscing elit
4. Integer molestie lorem at massa

或者全部使用`1.`

1. 或者全部使用`1.`
1. 或者全部使用`1.`

从指定数字开始：

57. foo
1. bar

## 代码
### 行内代码
```md
行内`代码`
```

### 缩进代码
    // 一些注释
    第一行代码
    第二行代码
    第三行代码

### 代码块
```
示例文本...
```


### 语法高亮
```js
var foo = function (bar) {
	return bar++;
};

console.log(foo(5));
```

### Rehype Pretty Code

添加标题

```js title="file.js"
console.log("标题示例");
```

Bash终端

```bash
echo "基础终端示例"
```

代码行高亮

```js title="line-markers.js" {7} {4-5}#add {3}#remove
function demo() {
    console.log("这行是普通显示");
    console.log("这行标记为删除");
    // 这行和下一行标记为新增
    console.log("这是第二行新增内容");

    return "这行使用中性默认标记类型";
}
```

行号显示

```python title="line-numbers.py" showLineNumbers
def greet(name):
    print("你好！")
    print(f"欢迎, {name}!")
    print("很高兴见到你。")
    return name
```

[Rehype Pretty Code](https://rehype-pretty.pages.dev/)是一个功能强大的工具，支持[自定义配置](https://rehype-pretty.pages.dev/)。

## 表格

| 选项   | 描述                                                                 |
| ------ | ------------------------------------------------------------------- |
| data   | 数据文件路径，用于提供传递给模板的数据。                             |
| engine | 用于处理模板的引擎。默认使用Handlebars。                             |
| ext    | 目标文件使用的扩展名。                                               |

### 表格对齐

| 商品         | 价格  | 库存数量 |
| ------------ | :---: | -------: |
| 多汁苹果     | 1.99  |      739 |
| 香蕉         | 1.89  |        6 |

### 键盘元素

| 操作                | 快捷键                                   |
| ------------------- | ---------------------------------------- |
| 垂直分割            | <kbd>Alt+Shift++</kbd>                   |
| 水平分割            | <kbd>Alt+Shift+-</kbd>                   |
| 自动分割            | <kbd>Alt+Shift+d</kbd>                   |
| 切换分割区域        | <kbd>Alt</kbd> + 方向键                  |
| 调整分割区域大小    | <kbd>Alt+Shift</kbd> + 方向键            |
| 关闭分割区域        | <kbd>Ctrl+Shift+W</kbd>                  |
| 最大化面板          | <kbd>Ctrl+Shift+P</kbd> + 切换面板缩放   |

## 图片

同文件夹中的图片：`src/content/post/markdown-elements/logo.png`

![Astro主题citrus标志](./logo.png)

## 链接

[markdown-it内容](https://markdown-it.github.io/)