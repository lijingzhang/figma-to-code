# figma-to-code — Claude Code Skill

将 Figma 设计稿一键转为可直接运行的 React 组件代码。

> **⚠️ 重要限制**
>
> 官方 Figma MCP（`mcp.figma.com`）**每月仅有 6 次免费调用额度**，超出后需付费订阅。
>
> 如果额度不够用，可以改用开源替代方案 **f2c-mcp**，调用次数不受限制，但设计稿还原精度相对较差（颜色变量、间距、层级结构的识别会有偏差），建议配合截图转代码方式兜底。
>
> | 方案 | 调用限制 | 还原效果 | 费用 |
> |---|---|---|---|
> | 官方 Figma MCP | 6 次/月（免费） | ⭐⭐⭐⭐⭐ 最佳 | 超出后付费 |
> | 开源 f2c-mcp | 无限制 | ⭐⭐⭐ 一般 | 免费 |
> | 截图直接识别 | 无限制 | ⭐⭐⭐ 一般 | 免费 |

---

## 前提条件

### 1. 安装 Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

安装完成后验证：

```bash
claude --version
```

---

### 2. 配置 Figma MCP Server

根据实际情况选择以下任一方案：

---

#### 方案 A：官方 Figma MCP（还原效果最佳，每月 6 次）

**命令行添加：**

```bash
claude mcp add --transport sse figma https://mcp.figma.com/sse
```

**或手动编辑配置文件**（`~/.claude/settings.json` 全局 / `.claude/settings.json` 项目级）：

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

---

#### 方案 B：开源 f2c-mcp（无调用次数限制，还原效果一般）

适合日常高频使用，或官方额度已用完的情况。

```bash
npx f2c-mcp init
```

手动配置：

```json
{
  "mcpServers": {
    "f2c-mcp": {
      "command": "npx",
      "args": ["-y", "f2c-mcp", "--stdio"],
      "env": {
        "FIGMA_API_TOKEN": "<your-figma-token>"
      }
    }
  }
}
```

> 还原精度较官方方案差，颜色变量、间距、图层层级识别可能有偏差，建议结合截图人工核对。

---

### 3. 获取 Figma API Token

1. 打开 [Figma](https://www.figma.com) 并登录
2. 点击右上角头像 → **Settings**
3. 左侧菜单选 **Security**
4. 找到 **Personal access tokens** → 点击 **Generate new token**
5. Token 名称随意填写，权限勾选 `File content: Read-only`
6. 生成后**立即复制**（只显示一次）
7. 将 token 替换上方配置中的 `<your-figma-token>`

---

### 4. 安装本 Skill

将 `figma-to-code.md` 文件放到以下任一目录：

| 作用范围 | 路径 |
|---|---|
| 当前项目 | `<项目根目录>/.claude/commands/figma-to-code.md` |
| 个人全局（所有项目可用） | `~/.claude/commands/figma-to-code.md`（Windows: `C:\Users\<用户名>\.claude\commands\`） |

---

### 5. 验证配置

重启 Claude Code，在对话中输入：

```
/figma-to-code
```

如果命令可以被识别，说明 skill 安装成功。再粘贴一个 Figma 链接，Claude 能读取到设计内容即全部就绪。

---

## 使用说明

### 基本用法

在 Claude Code 对话中先调用命令，再提供设计稿：

```
/figma-to-code

https://www.figma.com/design/xxxxxx/MyPage?node-id=1-2
```

或直接粘贴截图：

```
/figma-to-code

[粘贴设计稿截图]
```

### 两种输入方式

| 输入方式 | 说明 | 是否需要 MCP |
|---|---|---|
| Figma 链接 | 自动调用 MCP 读取完整设计数据（颜色变量、间距、组件层级） | ✅ 需要 |
| 设计稿截图 | Claude 直接识别图片中的 UI 结构 | ❌ 不需要 |

### 生成结果示例

执行后 Claude 会输出完整的文件结构：

```
src/pages/MyPage/
  index.js              ← 页面入口，只做布局拼装
  index.less
  variables.less        ← 共享 Less 变量
  components/
    PageHeader/
      index.js
      index.less
    DataTable/
      index.js
      index.less
```

所有文件代码完整输出，可直接复制到项目中运行。

---

## 代码规范说明

生成的代码遵循以下规范：

- **框架**：React 16 + Ant Design 4
- **样式**：Less + BEM 命名（`.cm-block__element--modifier`）
- **文件后缀**：统一 `.js`，不用 `.jsx`
- **图标**：只使用 `@ant-design/icons`
- **图片**：放 `src/images/`，用 `@/images/xxx.png` 引入
- **ESLint**：禁止 `console.log`、`<label>` 标签、未使用变量

---

## 常见问题

**Q：调用 Figma 链接时报 rate limit 错误？**

A：Figma MCP 有访问频率限制。此时 Claude 会自动改用 REST API 方式读取，需要在提示中提供 Figma Token。

**Q：生成的路径中 `@/images/` 找不到模块？**

A：确认项目的 `tsconfig.json` 或 `webpack.config.js` 中配置了 `@` 路径别名指向 `src/`。

**Q：skill 命令输入后没有反应？**

A：检查文件是否放在正确的 `commands/` 目录下，且文件名为 `figma-to-code.md`。重启 Claude Code 后重试。

---


