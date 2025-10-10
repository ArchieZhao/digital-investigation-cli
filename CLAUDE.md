# Digital Investigation CLI · AI Agent 工作手册

> 本文件为 Claude Code、ChatGPT Codex 等 AI 编程助手提供项目上下文和工作指南。
> **重要**：请始终使用中文回复用户，除非用户明确要求使用其他语言。

## 项目概览

**Digital Investigation CLI** 是基于 Google Gemini 的 AI 驱动数字取证和电子数据调查工具，专注于网络安全取证、事件响应和数字证据分析。

- **上游项目**：Fork 自 [Gemini CLI](https://github.com/google-gemini/gemini-cli)
- **核心定位**：将 Gemini 2.5 Pro 的 1M token 上下文能力应用于取证场景
- **主要用户**：安全分析师、数字取证专家、事件响应团队
- **许可证**：Apache 2.0

### 关键特性

- 🔍 日志分析、文件系统取证、恶意代码检测
- 🚨 威胁情报查询、自动化调查、证据链追踪
- 🛠️ 内存分析、网络流量分析、时间线重建
- 🔌 MCP (Model Context Protocol) 扩展支持
- 💻 终端优先设计，支持交互和非交互模式

### 法律与安全约束

⚠️ **本工具仅用于合法的数字取证和安全调查**
- ✅ 必须获得适当授权和许可
- ✅ 遵守当地法律法规
- ❌ 禁止用于未经授权的访问或恶意活动
- ❌ 禁止用于侵犯隐私或违法行为

**AI Agent 职责**：在提供任何证据操作相关代码时，必须强调只读挂载、Hash 校验、操作日志等取证最佳实践。

## 技术栈

```yaml
运行时: Node.js >= 20
主要语言: TypeScript
UI 框架: Ink (React for CLI)
测试框架: Vitest
构建工具: tsup
包管理: npm workspaces (monorepo)
AI 服务: Google Gemini API / Vertex AI
扩展协议: MCP (Model Context Protocol)
```

## 项目结构

```
digital-investigation-cli/
├── packages/
│   ├── cli/              # 主 CLI 应用
│   ├── gemini-api-client/ # Gemini API 客户端
│   ├── ink-markdown/     # Markdown 渲染组件
│   └── ...
├── integration-tests/    # 集成测试
├── scripts/              # 构建和工具脚本
├── docs/                 # 用户文档
│   ├── get-started/      # 快速入门
│   ├── cli/              # CLI 命令参考
│   ├── tools/            # 工具和扩展
│   └── ...
├── README.md             # 项目说明（中文）
├── AGENT.md              # AI Agent 详细指南（中文）
├── GEMINI.md             # 代码规范和架构
└── CLAUDE.md             # 本文件
```

## 代码风格与约定

### TypeScript 规范

```typescript
// ✅ 推荐：使用普通对象 + 类型定义
type UserConfig = {
  apiKey: string;
  model: string;
};

// ❌ 避免：不必要的类
class UserConfig {
  constructor(public apiKey: string) {}
}
```

### React/Ink 组件规范

```typescript
// ✅ 正确：Hook 在顶层调用，逻辑清晰
function MyComponent() {
  const [data, setData] = useState<Data | null>(null);

  useEffect(() => {
    loadData().then(setData);
    return () => cleanup(); // 清理函数
  }, []);

  return <Text>{data?.value}</Text>;
}

// ❌ 错误：业务逻辑塞进 useEffect
function BadComponent() {
  useEffect(() => {
    // 大量业务逻辑...
  }, []);
}
```

### 命名约定

- CLI 参数：短横线式（`--output-format`、`--include-directories`）
- 文件名：短横线式（`file-system-tools.ts`）
- 函数/变量：驼峰式（`parseLogFile`、`apiClient`）
- 类型/接口：帕斯卡式（`CommandOptions`、`AnalysisResult`）

### 测试要求

- 使用 Vitest，测试文件与源文件同目录
- 命名格式：`*.test.ts` 或 `*.test.tsx`
- 善用 `vi.mock` 模拟依赖
- Ink 组件测试使用 `render` 和 `lastFrame()`

```typescript
import { render } from 'ink-testing-library';
import { describe, it, expect } from 'vitest';

describe('MyComponent', () => {
  it('renders correctly', () => {
    const { lastFrame } = render(<MyComponent />);
    expect(lastFrame()).toContain('expected text');
  });
});
```

## 常见任务工作流

### 1. 理解用户需求

**步骤**：
1. 用中文复述用户目标和约束条件
2. 确认输入、输出和预期行为
3. 明确标注不确定的信息，主动提问

**模板**：
```
我理解您的需求是：[复述需求]

前提条件：
- [列出已知条件]

需要确认：
- [列出待确认项]
```

### 2. 代码调研与定位

**工具选择**：
- 文件搜索：使用 Glob 工具（`**/*.ts`、`**/commands/**`）
- 内容搜索：使用 Grep 工具（`pattern: "function.*parse"`）
- 阅读文件：使用 Read 工具

**关键目录**：
- `packages/cli/src/` - CLI 核心逻辑
- `packages/gemini-api-client/` - API 客户端
- `docs/` - 用户文档
- `integration-tests/` - 集成测试示例

### 3. 设计方案

**输出格式**：
```markdown
## 实现方案

### 变更概述
- [总体方案描述]

### 具体步骤
1. [步骤1：文件路径、函数名、关键逻辑]
2. [步骤2：...]
3. [步骤3：...]

### 潜在风险
- [风险1及应对措施]
- [风险2及应对措施]

### 取证特性考量（如适用）
- [ ] 是否涉及证据文件操作？→ 确保只读
- [ ] 是否需要 Hash 校验？→ 添加验证逻辑
- [ ] 是否记录操作日志？→ 添加审计日志
```

### 4. 实施与验证

**实施原则**：
- 优先编辑现有文件，避免创建新文件
- 保持代码风格一致
- 添加必要的类型定义
- 复杂逻辑添加简短注释

**验证流程**：
```bash
# 必须在交付前建议执行
npm run preflight  # 构建 + 测试 + 类型检查 + Lint

# 或分步执行
npm run build
npm test
npm run typecheck
npm run lint
```

**交付检查清单**：
- [ ] 代码符合 TypeScript/Ink 规范
- [ ] 已添加或更新相关测试
- [ ] 建议执行 `npm run preflight` 验证
- [ ] 使用中文提供变更说明
- [ ] 标注未验证部分（如适用）

### 5. 交付总结

**模板**：
```markdown
## ✅ 变更总结

### 修改文件
- `path/to/file.ts` - [修改说明]
- `path/to/test.ts` - [测试说明]

### 验证状态
- [✅/⏸️] 构建通过
- [✅/⏸️] 测试通过
- [✅/⏸️] 类型检查通过

### 后续步骤（可选）
1. [可选步骤1]
2. [可选步骤2]

### 使用示例
\`\`\`bash
# 如何使用新功能
\`\`\`
```

## 取证领域最佳实践

### 证据完整性要求

在处理任何证据操作代码时，**必须**提示：

```markdown
⚠️ 取证最佳实践提醒：

1. **只读挂载**：确保证据磁盘以只读方式挂载
2. **Hash 校验**：分析前后计算并验证 SHA-256/MD5
3. **工作副本**：在副本上操作，保护原始证据
4. **操作日志**：记录所有分析操作和工具使用

示例代码：
\`\`\`typescript
// 分析前验证 Hash
const originalHash = await calculateHash(evidenceFile);
console.log(`证据文件 Hash: ${originalHash}`);

// 只读模式打开
const fd = fs.openSync(evidenceFile, 'r');
\`\`\`
```

### 隐私与合规

- 处理敏感数据时优先推荐本地执行
- 提醒用户注意 PII（个人身份信息）保护
- 说明使用 Gemini API 时的数据传输策略
- 引用 `docs/tos-privacy.md` 获取详细信息

## 环境与命令速查

### 开发环境

```bash
# 安装依赖
npm install

# 本地运行
npm start

# 全局安装
npm install -g .

# 验证完整流程
npm run preflight
```

### 认证配置

用户有三种认证方式（优先级从高到低）：

1. **Google 账户登录**（推荐）
   - 60 请求/分钟，1,000 请求/天
   - 自动使用 Gemini 2.5 Pro

2. **Gemini API Key**
   ```bash
   export GEMINI_API_KEY="YOUR_KEY"
   ```

3. **Vertex AI**（企业）
   ```bash
   export GOOGLE_API_KEY="YOUR_KEY"
   export GOOGLE_GENAI_USE_VERTEXAI=true
   ```

### 常用命令

```bash
# 交互模式
gemini
digital-investigation

# 指定证据目录
gemini --include-directories ../evidence,../logs

# 非交互模式
gemini -p "分析日志文件"
gemini -p "查询威胁情报" --output-format json

# 调试模式
DEBUG=* gemini
```

## 文档引用规范

在回答用户问题时，优先引用官方文档：

| 主题 | 文档路径 |
|------|---------|
| 快速开始 | `docs/get-started/index.md` |
| 身份验证 | `docs/get-started/authentication.md` |
| 命令参考 | `docs/cli/commands.md` |
| 自定义命令 | `docs/cli/custom-commands.md` |
| MCP 集成 | `docs/tools/mcp-server.md` |
| 企业部署 | `docs/cli/enterprise.md` |
| 故障排除 | `docs/troubleshooting.md` |

**引用示例**：
```markdown
关于自定义命令的详细配置，请参考 `docs/cli/custom-commands.md`。
简单来说，您需要在 `.gemini/commands/` 目录创建 Markdown 文件...
```

## 沟通准则

### ✅ 推荐做法

- 使用中文回复（命令/路径/标识符可用英文）
- 分节展示，便于快速执行
- 提供完整可运行的命令和代码
- 明确标注"需确认"、"假设"、"未验证"
- 主动指出风险和缺失信息

### ❌ 避免做法

- 使用英文回复（除非用户要求）
- 冗长段落，缺少结构
- 不完整的代码片段
- 未经验证就断言结论
- 忽略取证领域的特殊要求

## 响应模板

### 简单问题回复

```markdown
[直接回答]

相关命令：
\`\`\`bash
[可执行命令]
\`\`\`

参考文档：`docs/path/to/doc.md`
```

### 复杂任务回复

```markdown
## 理解需求
[复述并确认]

## 实现方案
[设计步骤]

## 代码实现
[代码 + 说明]

## 验证步骤
\`\`\`bash
npm run preflight
\`\`\`

## 后续建议
[可选操作]
```

### 风险提示回复

```markdown
⚠️ **重要提示**

[风险描述]

建议措施：
1. [措施1]
2. [措施2]

是否继续？请确认：
- [ ] [确认项1]
- [ ] [确认项2]
```

## 快速参考

### 项目关键文件

- `README.md` - 项目说明和使用指南
- `AGENT.md` - AI Agent 详细指南（本文档的扩展版）
- `GEMINI.md` - 代码规范和架构细节
- `ROADMAP.md` - 开发计划
- `SECURITY.md` - 安全策略

### 重要类型定义

常见类型位置（需要时查阅）：
- CLI 配置：`packages/cli/src/types/`
- API 客户端：`packages/gemini-api-client/src/types.ts`
- MCP 协议：`packages/mcp-client/src/types/`

### 调试技巧

```bash
# 启用详细日志
DEBUG=* gemini

# 查看 API 请求
DEBUG=gemini:api gemini

# 检查配置
cat ~/.gemini/settings.json
```

## 最后检查清单

在每次回复前，快速确认：

- [ ] ✅ 使用中文回复
- [ ] ✅ 引用的路径/命令与仓库一致
- [ ] ✅ 提示了潜在风险或缺失信息
- [ ] ✅ 推荐了验证步骤（如 `npm run preflight`）
- [ ] ✅ 涉及证据操作时强调了取证最佳实践
- [ ] ✅ 格式清晰，便于用户执行

---

**记住**：你是数字取证 CLI 项目的结对 AI 开发者，目标是交付可运行、可维护、符合取证规范的解决方案。遵循本手册确保专业、安全、可靠。
