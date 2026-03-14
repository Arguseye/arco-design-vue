# 在 arco-design-vue 项目中使用 DeepSeek-R1 模型指南

> 本文档介绍如何在 GitHub Copilot 或本地开发环境中结合 [DeepSeek-R1](https://github.com/deepseek-ai/DeepSeek-R1) 模型进行高效开发，适用于 arco-design-vue 项目的贡献者和使用者。

---

## 目录

1. [DeepSeek-R1 模型简介](#1-deepseek-r1-模型简介)
2. [集成方式概览](#2-集成方式概览)
3. [通过 OpenAI 兼容 API 调用](#3-通过-openai-兼容-api-调用)
4. [本地部署推理服务](#4-本地部署推理服务)
5. [结合 GitHub Copilot 使用 DeepSeek-R1](#5-结合-github-copilot-使用-deepseek-r1)
6. [示例代码](#6-示例代码)
7. [常见问题解答（FAQ）](#7-常见问题解答faq)
8. [官方文档与资源](#8-官方文档与资源)

---

## 1. DeepSeek-R1 模型简介

**DeepSeek-R1** 是由 [深度求索（DeepSeek）](https://www.deepseek.com/) 开发的一系列高性能大语言推理模型。其核心特点如下：

| 特性 | 描述 |
|------|------|
| **推理能力强** | 专为复杂推理、数学、代码生成等任务优化，在 AIME、MATH-500、SWE-bench 等权威基准上达到顶尖水平 |
| **开源可商用** | 以 MIT 许可证开源，完整权重公开发布在 HuggingFace，可免费用于商业项目 |
| **多规格可选** | 提供从 1.5B 到 671B 不同参数量的模型，可按资源条件灵活选择 |
| **OpenAI 兼容** | API 接口与 OpenAI Chat Completions 规范高度兼容，便于迁移和集成 |
| **思维链输出** | 支持输出详细的推理过程（`<think>...</think>`），便于调试和理解模型决策 |

**典型应用场景**：

- Vue 组件代码自动生成与补全
- TypeScript 类型推断与接口设计
- 测试用例自动生成
- 代码重构与 Bug 分析
- 技术文档撰写与翻译

---

## 2. 集成方式概览

```
DeepSeek-R1 集成方式
├── 云端 API
│   ├── DeepSeek 官方平台 (platform.deepseek.com)
│   ├── GitHub Models (github.com/marketplace/models)
│   └── 其他第三方托管（硅基流动、Together AI 等）
└── 本地部署
    ├── Ollama（推荐，适合个人开发者）
    ├── vLLM（适合生产级推理服务）
    └── llama.cpp（适合资源受限环境）
```

---

## 3. 通过 OpenAI 兼容 API 调用

### 3.1 使用 DeepSeek 官方平台

1. 注册并登录 [DeepSeek 开放平台](https://platform.deepseek.com/)
2. 创建 API Key
3. 安装客户端（以 Node.js 为例）：

```bash
npm install openai
```

4. 在项目根目录新建 `.env` 文件（**请勿提交到 Git**）：

```env
DEEPSEEK_API_KEY=your_api_key_here
```

5. 调用示例：

```typescript
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: process.env.DEEPSEEK_API_KEY,
  baseURL: 'https://api.deepseek.com',
});

const response = await client.chat.completions.create({
  model: 'deepseek-reasoner', // DeepSeek-R1 对应的模型名
  messages: [
    {
      role: 'user',
      content: '请帮我为 arco-design-vue 的 Button 组件编写单元测试。',
    },
  ],
});

console.log(response.choices[0].message.content);
```

### 3.2 使用 GitHub Models

GitHub Models 提供了对 DeepSeek-R1 的托管访问，无需注册额外账号（需要 GitHub 账号）：

1. 访问 [GitHub Models 市场](https://github.com/marketplace/models)，搜索 `DeepSeek-R1`
2. 在模型页面获取你的 GitHub Personal Access Token
3. 调用示例：

```typescript
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: process.env.GITHUB_TOKEN,
  baseURL: 'https://models.inference.ai.azure.com',
});

const response = await client.chat.completions.create({
  model: 'DeepSeek-R1',
  messages: [
    {
      role: 'user',
      content: '解释 Vue 3 的 Composition API 的核心优势。',
    },
  ],
});

console.log(response.choices[0].message.content);
```

---

## 4. 本地部署推理服务

### 4.1 使用 Ollama（推荐）

[Ollama](https://ollama.com/) 是最简单的本地模型运行方案，适合开发者快速上手。

**安装 Ollama**：

```bash
# macOS / Linux
curl -fsSL https://ollama.com/install.sh | sh

# Windows：访问 https://ollama.com/download 下载安装包
```

**拉取并运行 DeepSeek-R1**：

```bash
# 轻量版（适合本地开发，需约 8GB 内存）
ollama pull deepseek-r1:7b

# 中等规格（需约 16GB 内存）
ollama pull deepseek-r1:14b

# 启动服务（默认监听 http://localhost:11434）
ollama serve
```

**通过 OpenAI 兼容接口调用**：

```typescript
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: 'ollama', // Ollama 不需要真实 API Key
  baseURL: 'http://localhost:11434/v1',
});

const response = await client.chat.completions.create({
  model: 'deepseek-r1:7b',
  messages: [
    { role: 'user', content: '用 Vue 3 的 setup 语法糖写一个计数器组件。' },
  ],
});

console.log(response.choices[0].message.content);
```

### 4.2 使用 vLLM（生产级部署）

适合需要高并发推理服务的场景（需要 GPU）：

```bash
pip install vllm

python -m vllm.entrypoints.openai.api_server \
  --model deepseek-ai/DeepSeek-R1-Distill-Qwen-7B \
  --port 8000
```

启动后同样可通过 OpenAI 兼容接口访问 `http://localhost:8000`。

---

## 5. 结合 GitHub Copilot 使用 DeepSeek-R1

### 5.1 在 VS Code 中切换 Copilot 模型

GitHub Copilot 从 2025 年起支持在 Chat 面板中选择不同的底层模型，包括 DeepSeek-R1（需在支持的区域且具备相应权限）：

1. 打开 VS Code，确保已安装 **GitHub Copilot** 和 **GitHub Copilot Chat** 扩展
2. 按 `Ctrl+Shift+P`（macOS：`Cmd+Shift+P`），输入 `GitHub Copilot: Select Model`
3. 在弹出的模型列表中选择 **DeepSeek-R1**

> **提示**：如果列表中没有 DeepSeek-R1，可能需要通过 GitHub Models 页面申请访问权限，或等待在你所在区域的推出。

### 5.2 在 Copilot Chat 中使用 DeepSeek-R1 辅助 arco-design-vue 开发

打开 Copilot Chat 侧边栏，确认当前模型为 DeepSeek-R1，然后可以进行以下交互：

#### 自动生成 Vue 组件

```
@workspace 基于 arco-design-vue，帮我生成一个带分页功能的用户列表页面组件。
要求：
- 使用 <script setup lang="ts">
- 使用 a-table、a-pagination 组件
- 包含 loading 状态处理
- 数据类型用 TypeScript interface 定义
```

#### 代码审查与重构

```
/fix 这段代码有什么潜在的性能问题？如何用 Vue 3 的最佳实践改写？

[粘贴你的组件代码]
```

#### 生成单元测试

```
/tests 为以下 arco-design-vue 自定义组件生成 Vitest 单元测试，
覆盖主要的 props、emits 和用户交互场景：

[粘贴你的组件代码]
```

#### 文档生成

```
为以下 Vue 组件的 props 和 emits 生成中文文档注释（JSDoc 格式）：

[粘贴你的组件代码]
```

### 5.3 在编辑器内联补全中利用 DeepSeek-R1

在 VS Code 的 Copilot 内联补全（Tab 补全）中，通过清晰的注释引导模型生成高质量代码：

```vue
<script setup lang="ts">
// 使用 arco-design-vue 的 a-form 实现一个登录表单
// 包含用户名和密码字段，带表单验证规则
// 点击登录按钮后调用 login API，处理 loading 状态

</script>
```

输入注释后，Copilot 会自动补全完整的组件实现代码。

---

## 6. 示例代码

### 6.1 调用 DeepSeek-R1 生成 Vue 组件脚本

以下脚本可直接在 arco-design-vue 项目根目录运行，调用 DeepSeek-R1 API 生成组件代码：

```typescript
// scripts/generate-component.ts
import OpenAI from 'openai';
import { writeFileSync } from 'fs';

const client = new OpenAI({
  apiKey: process.env.DEEPSEEK_API_KEY,
  baseURL: 'https://api.deepseek.com',
});

async function generateComponent(componentName: string, description: string) {
  console.log(`正在生成组件：${componentName}...`);

  const response = await client.chat.completions.create({
    model: 'deepseek-reasoner',
    messages: [
      {
        role: 'system',
        content: `你是一个 Vue 3 + arco-design-vue 专家。
请生成符合以下规范的 Vue 组件：
- 使用 <script setup lang="ts">
- 使用 arco-design-vue 组件库
- 包含完整的 TypeScript 类型
- 遵循 arco-design-vue 的代码风格`,
      },
      {
        role: 'user',
        content: `生成一个名为 ${componentName} 的 Vue 组件。\n需求：${description}`,
      },
    ],
  });

  const content = response.choices[0].message.content ?? '';

  // 提取 Vue 代码块
  const match = content.match(/```vue\n([\s\S]*?)```/);
  if (match) {
    const filePath = `packages/web-vue/components/${componentName.toLowerCase()}/${componentName}.vue`;
    writeFileSync(filePath, match[1]);
    console.log(`✅ 组件已生成：${filePath}`);
  } else {
    console.log('生成内容：\n', content);
  }
}

// 示例调用
generateComponent(
  'UserCard',
  '展示用户头像、姓名、职位和联系方式的卡片组件，包含编辑和删除操作按钮'
);
```

**运行方式**：

```bash
DEEPSEEK_API_KEY=your_key npx tsx scripts/generate-component.ts
```

### 6.2 解析思维链输出

DeepSeek-R1 会在 `reasoning_content` 字段中输出推理过程，可用于调试：

```typescript
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: process.env.DEEPSEEK_API_KEY,
  baseURL: 'https://api.deepseek.com',
});

const response = await client.chat.completions.create({
  model: 'deepseek-reasoner',
  messages: [
    {
      role: 'user',
      content: '分析这段 Vue 代码的性能问题并给出优化建议：\n\n[你的代码]',
    },
  ],
});

const message = response.choices[0].message as {
  reasoning_content?: string;
  content: string | null;
};

if (message.reasoning_content) {
  console.log('🧠 模型推理过程：\n', message.reasoning_content);
}
console.log('\n✅ 最终回答：\n', message.content);
```

### 6.3 流式输出示例

```typescript
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: process.env.DEEPSEEK_API_KEY,
  baseURL: 'https://api.deepseek.com',
});

const stream = await client.chat.completions.create({
  model: 'deepseek-reasoner',
  messages: [
    { role: 'user', content: '帮我优化这个 Vue 组件的性能' },
  ],
  stream: true,
});

for await (const chunk of stream) {
  const delta = chunk.choices[0]?.delta;
  // 输出推理过程
  if ((delta as { reasoning_content?: string }).reasoning_content) {
    process.stdout.write((delta as { reasoning_content?: string }).reasoning_content!);
  }
  // 输出最终内容
  if (delta?.content) {
    process.stdout.write(delta.content);
  }
}
```

---

## 7. 常见问题解答（FAQ）

### Q1：DeepSeek-R1 和 DeepSeek-V3 有什么区别？

**A**：
- **DeepSeek-R1** 是专注于**推理**的模型，通过强化学习训练，特别擅长需要逐步思考的任务（数学、代码分析、逻辑推理）。它会输出详细的"思维链"。
- **DeepSeek-V3** 是通用对话模型，响应速度更快，适合日常问答和简单代码生成任务。
- 在 arco-design-vue 开发中，**复杂组件设计、Bug 分析**推荐用 R1；**快速代码补全**推荐用 V3。

### Q2：免费使用限额是多少？

**A**：
- **DeepSeek 官方平台**：新用户注册后有免费额度，之后按 token 付费。具体价格见[官方定价页](https://platform.deepseek.com/api-docs/pricing/)。
- **GitHub Models**：在 GitHub 上通过 GitHub Models 访问时，个人用户有每日免费请求配额（具体限额见 [GitHub Models 文档](https://docs.github.com/en/github-models)）。
- **本地 Ollama 部署**：完全免费，无请求限制，但需要本地硬件支持。

### Q3：API 调用出现 401 错误怎么办？

**A**：
1. 检查 API Key 是否正确复制（注意前后空格）
2. 确认 `baseURL` 与你使用的平台匹配（官方平台用 `https://api.deepseek.com`，GitHub Models 用 `https://models.inference.ai.azure.com`）
3. 检查账户余额是否充足
4. 确认 API Key 未过期

### Q4：本地 Ollama 运行太慢怎么办？

**A**：
- 优先选择 **7B** 或 **1.5B** 参数量的蒸馏版本（如 `deepseek-r1:1.5b`），这些版本在普通笔记本电脑上即可流畅运行
- 确保 Ollama 已启用 GPU 加速（Apple Silicon Mac 默认启用，NVIDIA GPU 需安装 CUDA）
- 对于纯 CPU 运行，可以通过 `OLLAMA_NUM_THREADS` 环境变量增加线程数

### Q5：GitHub Copilot 中没有看到 DeepSeek-R1 怎么办？

**A**：
1. 确认 GitHub Copilot 和 Copilot Chat 扩展已更新到最新版本
2. DeepSeek-R1 在 GitHub Models 中可能处于预览阶段，需要通过 [GitHub Models 页面](https://github.com/marketplace/models) 申请访问
3. 也可以安装 [Continue](https://github.com/continuedev/continue) 扩展，手动配置 DeepSeek-R1 作为 AI 助手后端（支持 Ollama 和 API 两种模式）

### Q6：如何在 arco-design-vue 的 CI 流程中使用 DeepSeek-R1？

**A**：可以在 GitHub Actions 中集成 DeepSeek-R1 进行代码审查，在 `.github/workflows/` 中添加工作流，使用 `DEEPSEEK_API_KEY` Secret 调用 API。注意：在 CI 环境中使用时，请评估 API 调用成本，建议仅在关键场景（如 PR 时自动生成测试）中使用。

### Q7：如何保护 API Key 安全？

**A**：
- **永远不要**将 API Key 硬编码在源码中
- 使用 `.env` 文件存储，并将 `.env` 加入 `.gitignore`（arco-design-vue 项目的 `.gitignore` 已覆盖 `.env` 文件）
- 在 CI/CD 环境中使用 GitHub Secrets 存储 API Key
- 定期轮换 API Key，并为不同环境（开发/生产）使用不同的 Key

---

## 8. 官方文档与资源

### DeepSeek-R1 官方资源

| 资源 | 链接 |
|------|------|
| 📦 GitHub 仓库 | [deepseek-ai/DeepSeek-R1](https://github.com/deepseek-ai/DeepSeek-R1) |
| 🤗 HuggingFace 模型 | [deepseek-ai/DeepSeek-R1](https://huggingface.co/deepseek-ai/DeepSeek-R1) |
| 📄 技术报告（论文） | [DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning](https://arxiv.org/abs/2501.12948) |
| 🔑 开放平台 API | [platform.deepseek.com/api-docs](https://platform.deepseek.com/api-docs/) |
| 💰 API 定价 | [platform.deepseek.com/api-docs/pricing](https://platform.deepseek.com/api-docs/pricing/) |

### GitHub Models 资源

| 资源 | 链接 |
|------|------|
| 🛒 GitHub Models 市场 | [github.com/marketplace/models](https://github.com/marketplace/models) |
| 📖 GitHub Models 文档 | [docs.github.com/en/github-models](https://docs.github.com/en/github-models) |

### 本地部署工具

| 工具 | 链接 | 适用场景 |
|------|------|----------|
| Ollama | [ollama.com](https://ollama.com/) | 个人开发者，一键本地部署 |
| vLLM | [github.com/vllm-project/vllm](https://github.com/vllm-project/vllm) | 生产级高并发推理 |
| llama.cpp | [github.com/ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp) | 轻量化，适合 CPU 环境 |

### arco-design-vue 相关资源

| 资源 | 链接 |
|------|------|
| 官网 | [arco.design/vue](https://arco.design/vue) |
| GitHub 仓库 | [arco-design/arco-design-vue](https://github.com/arco-design/arco-design-vue) |
| 贡献指南 | [CONTRIBUTING.zh-CN.md](../CONTRIBUTING.zh-CN.md) |

---

*最后更新：2026 年 3 月*
