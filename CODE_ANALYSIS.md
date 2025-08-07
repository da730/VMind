# 代码解读文档: @visactor/vmind

## 1. 项目概述

`@visactor/vmind` 是一个基于大语言模型（LLM）的智能图表库。其核心目标是让开发者和用户通过简单的自然语言指令，就能快速地完成数据处理、图表生成和可视化叙事。

- **主要功能**: 自然语言生成图表、智能数据聚合、图表类型推荐、对话式图表编辑。
- **技术栈**: TypeScript, Rush.js (Monorepo 管理), pnpm, VChart (底层图表库)。
- **运行环境**: 既可以作为前端库在浏览器中使用，也可以在 Node.js 环境中运行。

## 2. 仓库结构

这是一个使用 **Rush.js** 管理的 Monorepo（单一代码库），意味着在一个代码仓库中包含了多个相互独立的子项目（package）。这种结构便于统一管理依赖、版本和发布流程。

关键的配置文件是根目录下的 `rush.json`，它定义了仓库中所有的子项目及其存放位置。

这些子项目主要分布在以下几个目录：

- `packages/`: 存放核心的功能模块包，这些包可以被独立发布和使用。
- `tools/`: 存放内部开发工具，如打包器、代码检查工具等。
- `share/`: 存放共享的配置文件，如 `eslint`、`tsconfig` 等。
- `docs/`: 存放项目的官方文档网站。

## 3. 功能模块与文件入口

本项目的核心功能被拆分到 `packages/` 目录下的多个子项目中，每个项目负责一部分具体的功能。下面是主要功能模块的解读：

---

### 3.1. 主包: `@visactor/vmind`

这是整个项目的入口和协调者，它整合了其他包的功能，并对外提供统一的 API。

- **功能**:
  - 提供 `VMind` 主类，作为用户交互的入口。
  - 解析用户输入的自然语言指令。
  - 调度 `chart-advisor`、`calculator` 等下游模块完成任务。
  - 管理与大语言模型（LLM）的通信。
- **文件入口**:
  - `packages/vmind/src/index.ts`: 这是该包的入口文件，导出了 `VMind` 类和相关的类型定义。
  - `packages/vmind/src/core/`: 存放 `VMind` 类的实现和核心调度逻辑。
  - `packages/vmind/src/applications/`: 存放不同应用场景的实现，例如智能图表生成、数据洞察等。
  - `packages/vmind/src/llm/`: 封装了与不同大语言模型（GPT, 豆包等）的 API 调用逻辑。

---

### 3.2. 数据计算包: `@visactor/calculator`

- **功能**: 负责数据的处理和计算。当用户输入的指令需要对数据进行聚合、排序、筛选等操作时，此模块会被调用。它会将用户的自然语言指令转换为具体的 SQL 查询语句，在内存中对数据进行处理。
- **文件入口**: `packages/calculator/src/index.ts`

---

### 3.3. 图表推荐包: `@visactor/chart-advisor`

- **功能**: 根据输入的数据特征，推荐最适合的图表类型。例如，如果数据显示时间序列趋势，它可能会推荐折线图。
- **文件入口**: `packages/chart-advisor/src/index.ts`

---

### 3.4. 图表生成包: `@visactor/generate-vchart`

- **功能**: 接收结构化的信息（包括数据、图表类型、字段映射等），并生成一个符合 [VChart](https://visactor.io/vchart) 图表库规范的 `spec` 配置文件。这个 `spec` 可以被 VChart 直接渲染成图表。
- **文件入口**: `packages/generate-vchart/src/index.ts`

## 4. 高层工作流程

当用户使用 `@visactor/vmind` 时，一个典型的“自然语言生成图表”的工作流程如下：

1.  **用户调用**: 用户实例化 `VMind` 类，并调用 `generateChart` 方法，传入数据和一句自然语言指令（如 `show me the sales of different products`）。
2.  **指令解析**: `@visactor/vmind` 包接收到指令，通过 LLM 对指令进行意图识别，判断出用户想要生成的图表类型、需要分析的字段等。
3.  **数据聚合 (可选)**: 如果指令包含聚合要求（如“总销量”、“平均值”），`@visactor/vmind` 会调用 `@visactor/calculator` 包，对原始数据进行计算，生成聚合后的新数据。
4.  **图表推荐**: `@visactor/vmind` 调用 `@visactor/chart-advisor` 包，根据数据特征和用户意图，确定最合适的图表类型（如柱状图）。
5.  **生成配置**: `@visactor/vmind` 将数据、图表类型、字段映射等信息传递给 `@visactor/generate-vchart` 包。
6.  **返回结果**: `@visactor/generate-vchart` 生成 VChart 的 `spec` 配置文件，并将其返回给 `@visactor/vmind`，最终由 `vmind` 实例返回给用户。
7.  **前端渲染**: 用户拿到 `spec` 后，可以将其传递给 VChart 库进行渲染，从而在页面上看到最终的图表。

希望这份文档能帮助您更好地理解 `@visactor/vmind` 的代码结构和核心逻辑。
