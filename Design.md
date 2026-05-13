# Axn Style Sheet (`.ass`) 完整规范

## 目录

1. [设计哲学](#设计哲学)
2. [语法规范](#语法规范)
3. [选择器](#选择器)
4. [属性大全](#属性大全)
5. [值类型与单位](#值类型与单位)
6. [盒模型](#盒模型)
7. [布局模式](#布局模式)
8. [定位系统](#定位系统)
9. [伪类与伪元素](#伪类与伪元素)
10. [@规则](#规则)
11. [优先级与级联](#优先级与级联)
12. [Python API](#python-api)
13. [完整示例](#完整示例)

---

## 设计哲学

**ASS (Axn Style Sheet)** 是 CSS 2.1 的完整子集实现，专为 Axn-Plus 视觉小说引擎设计。

### 核心原则

1. **标准优先**：语法与 CSS 2.1 保持一致，降低学习成本
2. **性能优先**：针对游戏 UI 场景优化，避免复杂计算
3. **可嵌入**：提供完整的 C++ 核心 + Python 绑定
4. **渐进增强**：支持 CSS 2.1 全部特性，为后续扩展留空间

### 与标准 CSS 的差异

| 特性 | CSS 2.1 | ASS | 原因 |
|------|---------|-----|------|
| 媒体查询 | ✅ | ❌ | 游戏 UI 不需要响应式 |
| 分页媒体 | ✅ | ❌ | 无打印需求 |
| 听觉样式 | ✅ | ❌ | 游戏用音频引擎独立控制 |
| @import | ✅ | ✅ | 支持，编译期展开 |
| 自定义字体 | ✅ | ✅ | 通过 `@font-face` |
| 计数器 | ✅ | ❌ | 复杂度高，游戏用变量替代 |

---

## 语法规范

### 基本语法

```css
/* 注释：CSS 风格的注释 */
selector {
    property: value;
    another-property: value2;
}

/* 多选择器共享样式 */
selector1, selector2 {
    property: value;
}
```

### 大小写

- 选择器：**区分大小写**（与 XML/HTML 一致）
- 属性名：**小写**
- 属性值：按类型区分（关键字小写，字符串保留原样）

### 字符编码

默认 UTF-8，支持 `@charset "UTF-8";` 声明

---

## 选择器

### 基础选择器

| 模式 | 描述 | 示例 |
|------|------|------|
| `*` | 通配符，匹配所有元素 | `* { margin: 0; }` |
| `element` | 类型选择器 | `button { color: red; }` |
| `.class` | 类选择器 | `.primary { background: blue; }` |
| `#id` | ID 选择器 | `#submit-btn { width: 100px; }` |

### 组合选择器

| 模式 | 描述 | 示例 |
|------|------|------|
| `A B` | 后代选择器 | `panel button { }` |
| `A > B` | 子元素选择器 | `menu > item { }` |
| `A + B` | 相邻兄弟选择器 | `label + input { }` |
| `A ~ B` | 通用兄弟选择器 | `h1 ~ p { }` |

### 属性选择器

| 模式 | 描述 | 示例 |
|------|------|------|
| `[attr]` | 存在指定属性 | `[disabled] { opacity: 0.5; }` |
| `[attr=value]` | 属性精确匹配 | `[type="button"] { }` |
| `[attr~=value]` | 属性值包含指定词（空格分隔） | `[class~="highlight"] { }` |
| `[attr\|=value]` | 属性值以 value 开头后跟连字符 | `[lang\|="en"] { }` |

### 伪类选择器

| 伪类 | 描述 | 示例 |
|------|------|------|
| `:link` | 未访问链接 | `a:link { }` |
| `:visited` | 已访问链接 | `a:visited { }` |
| `:hover` | 鼠标悬停 | `button:hover { }` |
| `:active` | 激活状态（鼠标按下） | `button:active { }` |
| `:focus` | 获得焦点 | `input:focus { }` |
| `:first-child` | 第一个子元素 | `li:first-child { }` |
| `:last-child` | 最后一个子元素 | `li:last-child { }` |
| `:only-child` | 唯一子元素 | `div:only-child { }` |
| `:empty` | 无子元素 | `div:empty { display: none; }` |
| `:root` | 根元素 | `:root { font-size: 16px; }` |

### 伪元素

| 伪元素 | 描述 | 示例 |
|------|------|------|
| `::first-line` | 首行 | `p::first-line { font-weight: bold; }` |
| `::first-letter` | 首字母 | `p::first-letter { font-size: 200%; }` |
| `::before` | 之前插入内容 | `.required::before { content: "*"; }` |
| `::after` | 之后插入内容 | `.link::after { content: " →"; }` |

---

## 属性大全

### 1. 背景与颜色

| 属性 | 值 | 默认 | 继承 |
|------|---|------|------|
| `color` | `<color>` | 由 UA 决定 | ✅ |
| `background-color` | `<color> \| transparent` | transparent | ❌ |
| `background-image` | `<uri> \| none` | none | ❌ |
| `background-repeat` | repeat \| repeat-x \| repeat-y \| no-repeat | repeat | ❌ |
| `background-attachment` | scroll \| fixed | scroll | ❌ |
| `background-position` | `<position>` | 0% 0% | ❌ |
| `background` | 简写 | - | ❌ |

**`<position>` 语法**：
```
[ [ <percentage> | <length> | left | center | right ] [ <percentage> | <length> | top | center | bottom ]? ] | [ [ left | center | right ] || [ top | center | bottom ] ]
```

示例：
```css
background-position: top;
background-position: 25% 75%;
background-position: 10px 20px;
background-position: right center;
```

### 2. 字体

| 属性 | 值 | 默认 | 继承 |
|------|---|------|------|
| `font-family` | `<family-name> [, <family-name>]*` | UA 决定 | ✅ |
| `font-size` | `<absolute-size> \| <relative-size> \| <length> \| <percentage>` | medium | ✅ |
| `font-weight` | normal \| bold \| 100-900 | normal | ✅ |
| `font-style` | normal \| italic \| oblique | normal | ✅ |
| `font-variant` | normal \| small-caps | normal | ✅ |
| `font` | 简写 | - | ✅ |

**绝对尺寸**：xx-small, x-small, small, medium, large, x-large, xx-large
**相对尺寸**：larger, smaller

### 3. 文本

| 属性 | 值 | 默认 | 继承 |
|------|---|------|------|
| `text-indent` | `<length> \| <percentage>` | 0 | ✅ |
| `text-align` | left \| right \| center \| justify | left | ✅ |
| `text-decoration` | none \| [ underline \| overline \| line-through ] | none | ❌ |
| `text-transform` | none \| capitalize \| uppercase \| lowercase | none | ✅ |
| `letter-spacing` | normal \| `<length>` | normal | ✅ |
| `word-spacing` | normal \| `<length>` | normal | ✅ |
| `white-space` | normal \| pre \| nowrap \| pre-wrap \| pre-line | normal | ✅ |
| `line-height` | normal \| `<number>` \| `<length>` \| `<percentage>` | normal | ✅ |

### 4. 盒模型

| 属性 | 值 | 默认 | 继承 |
|------|---|------|------|
| `width` | `<length> \| <percentage> \| auto` | auto | ❌ |
| `min-width` | `<length> \| <percentage>` | 0 | ❌ |
| `max-width` | `<length> \| <percentage> \| none` | none | ❌ |
| `height` | `<length> \| <percentage> \| auto` | auto | ❌ |
| `min-height` | `<length> \| <percentage>` | 0 | ❌ |
| `max-height` | `<length> \| <percentage> \| none` | none | ❌ |
| `margin` | `<margin-width>`{1,4} | 0 | ❌ |
| `margin-top` | `<margin-width>` | 0 | ❌ |
| `margin-right` | `<margin-width>` | 0 | ❌ |
| `margin-bottom` | `<margin-width>` | 0 | ❌ |
| `margin-left` | `<margin-width>` | 0 | ❌ |
| `padding` | `<padding-width>`{1,4} | 0 | ❌ |
| `padding-top` | `<padding-width>` | 0 | ❌ |
| `padding-right` | `<padding-width>` | 0 | ❌ |
| `padding-bottom` | `<padding-width>` | 0 | ❌ |
| `padding-left` | `<padding-width>` | 0 | ❌ |

**`<margin-width>`**：`<length>` | `<percentage>` | auto

### 5. 边框

| 属性 | 值 | 默认 | 继承 |
|------|---|------|------|
| `border-width` | `<border-width>`{1,4} | medium | ❌ |
| `border-style` | `<border-style>`{1,4} | none | ❌ |
| `border-color` | `<color>`{1,4} | 元素 color | ❌ |
| `border-top/right/bottom/left-*` | 同上 | - | ❌ |
| `border` | 简写 | - | ❌ |

**`<border-style>`**：none | hidden | dotted | dashed | solid | double | groove | ridge | inset | outset

### 6. 布局

| 属性 | 值 | 默认 | 继承 |
|------|---|------|------|
| `display` | block \| inline \| inline-block \| none | inline | ❌ |
| `position` | static \| relative \| absolute \| fixed | static | ❌ |
| `top/right/bottom/left` | `<length>` \| `<percentage>` \| auto | auto | ❌ |
| `float` | left \| right \| none | none | ❌ |
| `clear` | none \| left \| right \| both | none | ❌ |
| `visibility` | visible \| hidden \| collapse | visible | ✅ |
| `overflow` | visible \| hidden \| scroll \| auto | visible | ❌ |
| `z-index` | auto \| `<integer>` | auto | ❌ |

### 7. 列表

| 属性 | 值 | 默认 | 继承 |
|------|---|------|------|
| `list-style-type` | disc \| circle \| square \| decimal \| none | disc | ✅ |
| `list-style-image` | `<uri>` \| none | none | ✅ |
| `list-style-position` | inside \| outside | outside | ✅ |
| `list-style` | 简写 | - | ✅ |

### 8. 表格

| 属性 | 值 | 默认 | 继承 |
|------|---|------|------|
| `border-collapse` | collapse \| separate | separate | ✅ |
| `border-spacing` | `<length>` `<length>`? | 0 | ✅ |
| `caption-side` | top \| bottom | top | ✅ |
| `empty-cells` | show \| hide | show | ✅ |
| `table-layout` | auto \| fixed | auto | ✅ |

### 9. 内容生成

| 属性 | 值 | 默认 | 继承 |
|------|---|------|------|
| `content` | normal \| none \| `<string>` \| `<uri>` \| counter() \| attr() | normal | ❌ |
| `quotes` | `<string>`+ \| none | UA 决定 | ✅ |
| `counter-reset` | `<identifier>` `<integer>`? \| none | none | ❌ |
| `counter-increment` | `<identifier>` `<integer>`? \| none | none | ❌ |

### 10. 视觉效果

| 属性 | 值 | 默认 | 继承 |
|------|---|------|------|
| `opacity` | `<number>` (0.0-1.0) | 1.0 | ❌ |
| `cursor` | auto \| pointer \| wait \| text \| crosshair \| move \| `<uri>` | auto | ✅ |
| `box-shadow` | none \| `<length>`{2,4} `<color>`? | none | ❌ |
| `border-radius` | `<length>`{1,4} | 0 | ❌ |
| `transform` | none \| scale() \| rotate() \| translate() | none | ❌ |
| `transition` | `<property>` `<duration>` `<timing>` `<delay>`? | all 0s ease | ❌ |

### 简写规则

```css
/* margin/padding: 1-4 个值 */
margin: 10px;                    /* 全部 */
margin: 10px 20px;               /* 上下 左右 */
margin: 10px 20px 30px;          /* 上 左右 下 */
margin: 10px 20px 30px 40px;     /* 上 右 下 左 */

/* background 简写（顺序无关） */
background: #fff url(bg.png) no-repeat right top;

/* font 简写（必须按顺序） */
font: [font-style] [font-variant] [font-weight] font-size[/line-height] font-family;
font: italic small-caps bold 16px/1.5 "Noto Sans", sans-serif;

/* border 简写 */
border: 1px solid #ccc;
border-top: 2px dashed red;

/* list-style 简写 */
list-style: square inside url(bullet.png);
```

---

## 值类型与单位

### 颜色

| 格式 | 示例 | 说明 |
|------|------|------|
| 关键字 | `red`, `blue`, `transparent` | 16 个基础色 + transparent |
| 十六进制 | `#ff0000`, `#f00` | 6 位或 3 位 |
| RGB | `rgb(255, 0, 0)` | 0-255 |
| RGB 百分百 | `rgb(100%, 0%, 0%)` | 0%-100% |

### 长度单位

| 单位 | 相对基准 | 示例 |
|------|----------|------|
| `px` | 设备像素 | `16px` |
| `em` | 父元素 font-size | `1.5em` |
| `ex` | 字体 x-height | `12ex` |
| `pt` | 1/72 英寸 | `12pt` |
| `pc` | 12pt = 1pc | `1pc` |
| `cm` | 厘米 | `1cm` |
| `mm` | 毫米 | `10mm` |
| `in` | 英寸 | `1in` |

### 百分比

百分比总是相对于**父元素的相同属性值**：
- `width: 50%` → 父元素宽度的 50%
- `font-size: 120%` → 父元素字体大小的 120%

### URL

```css
background-image: url("assets/bg.png");
background-image: url('assets/bg.png');
background-image: url(assets/bg.png);
```

### 字符串

```css
content: "Hello, World!";
font-family: "Noto Sans", 'Microsoft YaHei', monospace;
```

---

## 盒模型

### 标准盒模型

```
┌─────────────────────────────────────┐
│            margin (外边距)           │
│  ┌───────────────────────────────┐  │
│  │         border (边框)          │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │      padding (内边距)    │  │  │
│  │  │  ┌───────────────────┐  │  │  │
│  │  │  │    content        │  │  │  │
│  │  │  │    (内容区)        │  │  │  │
│  │  │  └───────────────────┘  │  │  │
│  │  └─────────────────────────┘  │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

### 宽度计算

```css
/* 标准模式 */
box-sizing: content-box;  /* 默认 */
width = content-width
total-width = content-width + padding-left + padding-right + border-left-width + border-right-width

/* IE 模式（可选扩展） */
box-sizing: border-box;
width = content-width + padding + border
```

### 外边距合并

相邻块级元素的垂直外边距会合并：
```css
/* 两个 margin: 20px 的元素，间距为 20px，不是 40px */
.box1 { margin-bottom: 20px; }
.box2 { margin-top: 20px; }
/* 实际间距 = max(20px, 20px) = 20px */
```

特殊情况：
- 父元素与第一个/最后一个子元素的外边距合并
- 空元素的外边距合并
- 浮动元素、绝对定位元素的外边距不合并

---

## 布局模式

### 块级布局 (display: block)

块级元素特性：
- 独占一行（前后换行）
- 宽度默认 auto（填满父容器）
- 可设置宽高、内外边距

```css
.block {
    display: block;
    width: 200px;
    margin: 10px auto;  /* 水平居中 */
}
```

### 行内布局 (display: inline)

行内元素特性：
- 与其他行内元素共享一行
- 宽度、高度由内容决定
- 忽略宽高、上下边距

```css
.inline {
    display: inline;
    padding: 0 5px;     /* 左右边距有效 */
    margin: 0 10px;     /* 左右有效 */
}
```

### 行内块级 (display: inline-block)

兼具两者特性：
- 与其他元素共享一行
- 可设置宽高、内外边距

```css
.inline-block {
    display: inline-block;
    width: 100px;
    height: 50px;
    vertical-align: middle;
}
```

### 隐藏元素 (display: none)

完全隐藏，不占据空间：

```css
.hidden {
    display: none;  /* 元素不可见，不占位 */
}
```

与 `visibility: hidden` 的区别：
- `visibility: hidden`：元素不可见，**仍占位**
- `display: none`：元素不可见，**不占位**

---

## 定位系统

### static（静态定位）

默认值，元素按正常文档流排列：

```css
.static {
    position: static;
    /* top/right/bottom/left 无效 */
}
```

### relative（相对定位）

相对于**元素原本位置**偏移，原位置保留：

```css
.relative {
    position: relative;
    top: 10px;      /* 向下偏移 10px */
    left: 20px;     /* 向右偏移 20px */
    /* 原位置仍占据空间 */
}
```

### absolute（绝对定位）

相对于**最近的已定位祖先**（非 static）偏移：

```css
.container {
    position: relative;  /* 锚点 */
}

.absolute {
    position: absolute;
    top: 0;
    right: 0;
    /* 相对于 .container 右上角定位 */
}
```

### fixed（固定定位）

相对于**视口**定位，滚动时不移动：

```css
.fixed-header {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    z-index: 100;
}
```

### z-index 堆叠顺序

```css
.layer1 {
    z-index: 1;  /* 较低层 */
}

.layer2 {
    z-index: 2;  /* 较高层，覆盖 layer1 */
}
```

堆叠上下文规则：
- 根元素生成根堆叠上下文
- `position` 非 `static` 且 `z-index` 非 `auto` 的元素生成堆叠上下文
- `opacity < 1` 的元素生成堆叠上下文

---

## 伪类与伪元素

### 动态伪类

```css
/* 链接状态 */
a:link { color: blue; }
a:visited { color: purple; }
a:hover { color: red; }
a:active { color: green; }

/* 表单状态 */
input:focus {
    outline: 2px solid blue;
}

button:active {
    transform: scale(0.98);
}
```

### 结构伪类

```css
/* 第一个子元素 */
li:first-child {
    font-weight: bold;
}

/* 最后一个子元素 */
li:last-child {
    border-bottom: none;
}

/* 唯一子元素 */
div:only-child {
    margin: 0;
}

/* 空元素 */
div:empty {
    display: none;
}
```

### 伪元素

```css
/* 首行样式 */
p::first-line {
    font-weight: bold;
    color: #333;
}

/* 首字母样式 */
p::first-letter {
    font-size: 200%;
    float: left;
    margin-right: 5px;
}

/* 前置内容 */
.required::before {
    content: "*";
    color: red;
    margin-right: 4px;
}

/* 后置内容 */
.external-link::after {
    content: " ↗";
    font-size: 0.8em;
}
```

---

## @规则

### @charset

指定样式表字符编码：

```css
@charset "UTF-8";
```

**必须**：文件第一行，前面不能有空格或注释。

### @import

导入其他样式表：

```css
@import url("base.ass");
@import "theme.ass";
@import url("responsive.ass") print;  /* 媒体类型（保留但忽略） */
```

**必须**：放在 `@charset` 之后，其他规则之前。

### @font-face

定义自定义字体：

```css
@font-face {
    font-family: "MyFont";
    src: url("fonts/MyFont.otf") format("opentype");
    font-weight: normal;
    font-style: normal;
}

@font-face {
    font-family: "MyFont";
    src: url("fonts/MyFont-Bold.otf") format("opentype");
    font-weight: bold;
}

/* 使用 */
body {
    font-family: "MyFont", sans-serif;
}
```

支持的 `src` 格式：
- `format("truetype")` - .ttf
- `format("opentype")` - .otf
- `format("woff")` - .woff
- `format("woff2")` - .woff2

### @media

媒体查询（语法保留，但媒体类型判断由引擎控制）：

```css
@media screen {
    /* 屏幕显示 */
    body { background: white; }
}

@media print {
    /* 打印样式（游戏通常不涉及） */
    body { background: none; }
}
```

ASS 实现保留解析，但由宿主引擎决定应用哪些媒体。

### @page

页面规则（保留语法，游戏场景可用作"场景"样式）：

```css
@page {
    size: 1920x1080;
    margin: 0;
}

@page :first {
    margin: 0;
}
```

---

## 优先级与级联

### 权重计算

CSS 用 4 个数字表示选择器优先级（千、百、十、个）：

| 选择器类型 | 权重 |
|-----------|------|
| 内联样式（style 属性） | 1,0,0,0 |
| ID 选择器 | 0,1,0,0 |
| 类、属性、伪类选择器 | 0,0,1,0 |
| 元素、伪元素选择器 | 0,0,0,1 |
| 通配符 (*)、组合符 (+, >, ~) | 0,0,0,0 |

**示例**：

```css
* {}                         /* 0,0,0,0 */
li {}                        /* 0,0,0,1 */
ul li {}                     /* 0,0,0,2 */
.nav {}                      /* 0,0,1,0 */
#nav {}                      /* 0,1,0,0 */
ul.nav li {}                 /* 0,0,1,2 (0,0,1 + 0,0,0,2) */
#nav .selected:hover {}      /* 0,1,1,1 */
```

### !important 规则

```css
.important {
    color: red !important;
    /* 覆盖任何普通声明 */
}
```

`!important` 声明的优先级高于所有普通声明。多个 `!important` 之间按源排序。

### 样式来源优先级

从高到低：

1. 用户代理样式表（浏览器的默认样式）
2. 用户样式表（用户自定义）
3. 作者样式表（开发者的 .ass 文件）
   - 内联样式（style 属性）
   - ID 选择器
   - 类、属性、伪类
   - 元素、伪元素
4. 样式表来源的重要性：
   - 带有 `!important` 的作者样式
   - 带有 `!important` 的用户样式
   - 带有 `!important` 的用户代理样式

### 级联顺序

当优先级相同时，后声明的规则覆盖前声明的：

```css
/* 两个规则优先级相同，后面覆盖前面 */
.title { color: blue; }
.title { color: red; }   /* 最终是红色 */
```

### 继承

某些属性值会从父元素继承：

```css
/* 继承属性示例 */
body {
    color: #333;        /* 子元素继承 */
    font-family: sans-serif;  /* 子元素继承 */
}

/* 强制继承 */
.child {
    color: inherit;     /* 主动继承父元素的值 */
}
```

### 初始值

```css
.reset {
    color: initial;     /* 重置为默认值 */
    display: initial;   /* inline（块元素的初始值是 block？等） */
}
```

---

## Python API

### 安装

```bash
pip install axn-ass
```

### 核心 API

```python
from axn_ass import StyleSheet, StyleEngine, StyleContext

# ============ 1. 加载样式表 ============
sheet = StyleSheet()
sheet.load_string("""
    button {
        background: #3498db;
        border-radius: 4px;
        padding: 8px 16px;
    }
    button:hover {
        background: #2980b9;
    }
""")

# 或从文件加载
sheet.load_file("ui/main.ass")

# 或从多个文件合并
sheet.load_files(["ui/base.ass", "ui/theme.ass"])


# ============ 2. 解析与计算 ============
engine = StyleEngine(sheet)

# 创建样式上下文（文档对象模型）
context = StyleContext()

# 添加元素
element = context.add_element(
    tag="button",
    id="submit-btn",
    classes=["primary", "large"],
    parent=None  # None 表示根元素
)

# 添加子元素
text = context.add_element(
    tag="span",
    classes=["label"],
    parent=element
)

# 计算样式
engine.calculate_styles(context)


# ============ 3. 获取样式值 ============
# 获取元素的最终样式
styles = engine.get_styles(element)

# 获取具体属性值（带类型转换）
bg_color = styles.get_color("background-color")        # (255, 0, 0)
width = styles.get_length("width")                     # 100.0 (px)
margin = styles.get_box("margin")                      # (10, 10, 10, 10)
display = styles.get_display()                         # "block"

# 直接访问原始字符串
bg_raw = styles.get_raw("background-color")            # "#ff0000"


# ============ 4. 伪类状态管理 ============
# 设置伪类状态
element.set_pseudo_class("hover", True)
element.set_pseudo_class("active", True)
element.set_pseudo_class("focus", True)

# 清除所有伪类
element.clear_pseudo_classes()

# 重新计算样式（状态变化后）
engine.calculate_styles(context)


# ============ 5. 布局计算 ============
layout = engine.create_layout(context, viewport_width=1920, viewport_height=1080)
layout.calculate()

# 获取元素的布局框
rect = layout.get_bounding_box(element)
print(f"位置: ({rect.x}, {rect.y})")
print(f"尺寸: {rect.width} x {rect.height}")

# 获取外边距框
margin_rect = layout.get_margin_box(element)

# 获取边框框（在布局中使用）
border_rect = layout.get_border_box(element)

# 获取内容框
content_rect = layout.get_content_box(element)


# ============ 6. DOM 操作 ============
# 创建复杂 DOM 树
root = StyleContext()

def create_button(text, id, parent):
    btn = root.add_element(
        tag="button",
        id=id,
        classes=["btn"],
        parent=parent
    )
    
    span = root.add_element(
        tag="span",
        classes=["btn-text"],
        parent=btn
    )
    
    text_node = root.add_text_node(text, parent=span)
    
    return btn

# 移除元素
root.remove_element(element)

# 移动元素
root.move_element(element, new_parent)


# ============ 7. 媒体查询支持 ============
# 设置媒体类型
engine.set_media_type("screen")  # "screen", "print", "all"

# 设置媒体特性（用于扩展查询）
engine.set_media_feature("width", 1920)
engine.set_media_feature("height", 1080)


# ============ 8. 自定义属性扩展 ============
# 添加自定义属性处理器
def custom_property_handler(value):
    # 处理自定义属性
    return parse_my_special_value(value)

engine.register_property_handler("animation-timing", custom_property_handler)


# ============ 9. 调试与性能分析 ============
# 获取样式匹配信息（调试用）
match_info = engine.debug_get_matches(element)
for selector, specificity, rule in match_info:
    print(f"{selector} -> {specificity}")

# 性能统计
stats = engine.get_stats()
print(f"解析耗时: {stats['parse_time_ms']}ms")
print(f"匹配次数: {stats['matches_count']}")
print(f"元素总数: {stats['element_count']}")

# 导出样式表
ass_code = sheet.to_string()
print(ass_code)


# ============ 10. 集成到游戏引擎 ============
class GameUI:
    def __init__(self):
        self.style_engine = StyleEngine()
        self.style_engine.load_file("game.ass")
        self.context = StyleContext()
        self.layout = None
        
    def build_ui(self):
        # 构建 UI 树
        self.root = self.context.add_element(tag="root")
        # ... 添加控件
        
        # 每帧更新
        self.style_engine.calculate_styles(self.context)
        self.layout = self.style_engine.create_layout(
            self.context, 
            viewport_width=1920, 
            viewport_height=1080
        )
        self.layout.calculate()
        
    def handle_hover(self, element, is_hovering):
        # 更新伪类状态并重新计算
        element.set_pseudo_class("hover", is_hovering)
        self.style_engine.calculate_styles(self.context)
        self.layout.calculate()  # 重新布局（如果需要）
        
    def draw(self, renderer):
        # 遍历元素，根据样式和布局渲染
        for element in self.context.traverse():
            rect = self.layout.get_bounding_box(element)
            styles = self.style_engine.get_styles(element)
            
            # 获取样式值
            bg = styles.get_color("background-color")
            border = styles.get_box("border-width")
            
            # 调用渲染器
            renderer.draw_rect(rect, bg, border=border)
```

### 高级用法

```python
# ============ 自定义属性值解析 ============
class CustomValueParser:
    def parse_color(self, value):
        """自定义颜色解析"""
        if value.startswith("--"):
            return self.theme_colors[value[2:]]
        return super().parse_color(value)

engine.set_value_parser(CustomValueParser())


# ============ 样式表热重载 ============
class HotReloadableUI:
    def __init__(self, style_path):
        self.style_path = style_path
        self.last_mtime = 0
        self.reload()
    
    def reload(self):
        mtime = os.path.getmtime(self.style_path)
        if mtime > self.last_mtime:
            self.engine.load_file(self.style_path)
            self.last_mtime = mtime
            self.recompute_all_styles()
            return True
        return False
    
    def update(self):
        if self.reload():
            print("样式已更新")
        # ... 正常更新


# ============ 性能优化：缓存 ============
class CachedStyleEngine(StyleEngine):
    def __init__(self):
        super().__init__()
        self._style_cache = {}
    
    def calculate_styles(self, context):
        # 缓存键：元素路径 + 伪类状态
        for element in context.traverse():
            key = self._make_cache_key(element)
            if key not in self._style_cache:
                self._style_cache[key] = super()._compute_style(element)
        
        # 应用缓存
        for element in context.traverse():
            key = self._make_cache_key(element)
            element.set_computed_styles(self._style_cache[key])
```

---

## 完整示例

### 示例 1：对话气泡

```css
/* dialogue-bubble.ass */
.dialogue-bubble {
    position: relative;
    background: #ffffff;
    border: 2px solid #333333;
    border-radius: 16px;
    padding: 12px 20px;
    max-width: 400px;
    font-family: "Noto Sans", sans-serif;
    font-size: 16px;
    color: #333333;
}

.dialogue-bubble::before {
    content: "";
    position: absolute;
    bottom: -10px;
    left: 20px;
    border-width: 10px 10px 0 0;
    border-style: solid;
    border-color: #ffffff transparent transparent transparent;
}

.dialogue-bubble::after {
    content: "";
    position: absolute;
    bottom: -13px;
    left: 18px;
    border-width: 12px 12px 0 0;
    border-style: solid;
    border-color: #333333 transparent transparent transparent;
}

.dialogue-bubble .speaker {
    font-weight: bold;
    color: #ff8800;
    margin-bottom: 6px;
}

.dialogue-bubble .text {
    line-height: 1.4;
}
```

### 示例 2：主菜单

```css
/* main-menu.ass */
.menu-container {
    position: fixed;
    width: 100%;
    height: 100%;
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
    display: flex;
    justify-content: center;
    align-items: center;
}

.menu-panel {
    background: rgba(0, 0, 0, 0.8);
    border-radius: 16px;
    padding: 40px;
    min-width: 300px;
    text-align: center;
}

.menu-title {
    font-size: 48px;
    font-weight: bold;
    color: #ff8800;
    margin-bottom: 40px;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
}

.menu-button {
    display: block;
    width: 100%;
    padding: 12px 24px;
    margin: 10px 0;
    background: transparent;
    border: 2px solid #ff8800;
    border-radius: 8px;
    color: #ff8800;
    font-size: 18px;
    cursor: pointer;
    transition: all 0.3s ease;
}

.menu-button:hover {
    background: #ff8800;
    color: #1a1a2e;
    transform: scale(1.05);
}

.menu-button:active {
    transform: scale(0.98);
}
```

### 示例 3：物品栏网格

```css
/* inventory.ass */
.inventory-grid {
    display: block;
    padding: 20px;
    background: rgba(0, 0, 0, 0.6);
    border-radius: 12px;
}

.inventory-header {
    font-size: 24px;
    font-weight: bold;
    margin-bottom: 20px;
    color: #ffffff;
}

.item-slot {
    display: inline-block;
    width: 80px;
    height: 80px;
    margin: 8px;
    background: rgba(255, 255, 255, 0.1);
    border: 2px solid rgba(255, 255, 255, 0.2);
    border-radius: 8px;
    text-align: center;
    transition: all 0.2s ease;
}

.item-slot:hover {
    background: rgba(255, 136, 0, 0.3);
    border-color: #ff8800;
    transform: translateY(-2px);
}

.item-slot.selected {
    background: rgba(255, 136, 0, 0.5);
    border-color: #ff8800;
    box-shadow: 0 0 10px rgba(255, 136, 0, 0.5);
}

.item-icon {
    width: 48px;
    height: 48px;
    margin: 8px auto 0 auto;
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
}

.item-count {
    font-size: 12px;
    color: #ff8800;
    margin-top: 4px;
}
```

### 使用示例

```python
# Python 端使用
from axn_ass import StyleEngine, StyleContext

# 加载样式
engine = StyleEngine()
engine.load_file("ui/main-menu.ass")

# 构建 UI
context = StyleContext()
root = context.add_element(tag="menu-container")
panel = context.add_element(tag="menu-panel", parent=root)
title = context.add_element(tag="menu-title", parent=panel)
title.text_node = context.add_text_node("Axn-Plus", parent=title)

buttons = ["开始游戏", "继续游戏", "设置", "退出"]
for text in buttons:
    btn = context.add_element(
        tag="menu-button",
        classes=["menu-button"],
        parent=panel
    )
    btn.text_node = context.add_text_node(text, parent=btn)

# 计算样式和布局
engine.calculate_styles(context)
layout = engine.create_layout(context, viewport_width=1920, viewport_height=1080)
layout.calculate()

# 渲染
for element in context.traverse():
    rect = layout.get_bounding_box(element)
    styles = engine.get_styles(element)
    # ... 调用游戏引擎的渲染 API
```

---

## 附录

### A. 支持的颜色名

```
black, silver, gray, white, maroon, red, purple, fuchsia, green, 
lime, olive, yellow, navy, blue, teal, aqua, transparent
```

### B. 系统字体

```css
/* 通用字体族 */
font-family: serif;      /* 衬线字体 */
font-family: sans-serif; /* 无衬线字体 */
font-family: monospace;  /* 等宽字体 */
font-family: cursive;    /* 手写体 */
font-family: fantasy;    /* 装饰体 */
```

### C. 性能建议

1. **避免过深的选择器嵌套**：选择器深度每增加一层，匹配时间增加约 10%
2. **尽量使用类选择器**：比元素选择器快，比后代选择器快很多
3. **减少通配符 `*` 的使用**：会匹配每个元素
4. **缓存常用样式**：对于频繁更新的元素，使用 `StyleCache`
5. **合理使用 `transform`**：触发硬件加速，提高动画性能

### D. 已知限制

1. 不支持 CSS 变量（`var(--custom)`）
2. 不支持 `calc()` 函数
3. 不支持 `@keyframes` 动画（用游戏的动画系统替代）
4. 不支持 `flexbox` 和 `grid`（用块级 + 行内布局替代）
5. 不支持 `filter` 和 `backdrop-filter`（用游戏特效系统替代）

---

这份文档涵盖了 CSS 2.1 的完整内容，并提供了可直接使用的 Python API。ASS 的设计目标是**在游戏 UI 场景下，提供接近标准 CSS 的体验，同时保持高性能和易集成性**。
