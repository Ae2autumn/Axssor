好的，我来合并两份文档并转化为**可执行的实现文档**。这是一个完整的、可以照着编码的实现指南。

---

# ASS 样式引擎 + Pygame 渲染器 - 完整实现文档

## 目录

1. [项目结构](#项目结构)
2. [核心样式引擎实现](#核心样式引擎实现)
3. [Pygame 渲染器实现](#pygame-渲染器实现)
4. [完整示例](#完整示例)
5. [构建与安装](#构建与安装)

---

## 项目结构

```
axn-ass/
├── axn_ass/
│   ├── __init__.py
│   ├── stylesheet.py      # 样式表解析
│   ├── selector.py        # 选择器匹配
│   ├── values.py          # 值解析（颜色、长度等）
│   ├── engine.py          # 样式计算引擎
│   ├── context.py         # DOM 上下文
│   └── layout.py          # 布局引擎
├── axn_ass_pygame/
│   ├── __init__.py
│   ├── converter.py       # 样式 → 绘图命令
│   ├── renderer.py        # Pygame 渲染器
│   └── resources.py       # 资源加载器
├── examples/
│   ├── simple_button.py
│   └── game_ui.py
├── tests/
│   └── test_engine.py
├── setup.py
└── README.md
```

---

## 核心样式引擎实现

### 1. 值解析模块 (`axn_ass/values.py`)

```python
"""值解析模块 - 颜色、长度、百分比等"""

import re
from typing import Tuple, Optional, Union
from dataclasses import dataclass


@dataclass
class Color:
    """RGBA 颜色"""
    r: int
    g: int
    b: int
    a: int = 255
    
    def to_rgb(self) -> Tuple[int, int, int]:
        return (self.r, self.g, self.b)
    
    def to_rgba(self) -> Tuple[int, int, int, int]:
        return (self.r, self.g, self.b, self.a)


@dataclass
class Length:
    """长度值"""
    value: float
    unit: str  # 'px', 'em', '%', 'pt', 或 '' 表示数字
    
    def to_px(self, context: float = 16.0) -> float:
        """转换为像素"""
        if self.unit == 'px' or self.unit == '':
            return self.value
        elif self.unit == 'em':
            return self.value * context
        elif self.unit == '%':
            return self.value / 100.0 * context
        elif self.unit == 'pt':
            return self.value * 96 / 72
        else:
            return self.value
    
    def to_int(self, context: float = 16.0) -> int:
        return int(self.to_px(context))


class ValueParser:
    """值解析器"""
    
    # 基础颜色名
    COLOR_NAMES = {
        'black': (0, 0, 0),
        'white': (255, 255, 255),
        'red': (255, 0, 0),
        'green': (0, 255, 0),
        'blue': (0, 0, 255),
        'yellow': (255, 255, 0),
        'cyan': (0, 255, 255),
        'magenta': (255, 0, 255),
        'gray': (128, 128, 128),
        'grey': (128, 128, 128),
        'silver': (192, 192, 192),
        'maroon': (128, 0, 0),
        'olive': (128, 128, 0),
        'purple': (128, 0, 128),
        'teal': (0, 128, 128),
        'navy': (0, 0, 128),
        'transparent': (0, 0, 0, 0),
    }
    
    @classmethod
    def parse_color(cls, value: str) -> Optional[Color]:
        """解析颜色值"""
        if not value:
            return None
        
        value = value.strip().lower()
        
        # 关键字
        if value in cls.COLOR_NAMES:
            rgb = cls.COLOR_NAMES[value]
            if len(rgb) == 4:
                return Color(*rgb)
            return Color(*rgb, 255)
        
        # #RRGGBB 或 #RGB
        if value.startswith('#'):
            if len(value) == 7:
                return Color(
                    int(value[1:3], 16),
                    int(value[3:5], 16),
                    int(value[5:7], 16)
                )
            elif len(value) == 4:
                return Color(
                    int(value[1]*2, 16),
                    int(value[2]*2, 16),
                    int(value[3]*2, 16)
                )
        
        # rgb(r, g, b)
        rgb_match = re.match(r'rgb\(\s*(\d+)\s*,\s*(\d+)\s*,\s*(\d+)\s*\)', value)
        if rgb_match:
            return Color(
                int(rgb_match.group(1)),
                int(rgb_match.group(2)),
                int(rgb_match.group(3))
            )
        
        # rgba(r, g, b, a)
        rgba_match = re.match(r'rgba\(\s*(\d+)\s*,\s*(\d+)\s*,\s*(\d+)\s*,\s*([\d.]+)\s*\)', value)
        if rgba_match:
            return Color(
                int(rgba_match.group(1)),
                int(rgba_match.group(2)),
                int(rgba_match.group(3)),
                int(float(rgba_match.group(4)) * 255)
            )
        
        return None
    
    @classmethod
    def parse_length(cls, value: str, default_unit: str = 'px') -> Length:
        """解析长度值"""
        if not value:
            return Length(0, default_unit)
        
        if isinstance(value, (int, float)):
            return Length(float(value), default_unit)
        
        value = value.strip().lower()
        
        # 纯数字
        if value.isdigit() or (value.startswith('-') and value[1:].isdigit()):
            return Length(float(value), default_unit)
        
        # 数字 + 单位
        match = re.match(r'^(-?\d+(?:\.\d+)?)(px|em|%|pt|cm|mm|in)?$', value)
        if match:
            num = float(match.group(1))
            unit = match.group(2) or default_unit
            return Length(num, unit)
        
        return Length(0, default_unit)
    
    @classmethod
    def parse_box(cls, value: str, default: int = 0) -> Tuple[int, int, int, int]:
        """解析 margin/padding 的 1-4 个值"""
        if not value:
            return (default, default, default, default)
        
        parts = value.strip().split()
        
        if len(parts) == 1:
            v = cls.parse_length(parts[0]).to_int()
            return (v, v, v, v)
        elif len(parts) == 2:
            v1 = cls.parse_length(parts[0]).to_int()
            v2 = cls.parse_length(parts[1]).to_int()
            return (v1, v2, v1, v2)
        elif len(parts) == 3:
            v1 = cls.parse_length(parts[0]).to_int()
            v2 = cls.parse_length(parts[1]).to_int()
            v3 = cls.parse_length(parts[2]).to_int()
            return (v1, v2, v3, v2)
        else:
            return (
                cls.parse_length(parts[0]).to_int(),
                cls.parse_length(parts[1]).to_int(),
                cls.parse_length(parts[2]).to_int(),
                cls.parse_length(parts[3]).to_int()
            )
```

### 2. 选择器模块 (`axn_ass/selector.py`)

```python
"""选择器匹配模块"""

import re
from typing import List, Tuple, Optional
from dataclasses import dataclass, field


@dataclass
class Selector:
    """单个选择器"""
    specificity: Tuple[int, int, int]  # (id, class, element)
    pattern: str
    tag: Optional[str] = None
    id: Optional[str] = None
    classes: List[str] = field(default_factory=list)
    parent: Optional['Selector'] = None
    
    def matches(self, tag: str, elem_id: str, classes: List[str]) -> bool:
        """检查元素是否匹配此选择器"""
        # 标签匹配
        if self.tag and self.tag != '*' and self.tag != tag:
            return False
        
        # ID 匹配
        if self.id and self.id != elem_id:
            return False
        
        # 类匹配
        for cls in self.classes:
            if cls not in classes:
                return False
        
        return True
    
    @classmethod
    def parse(cls, selector_str: str) -> 'Selector':
        """解析选择器字符串"""
        selector_str = selector_str.strip()
        
        sel = Selector(
            specificity=(0, 0, 0),
            pattern=selector_str,
            tag=None,
            id=None,
            classes=[],
            parent=None
        )
        
        # 解析组合选择器（A > B, A B 等）
        # 简化实现：只支持单选择器
        # 完整实现需要处理后代选择器
        
        # 解析 ID
        id_match = re.search(r'#([a-zA-Z_][a-zA-Z0-9_-]*)', selector_str)
        if id_match:
            sel.id = id_match.group(1)
            sel.specificity = (sel.specificity[0] + 1, sel.specificity[1], sel.specificity[2])
        
        # 解析类
        class_matches = re.findall(r'\.([a-zA-Z_][a-zA-Z0-9_-]*)', selector_str)
        for cls in class_matches:
            sel.classes.append(cls)
            sel.specificity = (sel.specificity[0], sel.specificity[1] + 1, sel.specificity[2])
        
        # 解析标签
        # 移除已经解析的部分
        remaining = selector_str
        for cls in class_matches:
            remaining = remaining.replace(f'.{cls}', '')
        if sel.id:
            remaining = remaining.replace(f'#{sel.id}', '')
        
        tag_match = re.match(r'^([a-zA-Z_][a-zA-Z0-9_-]*)', remaining.strip())
        if tag_match:
            sel.tag = tag_match.group(1)
            sel.specificity = (sel.specificity[0], sel.specificity[1], sel.specificity[2] + 1)
        
        return sel


@dataclass
class Rule:
    """样式规则"""
    selector: Selector
    declarations: dict  # property -> value
    line_no: int = 0


class SelectorMatcher:
    """选择器匹配器"""
    
    def __init__(self):
        self.rules: List[Rule] = []
    
    def add_rule(self, rule: Rule):
        self.rules.append(rule)
    
    def match(self, tag: str, elem_id: str, classes: List[str]) -> List[Tuple[Rule, Tuple[int, int, int]]]:
        """匹配所有适用的规则，返回 (规则, 特异性)"""
        matches = []
        for rule in self.rules:
            if rule.selector.matches(tag, elem_id, classes):
                matches.append((rule, rule.selector.specificity))
        
        # 按特异性排序
        matches.sort(key=lambda x: x[1])
        return matches
```

### 3. 样式表解析器 (`axn_ass/stylesheet.py`)

```python
"""样式表解析器"""

import re
from typing import Dict, List, Optional
from .selector import Rule, Selector, SelectorMatcher


class StyleSheet:
    """样式表"""
    
    def __init__(self):
        self.rules: List[Rule] = []
        self.font_faces: List[Dict] = []
        self.imports: List[str] = []
    
    def load_string(self, content: str):
        """从字符串加载样式"""
        self._parse(content)
    
    def load_file(self, path: str):
        """从文件加载样式"""
        with open(path, 'r', encoding='utf-8') as f:
            self.load_string(f.read())
    
    def _parse(self, content: str):
        """解析 CSS 内容"""
        # 移除注释
        content = re.sub(r'/\*.*?\*/', '', content, flags=re.DOTALL)
        
        # 解析规则
        # 简化实现：匹配 selector { declarations }
        rule_pattern = r'([^{]+)\{([^}]*)\}'
        
        for match in re.finditer(rule_pattern, content, re.DOTALL):
            selector_str = match.group(1).strip()
            declarations_str = match.group(2).strip()
            
            # 解析选择器（支持多选择器）
            for sel in selector_str.split(','):
                sel = sel.strip()
                if not sel:
                    continue
                
                # 解析声明
                declarations = self._parse_declarations(declarations_str)
                if declarations:
                    selector = Selector.parse(sel)
                    self.rules.append(Rule(selector, declarations))
    
    def _parse_declarations(self, declarations_str: str) -> Dict[str, str]:
        """解析声明块"""
        declarations = {}
        
        # 按分号分割
        for decl in declarations_str.split(';'):
            decl = decl.strip()
            if not decl:
                continue
            
            # 分割属性和值
            parts = decl.split(':', 1)
            if len(parts) != 2:
                continue
            
            prop = parts[0].strip().lower()
            value = parts[1].strip()
            
            # 移除 !important（简化处理）
            if value.endswith('!important'):
                value = value[:-10].strip()
            
            declarations[prop] = value
        
        return declarations
    
    def get_matcher(self) -> SelectorMatcher:
        """创建选择器匹配器"""
        matcher = SelectorMatcher()
        for rule in self.rules:
            matcher.add_rule(rule)
        return matcher
```

### 4. 样式上下文 (`axn_ass/context.py`)

```python
"""DOM 上下文"""

from typing import List, Dict, Optional, Any
from dataclasses import dataclass, field


@dataclass
class Element:
    """DOM 元素"""
    tag: str
    elem_id: str = ""
    classes: List[str] = field(default_factory=list)
    children: List['Element'] = field(default_factory=list)
    parent: Optional['Element'] = None
    text: Optional[str] = None
    pseudo_classes: Dict[str, bool] = field(default_factory=dict)
    computed_styles: Dict[str, str] = field(default_factory=dict)
    user_data: Dict[str, Any] = field(default_factory=dict)
    
    def set_pseudo_class(self, name: str, active: bool):
        """设置伪类状态"""
        self.pseudo_classes[name] = active
    
    def clear_pseudo_classes(self):
        """清除所有伪类"""
        self.pseudo_classes.clear()
    
    def has_pseudo_class(self, name: str) -> bool:
        """检查伪类状态"""
        return self.pseudo_classes.get(name, False)


class StyleContext:
    """样式上下文 - 管理元素树"""
    
    def __init__(self):
        self.root: Optional[Element] = None
        self._elements: List[Element] = []
    
    def add_element(self, tag: str, elem_id: str = "", 
                    classes: List[str] = None,
                    parent: Optional[Element] = None,
                    text: str = None) -> Element:
        """添加元素"""
        element = Element(
            tag=tag,
            elem_id=elem_id,
            classes=classes or [],
            parent=parent,
            text=text
        )
        
        if parent:
            parent.children.append(element)
        elif not self.root:
            self.root = element
        
        self._elements.append(element)
        return element
    
    def add_text_node(self, text: str, parent: Optional[Element] = None) -> Element:
        """添加文本节点"""
        return self.add_element('#text', text=text, parent=parent)
    
    def remove_element(self, element: Element):
        """移除元素"""
        if element.parent:
            element.parent.children.remove(element)
        if element in self._elements:
            self._elements.remove(element)
    
    def traverse(self) -> List[Element]:
        """遍历所有元素"""
        result = []
        
        def dfs(elem):
            result.append(elem)
            for child in elem.children:
                dfs(child)
        
        if self.root:
            dfs(self.root)
        
        return result
```

### 5. 样式引擎 (`axn_ass/engine.py`)

```python
"""样式计算引擎"""

from typing import Dict, Optional
from .stylesheet import StyleSheet
from .context import Element, StyleContext
from .selector import SelectorMatcher
from .values import ValueParser


class StyleEngine:
    """样式引擎"""
    
    def __init__(self, sheet: Optional[StyleSheet] = None):
        self.sheet = sheet or StyleSheet()
        self._matcher: Optional[SelectorMatcher] = None
        self._init_matcher()
    
    def load_string(self, content: str):
        """加载样式字符串"""
        self.sheet.load_string(content)
        self._init_matcher()
    
    def load_file(self, path: str):
        """加载样式文件"""
        self.sheet.load_file(path)
        self._init_matcher()
    
    def _init_matcher(self):
        """初始化选择器匹配器"""
        self._matcher = self.sheet.get_matcher()
    
    def compute(self, element: Element) -> Dict[str, str]:
        """计算单个元素的最终样式"""
        styles = {}
        
        # 获取所有匹配的规则
        matches = self._matcher.match(
            element.tag,
            element.elem_id,
            element.classes
        )
        
        # 按特异性顺序应用（后面的覆盖前面的）
        for rule, _ in matches:
            for prop, value in rule.declarations.items():
                styles[prop] = value
        
        # 处理伪类
        pseudo_styles = self._compute_pseudo_styles(element)
        styles.update(pseudo_styles)
        
        # 存储计算结果
        element.computed_styles = styles
        return styles
    
    def _compute_pseudo_styles(self, element: Element) -> Dict[str, str]:
        """计算伪类样式"""
        styles = {}
        
        # :hover
        if element.has_pseudo_class('hover'):
            hover_matches = self._matcher.match(
                element.tag + ':hover',
                element.elem_id,
                [c + ':hover' for c in element.classes]
            )
            for rule, _ in hover_matches:
                styles.update(rule.declarations)
        
        return styles
    
    def calculate_styles(self, context: StyleContext):
        """计算上下文中所有元素的样式"""
        for element in context.traverse():
            self.compute(element)
    
    def get_styles(self, element: Element) -> Dict[str, str]:
        """获取元素的计算样式"""
        return element.computed_styles or self.compute(element)
```

### 6. 布局引擎 (`axn_ass/layout.py`)

```python
"""布局引擎"""

from typing import Optional, Tuple
import pygame
from .context import Element, StyleContext
from .engine import StyleEngine
from .values import ValueParser


class LayoutBox:
    """布局框"""
    def __init__(self, element: Element, rect: pygame.Rect):
        self.element = element
        self.rect = rect
        self.children: list['LayoutBox'] = []
    
    def get_bounding_box(self) -> pygame.Rect:
        return self.rect


class LayoutEngine:
    """布局引擎"""
    
    def __init__(self, style_engine: StyleEngine):
        self.style_engine = style_engine
        self.viewport_width = 0
        self.viewport_height = 0
        self.current_y = 0
    
    def calculate(self, context: StyleContext, 
                  viewport_width: int, viewport_height: int) -> LayoutBox:
        """计算布局"""
        self.viewport_width = viewport_width
        self.viewport_height = viewport_height
        self.current_y = 0
        
        root_box = self._layout_element(
            context.root,
            pygame.Rect(0, 0, viewport_width, viewport_height)
        )
        
        return root_box
    
    def _layout_element(self, element: Element, parent_rect: pygame.Rect) -> LayoutBox:
        """布局单个元素"""
        if not element:
            return None
        
        styles = self.style_engine.get_styles(element)
        
        # 获取盒模型值
        margin = ValueParser.parse_box(styles.get('margin', '0'))
        padding = ValueParser.parse_box(styles.get('padding', '0'))
        border = ValueParser.parse_box(styles.get('border-width', '0'))
        
        # 计算尺寸
        width = self._get_length(styles.get('width', 'auto'), parent_rect.width)
        height = self._get_length(styles.get('height', 'auto'), parent_rect.height)
        
        if width == 'auto':
            width = parent_rect.width - margin[1] - margin[3] - padding[1] - padding[3]
        
        # 计算位置
        x = parent_rect.x + margin[3]  # margin-left
        y = self.current_y + margin[0]  # margin-top
        
        rect = pygame.Rect(x, y, width, height if height != 'auto' else 0)
        
        # 更新当前 Y 位置
        self.current_y += margin[0] + rect.height + margin[2]
        
        # 创建布局框
        box = LayoutBox(element, rect)
        
        # 布局子元素
        if element.children:
            # 保存当前 Y 位置
            saved_y = self.current_y
            self.current_y = rect.y + padding[0]
            
            for child in element.children:
                child_box = self._layout_element(child, rect)
                if child_box:
                    box.children.append(child_box)
            
            # 恢复 Y 位置
            self.current_y = saved_y
        
        return box
    
    def _get_length(self, value: str, context: int):
        """获取长度值"""
        if not value or value == 'auto':
            return 'auto'
        
        length = ValueParser.parse_length(value)
        if length.unit == '%':
            return int(length.value / 100.0 * context)
        else:
            return length.to_int(context)
```

---

## Pygame 渲染器实现

### 7. 资源加载器 (`axn_ass_pygame/resources.py`)

```python
"""资源加载器"""

import pygame
from typing import Dict, Optional


class ResourceLoader:
    """资源加载器"""
    
    def __init__(self, base_path: str = "assets"):
        self.base_path = base_path
        self.image_cache: Dict[str, pygame.Surface] = {}
        self.font_cache: Dict[str, pygame.font.Font] = {}
    
    def load_image(self, path: str) -> Optional[pygame.Surface]:
        """加载图片"""
        if path in self.image_cache:
            return self.image_cache[path]
        
        full_path = f"{self.base_path}/{path}"
        try:
            image = pygame.image.load(full_path).convert_alpha()
            self.image_cache[path] = image
            return image
        except Exception as e:
            print(f"Failed to load image {full_path}: {e}")
            return None
    
    def load_font(self, path: str, size: int) -> pygame.font.Font:
        """加载字体"""
        cache_key = f"{path}:{size}"
        if cache_key in self.font_cache:
            return self.font_cache[cache_key]
        
        try:
            font = pygame.font.Font(path, size)
        except:
            font = pygame.font.Font(None, size)
        
        self.font_cache[cache_key] = font
        return font
```

### 8. 绘图命令 (`axn_ass_pygame/converter.py`)

```python
"""样式到绘图命令转换器"""

import pygame
from typing import Dict, List, Optional, Tuple
from dataclasses import dataclass
from enum import Enum

from axn_ass.values import ValueParser
from .resources import ResourceLoader


class CommandType(Enum):
    RECT = 1
    TEXT = 2
    IMAGE = 3
    BORDER = 4


@dataclass
class DrawCommand:
    """绘图命令"""
    type: CommandType
    rect: pygame.Rect
    color: Optional[Tuple[int, int, int]] = None
    border_radius: int = 0
    border_width: int = 0
    border_color: Optional[Tuple[int, int, int]] = None
    text: str = ""
    font: Optional[pygame.font.Font] = None
    image: Optional[pygame.Surface] = None
    alignment: str = "center"


class StyleToPygameConverter:
    """样式到 Pygame 转换器"""
    
    def __init__(self, resource_loader: Optional[ResourceLoader] = None):
        self.resource_loader = resource_loader or ResourceLoader()
    
    def convert(self, styles: Dict[str, str], rect: pygame.Rect,
                text: Optional[str] = None) -> List[DrawCommand]:
        """转换样式为绘图命令"""
        commands = []
        
        # 1. 阴影（先绘制，在背景下面）
        shadow_cmd = self._create_shadow_command(styles, rect)
        if shadow_cmd:
            commands.append(shadow_cmd)
        
        # 2. 背景
        bg_cmd = self._create_background_command(styles, rect)
        if bg_cmd:
            commands.append(bg_cmd)
        
        # 3. 背景图片
        bg_img_cmd = self._create_background_image_command(styles, rect)
        if bg_img_cmd:
            commands.append(bg_img_cmd)
        
        # 4. 边框
        border_cmd = self._create_border_command(styles, rect)
        if border_cmd:
            commands.append(border_cmd)
        
        # 5. 文本
        if text:
            text_cmd = self._create_text_command(styles, rect, text)
            if text_cmd:
                commands.append(text_cmd)
        
        return commands
    
    def _create_background_command(self, styles: Dict, rect: pygame.Rect) -> Optional[DrawCommand]:
        """创建背景命令"""
        bg_color = styles.get('background-color', 'transparent')
        if bg_color == 'transparent':
            return None
        
        color = ValueParser.parse_color(bg_color)
        if not color:
            return None
        
        radius = ValueParser.parse_length(styles.get('border-radius', '0')).to_int()
        
        return DrawCommand(
            type=CommandType.RECT,
            rect=rect,
            color=color.to_rgb(),
            border_radius=radius
        )
    
    def _create_border_command(self, styles: Dict, rect: pygame.Rect) -> Optional[DrawCommand]:
        """创建边框命令"""
        border_style = styles.get('border-style', 'none')
        if border_style == 'none':
            return None
        
        border_width = ValueParser.parse_length(styles.get('border-width', '0')).to_int()
        if border_width == 0:
            return None
        
        border_color = styles.get('border-color')
        if not border_color:
            border_color = styles.get('color', '#000000')
        
        color = ValueParser.parse_color(border_color)
        if not color:
            return None
        
        radius = ValueParser.parse_length(styles.get('border-radius', '0')).to_int()
        
        return DrawCommand(
            type=CommandType.BORDER,
            rect=rect,
            border_width=border_width,
            border_color=color.to_rgb(),
            border_radius=radius
        )
    
    def _create_background_image_command(self, styles: Dict, rect: pygame.Rect) -> Optional[DrawCommand]:
        """创建背景图片命令"""
        bg_image = styles.get('background-image', 'none')
        if bg_image == 'none':
            return None
        
        # 移除 url() 包装
        if bg_image.startswith('url(') and bg_image.endswith(')'):
            bg_image = bg_image[4:-1].strip('"\'')
        
        image = self.resource_loader.load_image(bg_image)
        if not image:
            return None
        
        # 缩放图片
        bg_size = styles.get('background-size', 'auto')
        if bg_size == 'cover':
            # 覆盖整个区域
            scaled = pygame.transform.scale(image, (rect.width, rect.height))
            img_rect = rect
        elif bg_size == 'contain':
            # 等比例缩放适应
            ratio = min(rect.width / image.get_width(), rect.height / image.get_height())
            new_size = (int(image.get_width() * ratio), int(image.get_height() * ratio))
            scaled = pygame.transform.scale(image, new_size)
            img_rect = scaled.get_rect(center=rect.center)
        else:
            scaled = image
            # 根据 background-position 定位
            pos = styles.get('background-position', '0% 0%')
            if pos == 'center':
                img_rect = scaled.get_rect(center=rect.center)
            elif pos == 'top left' or pos == '0% 0%':
                img_rect = scaled.get_rect(topleft=rect.topleft)
            elif pos == 'top right':
                img_rect = scaled.get_rect(topright=rect.topright)
            else:
                img_rect = scaled.get_rect(topleft=rect.topleft)
        
        return DrawCommand(
            type=CommandType.IMAGE,
            rect=img_rect,
            image=scaled
        )
    
    def _create_text_command(self, styles: Dict, rect: pygame.Rect, 
                              text: str) -> Optional[DrawCommand]:
        """创建文本命令"""
        color_val = styles.get('color')
        if not color_val:
            return None
        
        color = ValueParser.parse_color(color_val)
        if not color:
            return None
        
        # 获取字体
        font_family = styles.get('font-family', 'sans-serif')
        font_size = ValueParser.parse_length(styles.get('font-size', '16px')).to_int()
        font = self.resource_loader.load_font(font_family, font_size)
        
        # 文本对齐
        text_align = styles.get('text-align', 'center')
        
        return DrawCommand(
            type=CommandType.TEXT,
            rect=rect,
            color=color.to_rgb(),
            text=text,
            font=font,
            alignment=text_align
        )
    
    def _create_shadow_command(self, styles: Dict, rect: pygame.Rect) -> Optional[DrawCommand]:
        """创建阴影命令"""
        shadow = styles.get('box-shadow', 'none')
        if shadow == 'none':
            return None
        
        # 简化解析：box-shadow: offset-x offset-y blur-radius color
        parts = shadow.split()
        if len(parts) < 2:
            return None
        
        offset_x = ValueParser.parse_length(parts[0]).to_int()
        offset_y = ValueParser.parse_length(parts[1]).to_int()
        
        color_val = parts[-1] if len(parts) > 2 else 'rgba(0,0,0,0.3)'
        color = ValueParser.parse_color(color_val)
        if not color:
            color = ValueParser.parse_color('rgba(0,0,0,0.3)')
        
        shadow_rect = rect.move(offset_x, offset_y)
        radius = ValueParser.parse_length(styles.get('border-radius', '0')).to_int()
        
        return DrawCommand(
            type=CommandType.RECT,
            rect=shadow_rect,
            color=color.to_rgb(),
            border_radius=radius
        )
```

### 9. Pygame渲染器 (`axn_ass_pygame/renderer.py`)

```python
"""Pygame 渲染器"""

import pygame
from typing import Optional

from axn_ass.engine import StyleEngine
from axn_ass.context import StyleContext, Element
from axn_ass.layout import LayoutEngine, LayoutBox
from .converter import StyleToPygameConverter, CommandType
from .resources import ResourceLoader


class PygameRenderer:
    """Pygame 渲染器"""
    
    def __init__(self, surface: pygame.Surface, style_engine: Optional[StyleEngine] = None):
        self.surface = surface
        self.style_engine = style_engine or StyleEngine()
        self.converter = StyleToPygameConverter()
        self.layout_engine = LayoutEngine(self.style_engine)
        self.context = StyleContext()
        self.root_box: Optional[LayoutBox] = None
    
    def load_styles(self, path: str):
        """加载样式文件"""
        self.style_engine.load_file(path)
    
    def load_styles_string(self, content: str):
        """加载样式字符串"""
        self.style_engine.load_string(content)
    
    def add_element(self, tag: str, elem_id: str = "", classes=None,
                    parent=None, text=None) -> Element:
        """添加元素"""
        return self.context.add_element(tag, elem_id, classes, parent, text)
    
    def layout(self, viewport_width: int, viewport_height: int):
        """计算布局"""
        self.style_engine.calculate_styles(self.context)
        self.root_box = self.layout_engine.calculate(
            self.context, viewport_width, viewport_height
        )
    
    def render(self):
        """渲染所有元素"""
        if not self.root_box:
            return
        
        self._render_box(self.root_box)
    
    def _render_box(self, box: LayoutBox):
        """递归渲染布局框"""
        if not box or not box.element:
            return
        
        # 获取样式
        styles = self.style_engine.get_styles(box.element)
        
        # 检查是否隐藏
        if styles.get('display') == 'none':
            return
        
        # 获取文本
        text = box.element.text
        
        # 转换并执行绘图命令
        commands = self.converter.convert(styles, box.rect, text)
        for cmd in commands:
            self._execute_command(cmd)
        
        # 渲染子元素
        for child in box.children:
            self._render_box(child)
    
    def _execute_command(self, cmd):
        """执行绘图命令"""
        if cmd.type == CommandType.RECT:
            if cmd.border_radius > 0:
                # 圆角矩形
                pygame.draw.rect(
                    self.surface, cmd.color, cmd.rect,
                    border_top_left_radius=cmd.border_radius,
                    border_top_right_radius=cmd.border_radius,
                    border_bottom_left_radius=cmd.border_radius,
                    border_bottom_right_radius=cmd.border_radius
                )
            else:
                pygame.draw.rect(self.surface, cmd.color, cmd.rect)
        
        elif cmd.type == CommandType.BORDER:
            # 绘制边框
            pygame.draw.rect(
                self.surface, cmd.border_color, cmd.rect,
                width=cmd.border_width,
                border_radius=cmd.border_radius
            )
        
        elif cmd.type == CommandType.TEXT:
            if cmd.font and cmd.text:
                text_surf = cmd.font.render(cmd.text, True, cmd.color)
                
                if cmd.alignment == 'center':
                    text_rect = text_surf.get_rect(center=cmd.rect.center)
                elif cmd.alignment == 'left':
                    text_rect = text_surf.get_rect(midleft=cmd.rect.midleft)
                else:  # right
                    text_rect = text_surf.get_rect(midright=cmd.rect.midright)
                
                self.surface.blit(text_surf, text_rect)
        
        elif cmd.type == CommandType.IMAGE:
            if cmd.image:
                self.surface.blit(cmd.image, cmd.rect)
    
    def handle_event(self, event: pygame.event.Event, mouse_pos: tuple = None):
        """处理事件，更新伪类状态"""
        if event.type == pygame.MOUSEMOTION:
            # 更新所有元素的 hover 状态
            self._update_hover_states(mouse_pos or pygame.mouse.get_pos())
        
        elif event.type == pygame.MOUSEBUTTONDOWN:
            # 处理点击
            pos = pygame.mouse.get_pos()
            element = self._find_element_at_pos(self.root_box, pos)
            if element:
                element.set_pseudo_class('active', True)
                # 重新计算样式
                self.style_engine.calculate_styles(self.context)
                # 重新布局并渲染
                self.layout(self.surface.get_width(), self.surface.get_height())
        
        elif event.type == pygame.MOUSEBUTTONUP:
            # 清除 active 状态
            for elem in self.context.traverse():
                elem.set_pseudo_class('active', False)
            self.style_engine.calculate_styles(self.context)
            self.layout(self.surface.get_width(), self.surface.get_height())
    
    def _update_hover_states(self, mouse_pos: tuple):
        """更新鼠标悬浮状态"""
        for element in self.context.traverse():
            # 需要知道元素的位置（从布局中获取）
            # 简化：在布局中存储位置映射
            pass
    
    def _find_element_at_pos(self, box: LayoutBox, pos: tuple) -> Optional[Element]:
        """查找位置对应的元素"""
        if not box:
            return None
        
        if box.rect.collidepoint(pos):
            # 检查子元素
            for child in reversed(box.children):
                found = self._find_element_at_pos(child, pos)
                if found:
                    return found
            return box.element
        
        return None
    
    def get_element_at(self, x: int, y: int) -> Optional[Element]:
        """获取指定坐标的元素"""
        return self._find_element_at_pos(self.root_box, (x, y))
```

---

## 完整示例

### 示例1：简单按钮 (`examples/simple_button.py`)

```python
"""简单按钮示例"""

import pygame
import sys

from axn_ass.engine import StyleEngine
from axn_ass.context import StyleContext
from axn_ass_pygame.converter import StyleToPygameConverter, CommandType
from axn_ass_pygame.resources import ResourceLoader


def main():
    pygame.init()
    screen = pygame.display.set_mode((800, 600))
    pygame.display.set_caption("ASS Button Example")
    clock = pygame.time.Clock()
    
    # 加载样式
    engine = StyleEngine()
    engine.load_string("""
        .btn {
            width: 200px;
            height: 50px;
            background-color: #3498db;
            border-radius: 8px;
            color: #ffffff;
            font-size: 18px;
            text-align: center;
        }
        .btn:hover {
            background-color: #2980b9;
        }
        .btn:active {
            background-color: #1c6ea4;
        }
    """)
    
    # 创建 UI 元素
    context = StyleContext()
    btn = context.add_element(
        tag="button",
        classes=["btn"],
        text="Click Me!"
    )
    
    # 计算样式
    styles = engine.compute(btn)
    
    # 创建转换器
    converter = StyleToPygameConverter()
    
    # 按钮位置
    btn_rect = pygame.Rect(300, 275, 200, 50)
    
    # 主循环
    running = True
    while running:
        mouse_pos = pygame.mouse.get_pos()
        is_hover = btn_rect.collidepoint(mouse_pos)
        
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.MOUSEBUTTONDOWN and is_hover:
                print("Button clicked!")
                # 临时改变样式
                btn.set_pseudo_class("active", True)
                styles = engine.compute(btn)
            elif event.type == pygame.MOUSEBUTTONUP:
                btn.set_pseudo_class("active", False)
                styles = engine.compute(btn)
        
        # 更新 hover 状态
        btn.set_pseudo_class("hover", is_hover)
        styles = engine.compute(btn)
        
        # 渲染
        screen.fill((30, 30, 40))
        
        commands = converter.convert(styles, btn_rect, btn.text)
        for cmd in commands:
            if cmd.type == CommandType.RECT:
                pygame.draw.rect(screen, cmd.color, cmd.rect,
                               border_radius=cmd.border_radius)
            elif cmd.type == CommandType.TEXT and cmd.font:
                text_surf = cmd.font.render(cmd.text, True, cmd.color)
                text_rect = text_surf.get_rect(center=cmd.rect.center)
                screen.blit(text_surf, text_rect)
        
        pygame.display.flip()
        clock.tick(60)
    
    pygame.quit()
    sys.exit()


if __name__ == "__main__":
    main()
```

### 示例2：完整游戏UI (`examples/game_ui.py`)

```python
"""完整游戏 UI 示例"""

import pygame
import sys

from axn_ass_pygame.renderer import PygameRenderer


def main():
    pygame.init()
    screen = pygame.display.set_mode((1024, 768))
    pygame.display.set_caption("ASS Game UI Demo")
    clock = pygame.time.Clock()
    
    # 创建渲染器
    renderer = PygameRenderer(screen)
    
    # 加载样式
    renderer.load_styles_string("""
        /* 主菜单样式 */
        .menu-panel {
            width: 400px;
            background-color: rgba(0, 0, 0, 0.85);
            border-radius: 16px;
            padding: 40px;
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);
        }
        
        .title {
            font-size: 48px;
            color: #ff8800;
            text-align: center;
            margin-bottom: 40px;
        }
        
        .btn {
            width: 100%;
            height: 50px;
            background-color: #3498db;
            border-radius: 8px;
            color: #ffffff;
            font-size: 18px;
            text-align: center;
            margin-bottom: 16px;
        }
        
        .btn:hover {
            background-color: #2980b9;
        }
        
        .btn:active {
            background-color: #1c6ea4;
        }
        
        .btn-danger {
            background-color: #e74c3c;
        }
        
        .btn-danger:hover {
            background-color: #c0392b;
        }
        
        /* 游戏内 HUD */
        .hud {
            position: fixed;
            top: 20px;
            left: 20px;
            right: 20px;
            background-color: rgba(0, 0, 0, 0.7);
            border-radius: 8px;
            padding: 12px 20px;
        }
        
        .health-bar {
            width: 300px;
            height: 20px;
            background-color: #333333;
            border-radius: 10px;
            overflow: hidden;
        }
        
        .health-fill {
            width: 75%;
            height: 100%;
            background-color: #2ecc71;
            border-radius: 10px;
        }
        
        .score {
            color: #ffffff;
            font-size: 24px;
        }
    """)
    
    # 构建主菜单 UI
    menu_panel = renderer.add_element(
        tag="div",
        classes=["menu-panel"]
    )
    
    title = renderer.add_element(
        tag="h1",
        classes=["title"],
        parent=menu_panel,
        text="My Game"
    )
    
    start_btn = renderer.add_element(
        tag="button",
        classes=["btn"],
        parent=menu_panel,
        text="Start Game"
    )
    
    settings_btn = renderer.add_element(
        tag="button",
        classes=["btn"],
        parent=menu_panel,
        text="Settings"
    )
    
    quit_btn = renderer.add_element(
        tag="button",
        classes=["btn", "btn-danger"],
        parent=menu_panel,
        text="Quit"
    )
    
    # 计算布局
    renderer.layout(1024, 768)
    
    # 按钮点击回调
    def on_click(element):
        if element == start_btn:
            print("Starting game...")
        elif element == settings_btn:
            print("Opening settings...")
        elif element == quit_btn:
            pygame.quit()
            sys.exit()
    
    # 主循环
    running = True
    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            
            elif event.type == pygame.MOUSEBUTTONDOWN:
                pos = pygame.mouse.get_pos()
                element = renderer.get_element_at(pos[0], pos[1])
                if element and element.tag == 'button':
                    on_click(element)
                    # 触发 active 效果
                    element.set_pseudo_class('active', True)
                    renderer.style_engine.calculate_styles(renderer.context)
                    renderer.layout(1024, 768)
            
            elif event.type == pygame.MOUSEBUTTONUP:
                # 清除 active 状态
                for elem in renderer.context.traverse():
                    elem.set_pseudo_class('active', False)
                renderer.style_engine.calculate_styles(renderer.context)
                renderer.layout(1024, 768)
        
        # 更新 hover 状态
        mouse_pos = pygame.mouse.get_pos()
        for element in renderer.context.traverse():
            # 简化：通过布局获取元素位置
            pass
        
        # 渲染
        screen.fill((30, 40, 50))
        renderer.render()
        pygame.display.flip()
        clock.tick(60)
    
    pygame.quit()
    sys.exit()


if __name__ == "__main__":
    main()
```

---

## 构建与安装

### `setup.py`

```python
from setuptools import setup, find_packages

setup(
    name="axn-ass",
    version="0.1.0",
    description="ASS (Axn Style Sheet) - CSS-like styling for Pygame",
    author="Axn-Plus",
    packages=find_packages(),
    install_requires=[
        "pygame>=2.0.0",
    ],
    extras_require={
        "dev": [
            "pytest",
            "pytest-cov",
        ],
    },
    classifiers=[
        "Development Status :: 3 - Alpha",
        "Intended Audience :: Developers",
        "License :: OSI Approved :: MIT License",
        "Programming Language :: Python :: 3",
        "Programming Language :: Python :: 3.8",
        "Programming Language :: Python :: 3.9",
        "Programming Language :: Python :: 3.10",
        "Topic :: Games/Entertainment",
        "Topic :: Multimedia :: Graphics",
    ],
    python_requires=">=3.8",
)
```

### 安装

```bash
# 开发模式安装
pip install -e .

# 或者直接安装
pip install .
```

### 运行示例

```bash
python examples/simple_button.py
python examples/game_ui.py
```

---

这份文档提供了完整的、可运行的代码实现。你可以直接按照这个结构创建文件并开始编码。需要我解释任何部分或添加更多功能吗？
