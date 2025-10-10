# Digital Investigation CLI

[![License](https://img.shields.io/github/license/google-gemini/gemini-cli)](https://github.com/google-gemini/gemini-cli/blob/main/LICENSE)


**Digital Investigation CLI** 是一个基于 AI 的数字取证和电子数据调查工具，将强大的 Gemini AI 能力应用于网络安全取证、事件响应和数字证据分析领域。

> **Fork 说明**: 本项目基于 [Gemini CLI](https://github.com/google-gemini/gemini-cli) 开发，专注于数字取证和安全调查场景。

## 🔍 为什么选择 Digital Investigation CLI？

- **🎯 AI 驱动分析**: 利用 Gemini 2.5 Pro 的 1M token 上下文窗口，快速分析大量日志和取证数据
- **🛡️ 安全调查专用**: 专为数字取证、事件响应和安全分析设计
- **🔧 取证工具集成**: 内置文件系统分析、日志解析、恶意代码检测等功能
- **🔌 可扩展性**: 通过 MCP (Model Context Protocol) 集成专业取证工具
- **💻 终端优先**: 为安全分析师和取证专家量身打造
- **🛡️ 开源透明**: Apache 2.0 许可证，代码完全开放

## 📦 安装

### 系统要求

- Node.js 20 或更高版本
- macOS, Linux, 或 Windows
- Google 账户（用于 Gemini API 访问）

### 快速安装

#### 全局安装（推荐）

```bash
# Clone 本仓库
git clone https://github.com/ArchieZhao/digital-investigation-cli.git
cd digital-investigation-cli

# 安装依赖
npm install

# 全局安装
npm install -g .
```

#### 从源码运行

```bash
npm start
```

## 📋 核心功能

### 🔍 数字取证分析

- **日志分析**: 快速解析和分析系统日志、应用日志、Web 服务器日志
- **文件系统取证**: 检查文件元数据、时间线分析、删除文件恢复线索
- **恶意代码检测**: 利用 AI 识别可疑代码模式和行为特征
- **多模态分析**: 处理截图、PDF 报告等多种证据格式

### 🚨 事件响应

- **威胁情报查询**: 集成 Google Search 获取最新威胁信息和 IOC
- **自动化调查**: 根据预设规则自动执行取证任务
- **证据链追踪**: 维护完整的调查上下文和证据关联
- **报告生成**: 自动生成结构化的取证报告

### 🛠️ 取证工具集成

- **内存分析**: 分析内存转储文件，识别恶意进程
- **网络流量分析**: 解析 PCAP 文件，识别异常网络行为
- **注册表分析**: Windows 注册表取证和历史追踪
- **时间线重建**: 基于多源数据重建事件时间线

### 🔌 扩展性

- 通过 MCP 协议集成专业取证工具（如 Volatility、Autopsy、Sleuth Kit）
- 支持自定义取证脚本和工作流
- 可配置的调查模板和规则库

## 🔐 身份验证配置

本工具需要 Gemini API 访问权限，支持以下认证方式：

### 方式 1: Google 账户登录（推荐）

启动工具后选择 "Login with Google" 并完成浏览器认证：

```bash
npm start
# 或如果已全局安装
gemini
```

**优势**：

- 免费额度：60 请求/分钟，1,000 请求/天
- 自动使用 Gemini 2.5 Pro（1M token 上下文）
- 无需管理 API 密钥

### 方式 2: Gemini API Key

```bash
# 从 https://aistudio.google.com/apikey 获取密钥
export GEMINI_API_KEY="YOUR_API_KEY"
npm start
```

### 方式 3: Vertex AI（企业用户）

```bash
export GOOGLE_API_KEY="YOUR_API_KEY"
export GOOGLE_GENAI_USE_VERTEXAI=true
npm start
```

更多认证方式详见 [认证指南](./docs/get-started/authentication.md)。

## 🚀 快速开始

### 基本使用

#### 在当前目录启动

```bash
gemini
# 或使用别名
digital-investigation
```

#### 包含多个证据目录

```bash
gemini --include-directories ../evidence,../logs,../memory-dumps
```

#### 指定模型

```bash
gemini -m gemini-2.5-flash  # 快速分析
gemini -m gemini-2.5-pro    # 深度分析（默认）
```

#### 非交互模式（用于自动化脚本）

```bash
# 文本输出
gemini -p "分析这个目录中的所有日志文件，找出可疑活动"

# JSON 格式输出（便于脚本解析）
gemini -p "列出所有可疑文件及其 hash 值" --output-format json
```

### 使用示例

#### 日志分析

```bash
cd /var/log/
gemini
> 分析最近 24 小时内的系统日志，识别异常登录尝试和权限提升行为
```

#### 恶意代码检测

```bash
cd suspicious-files/
gemini
> 检查这些 PHP 文件中是否存在 webshell 或后门代码
```

#### 内存取证

```bash
cd memory-analysis/
gemini
> 分析这个内存转储文件，识别可疑进程和网络连接
```

#### 时间线重建

```bash
cd incident-evidence/
gemini
> 基于文件元数据、日志和注册表数据，重建 2024-01-15 的入侵时间线
```

#### 威胁情报查询

```bash
gemini
> 使用 Google Search 查询这个 IP 地址 192.168.1.100 的威胁情报信息
```

## 📚 文档

### 入门指南

- [**快速开始**](./docs/get-started/index.md) - 快速上手
- [**身份验证配置**](./docs/get-started/authentication.md) - 详细认证配置
- [**配置指南**](./docs/get-started/configuration.md) - 设置和自定义
- [**键盘快捷键**](./docs/cli/keyboard-shortcuts.md) - 提高效率

### 取证功能

- [**命令参考**](./docs/cli/commands.md) - 所有斜杠命令（`/help`, `/analyze` 等）
- [**自定义命令**](./docs/cli/custom-commands.md) - 创建可重用的取证命令
- [**上下文文件 (GEMINI.md)**](./docs/cli/gemini-md.md) - 为工具提供持久化上下文
- [**检查点**](./docs/cli/checkpointing.md) - 保存和恢复调查会话
- [**Token 缓存**](./docs/cli/token-caching.md) - 优化大规模分析的性能

### 取证工具

- [**内置工具概览**](./docs/tools/index.md)
  - [文件系统操作](./docs/tools/file-system.md) - 文件取证和元数据分析
  - [Shell 命令](./docs/tools/shell.md) - 执行取证命令
  - [Web 获取和搜索](./docs/tools/web-fetch.md) - 威胁情报查询
- [**MCP 服务器集成**](./docs/tools/mcp-server.md) - 集成专业取证工具
- [**自定义扩展**](./docs/extensions/index.md) - 开发和分享取证模块

### 高级主题

- [**无头模式（脚本）**](./docs/cli/headless.md) - 自动化取证工作流
- [**架构概览**](./docs/architecture.md) - 工具工作原理
- [**沙箱和安全**](./docs/cli/sandbox.md) - 安全执行环境
- [**可信文件夹**](./docs/cli/trusted-folders.md) - 按目录控制执行策略
- [**企业部署指南**](./docs/cli/enterprise.md) - 企业环境部署
- [**遥测和监控**](./docs/cli/telemetry.md) - 使用跟踪

### 疑难解答

- [**故障排除指南**](./docs/troubleshooting.md) - 常见问题和解决方案
- [**FAQ**](./docs/faq.md) - 常见问题解答
- 使用 `/bug` 命令直接从 CLI 报告问题

### 集成专业取证工具（MCP）

在 `~/.gemini/settings.json` 中配置 MCP 服务器来扩展功能：

```text
> @volatility 分析内存转储文件中的进程列表
> @sleuthkit 提取已删除文件的元数据
> @yara 使用规则扫描可疑文件
> @virustotal 查询文件 hash 的威胁情报
```

参见 [MCP 服务器集成指南](./docs/tools/mcp-server.md) 了解配置详情。

## 🤝 贡献

欢迎贡献！本项目完全开源（Apache 2.0），我们鼓励社区：

- **报告 Bug 和提出功能建议** - 提交 Issue 描述问题或需求
- **改进文档** - 完善中文文档和使用示例
- **提交代码改进** - 增强取证分析能力
- **分享取证模块** - 贡献 MCP 服务器和自定义扩展
- **提供取证案例** - 分享实际调查场景的使用经验

参见 [贡献指南](./CONTRIBUTING.md) 了解开发环境设置、编码规范和 PR 提交流程。

## 📖 资源

- **[项目 Roadmap](./ROADMAP.md)** - 查看开发计划
- **[更新日志](./docs/changelogs/index.md)** - 查看最新更新
- **[GitHub Issues](https://github.com/ArchieZhao/digital-investigation-cli/issues)** - 报告问题或请求功能
- **[上游项目](https://github.com/google-gemini/gemini-cli)** - Gemini CLI 原始项目

### 卸载

参见 [卸载指南](docs/cli/uninstall.md) 了解删除步骤。

## 🔒 取证最佳实践

### 证据完整性

- **只读挂载**: 始终以只读方式挂载证据磁盘
- **Hash 验证**: 在分析前后计算并验证证据文件的 hash 值
- **工作副本**: 在副本上进行分析，保护原始证据
- **操作日志**: 记录所有分析操作和工具使用情况

### 调查文档化

```bash
# 使用检查点功能保存调查会话
gemini
> /checkpoint save investigation-2024-01-15

# 生成调查报告
> /report generate --format markdown --output incident-report.md
```

### 证据链保护

- 使用 `--output-format json` 导出结构化分析结果
- 保存所有 AI 分析的输入和输出
- 记录使用的模型版本和参数
- 维护清晰的分析时间线

### 隐私保护

- **敏感数据处理**: 注意个人身份信息（PII）的保护
- **本地处理**: 敏感证据建议在本地环境处理
- **API 使用**: 了解使用 Gemini API 时的数据传输和存储政策
- **合规要求**: 遵守 GDPR、CCPA 等数据保护法规

## 📄 法律声明

- **许可证**: [Apache License 2.0](LICENSE)
- **隐私和服务条款**: [Terms &amp; Privacy](./docs/tos-privacy.md)
- **安全策略**: [Security Policy](SECURITY.md)
- **上游项目**: 基于 [Google Gemini CLI](https://github.com/google-gemini/gemini-cli)

## ⚠️ 免责声明

本工具仅用于**合法的数字取证和安全调查**目的。使用者必须：

- ✅ 遵守当地法律法规和计算机犯罪法
- ✅ 获得适当的授权和许可
- ✅ 在合法的调查范围内使用
- ❌ 禁止用于未经授权的访问或恶意活动
- ❌ 禁止用于侵犯他人隐私或违法行为

使用本工具造成的任何法律后果由使用者自行承担。

## 🎯 适用场景

- ✅ **事件响应**: 安全事件调查和分析
- ✅ **数字取证**: 合法的电子证据收集和分析
- ✅ **恶意软件分析**: 样本分析和行为研究
- ✅ **安全审计**: 系统安全检查和合规审计
- ✅ **威胁狩猎**: 主动威胁检测和分析
- ✅ **教育研究**: 网络安全教学和研究

---

<p align="center">
  基于 <a href="https://github.com/google-gemini/gemini-cli">Gemini CLI</a> 开发 |
  专注于数字取证和安全调查
</p>
