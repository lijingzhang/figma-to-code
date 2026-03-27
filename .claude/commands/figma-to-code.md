# Figma 转代码

根据用户提供的 **Figma 链接** 或 **设计稿截图**，严格按照以下规范生成可直接运行的 React 组件代码。

## 第一步：读取设计稿

- **Figma 链接**：调用 `mcp__figma__get_design_context` 工具读取设计稿（rate limit 时改用 REST API）
- **截图**：直接识别图片中的 UI 结构、颜色、间距、文字，无需 MCP

## 第二步：分析 & 拆分组件

按视觉区块将页面拆分为子组件：

- `index.js` 只负责布局拼装，不写业务逻辑
- 每个视觉区块（Header / Sidebar / TabBar / Section 等）独立成组件
- 公共小组件单独抽取，其他组件通过 `../ComponentName` 引用（不是 `./`）
- 静态数据（columns、mock data）写在组件文件内的常量区，不单独建文件

## 第三步：输出文件

### 目录结构

```
src/pages/<PageName>/
  index.js              # 页面入口，只做布局拼装
  index.less
  variables.less        # 共享 Less 变量
  components/
    <ComponentName>/
      index.js
      index.less
```

### JS 文件规范

**import 顺序（每个文件必须严格遵守）：**

```js
import React from 'react';                          // 1. React（必须第一行）
import { Button, Select, Table } from 'antd';        // 2. antd 组件
import { ReloadOutlined } from '@ant-design/icons';  // 3. 图标只用 @ant-design/icons
import someImg from '@/images/xxx.png';              // 4. 图片资源
import CardHeader from '../CardHeader';              // 5. 兄弟组件
import './index.less';                               // 6. 样式（最后）
```

**ESLint 要求（不得违反）：**

- 禁止 `<label>`，改用 `<span>` 避免 `jsx-a11y/label-has-associated-control`
- 不写 `console.log`
- 单参数箭头函数不加括号：`key => ...`
- 不声明未使用的变量
- **所有文件后缀用 `.js`，不用 `.jsx`**

### Less 文件规范

**文件头：**

```less
/* 默认 */
@import '../../variables.less';

```

**尺寸写法：**

```less
/* 默认直接写 px */
padding: 16px;

```

**BEM 命名：**

```less
.cm-block { }
.cm-block__element { }
.cm-block--modifier { }
```

**variables.less 常用变量参考：**

```less
@page-bg: #f8fafc;
@white: #ffffff;
@primary: #2d81f8;
@text-primary: #283857;
@text-secondary: #6b7a97;
@border-color: #e8edf3;
@sidebar-bg: #eef2f6;
```

### 图片资源

- 非 `@ant-design/icons` 的图标 / 图片下载到 `src/images/`，用 `@/images/xxx.png` 引入
- 矢量图标优先用 `@ant-design/icons`，不从其他图标库引入

## 第四步：完整输出

**必须完整输出所有文件（JS + Less），不得省略任何文件或代码内容。**

每个文件用代码块包裹，标注文件路径，例如：

```
// src/pages/MyPage/index.js
```

---

## Figma REST API（rate limit 备用）

```bash
# 获取节点树（Windows 输出到临时文件）
curl -H "X-Figma-Token: <token>" \
  "https://api.figma.com/v1/files/<fileKey>/nodes?ids=<nodeId>" \
  -o C:/Users/<username>/AppData/Local/Temp/figma_node.json

# 获取图片下载链接
curl -H "X-Figma-Token: <token>" \
  "https://api.figma.com/v1/images/<fileKey>?ids=<imageRef>&format=png" \
  -o C:/Users/<username>/AppData/Local/Temp/figma_img.json
```
