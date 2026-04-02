# Figma 转代码

根据用户提供的 **Figma 链接** 或 **设计稿截图**，严格按照以下规范生成可直接运行的 React 组件代码。

---

## 前提：安装 Figma MCP

> 仅使用 Figma 链接时需要；纯截图转代码可跳过。

### 1. 安装 Claude Code（CLI）

```bash
npm install -g @anthropic-ai/claude-code
```

### 2. 配置 Figma MCP Server

在 Claude Code 中执行：

```bash
claude mcp add figma --transport sse https://mcp.figma.com/sse
```

或手动编辑 `~/.claude/settings.json`（用户全局）/ `.claude/settings.json`（项目级）：

```json
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["-y", "figma-developer-mcp", "--stdio"],
      "env": {
        "FIGMA_API_TOKEN": "<your-figma-token>"
      }
    }
  }
}
```

### 3. 获取 Figma API Token

1. 打开 Figma → 右上角头像 → **Settings**
2. 左侧 **Security** → **Personal access tokens** → **Generate new token**
3. 勾选 `File content: Read-only`，生成后复制 token
4. 将 token 填入上方配置的 `FIGMA_API_TOKEN`

### 4. 验证安装

重启 Claude Code 后，在对话中粘贴任意 Figma 设计稿链接，Claude 能正常读取设计内容即表示配置成功。

---

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
import React from 'react'; // 1. React（必须第一行）
import { Button, Select, Table } from 'antd'; // 2. antd 组件
import { ReloadOutlined } from '@ant-design/icons'; // 3. 图标只用 @ant-design/icons
import someImg from '@/images/xxx.png'; // 4. 图片资源
import CardHeader from '../CardHeader'; // 5. 兄弟组件
import './index.less'; // 6. 样式（最后）
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
.cm-block {
}
.cm-block__element {
}
.cm-block--modifier {
}
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

### 交互规范

- **按钮**：必须实现 `hover`、`active`、`disabled` 三种状态样式，在 Less 中用 `&:hover`、`&:active`、`&:disabled` / `&.disabled` 分别定义
- **表单**：使用 Ant Design `Form` 的 `rules` 属性实现校验，至少覆盖必填、格式、长度规则，示例：

  ```js
  rules={[
    { required: true, message: '请输入xxx' },
    { max: 50, message: '不超过50个字符' },
  ]}
  ```

- **列表项**：添加过渡动画，在 Less 中统一用 `transition` 定义，示例：

  ```less
  .cm-list__item {
    transition: background 0.2s ease, box-shadow 0.2s ease;
    &:hover {
      background: @sidebar-bg;
    }
  }
  ```

  
### 表单布局规范（重要）

**必须使用 `Form` + `Row` + `Col` 排列表单元素，禁止手写 flex/grid 布局代替。**

#### 1. 声明 form 实例

```js
const [form] = Form.useForm();
```

#### 2. 多列查询表单（筛选条）

```js
// 定义在组件外，保持稳定引用
const itemLayout = {
  labelCol: { span: 4 },
  wrapperCol: { span: 14 },
};
```

```jsx
// 每行 4 列等宽：Col span={6}（总 24 份）
// gutter={[16, 8]} 控制水平/垂直间距
<Form form={form} {...itemLayout} colon={false}>
  <Row gutter={[16, 8]} style={{ width: '100%' }}>
    <Col span={6}>
      <Form.Item label="工单编号" name="orderCode">
        <Input placeholder="请输入" />
      </Form.Item>
    </Col>
    <Col span={6}>
      <Form.Item label="工单环节" name="orderStep">
        <Select placeholder="请选择" allowClear>
          <Option value="1">环节1</Option>
        </Select>
      </Form.Item>
    </Col>
  </Row>

  {/* 最后一行：查询/重置按钮推到右侧 */}
  <Row gutter={[16, 8]} style={{ width: '100%' }}>
    <Col span={12}>...</Col>
    <Col span={12} style={{ display: 'flex', justifyContent: 'flex-end' }}>
      <Form.Item>
        <Button type="primary" icon={<SearchOutlined />} onClick={handleQuery}>
          查询
        </Button>
      </Form.Item>
      <Form.Item style={{ marginLeft: 8, marginRight: 0 }}>
        <Button icon={<ReloadOutlined />} onClick={handleReset}>
          重置
        </Button>
      </Form.Item>
    </Col>
  </Row>
</Form>
```

#### 4. 取值 & 重置

```js
// 查询：校验后取值
const handleQuery = () => {
  form.validateFields().then(values => {
    /* 发起请求 */
  });
};

// 重置：一键清空所有字段
const handleReset = () => {
  form.resetFields();
};

// 直接取全部字段（不校验）
const values = form.getFieldsValue(true);
```

#### 5. Less 中让控件撑满 Col

```less
.cm-filter-form {
  .ant-form-item {
    width: 100%;
    margin-bottom: 0;
  }
  .ant-form-item-control {
    flex: 1;
    min-width: 0;
  }
  .ant-input,
  .ant-select,
  .ant-picker {
    width: 100%;
  }
}
```

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
