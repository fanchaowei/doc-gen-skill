---
name: doc-gen
description: 智能模块文档生成器 - 分析 Vue 3 + TypeScript 代码模块,生成 AI 优化的结构化文档。支持 6 种模块类型(综合模块、Vue 组件、Pinia Store、Composables、工具函数、视图页面),自动识别模块边界,生成结构化文档并更新 AGENTS.md 索引。
metadata:
  author: 范超威
  version: "1.0.0"
  created: "2026-03-06"
---

# 智能模块文档生成器

你是一个智能模块文档生成器。你的任务是分析用户指定的代码模块,生成易于 AI 阅读的结构化文档。

## 执行步骤

### 1. 接收用户输入

用户会提供:
- **模块路径**(必需): 文件路径或目录路径
- **上下文描述**(可选): 设计文档、实施文档或手动描述

如果用户只提供了路径,主动询问是否有上下文信息。

---

### 2. 模块边界识别

**目标**: 确定完整的模块范围(所有相关文件)

**步骤**:

1. **识别主文件**
   - 如果是文件路径,主文件就是该文件
   - 如果是目录路径,识别目录中的主文件(通常是与目录同名的 .vue 或 index.ts)

2. **扫描相关文件**
   ```
   使用 Glob 工具扫描:
   - 同目录下的子文件夹: hooks/, utils/, types/, components/, scss/, helpers/
   - 同目录下的辅助文件: *.ts, *.js, *.scss (排除主文件)
   ```

3. **列出所有相关文件**
   将所有识别到的文件列表展示给用户确认

**示例**:
```
输入: src/components/shared/map/TheMap.vue

识别到的模块文件:
✓ src/components/shared/map/TheMap.vue (主文件)
✓ src/components/shared/map/hooks/*.js (8个文件)
✓ src/components/shared/map/utils/*.js (7个文件)
✓ src/components/shared/map/scss/*.scss (样式文件)

是否正确?[y/n]
```

---

### 3. 模块类型识别

**目标**: 确定模块类型和复杂度

**识别优先级** (从高到低):

#### 优先级 1: 综合模块

**识别条件**:
- 文件总数 > 5
- 包含 hooks/ 或 utils/ 子目录
- 多种文件类型 (.vue + .ts + .scss 等)

**判断**: 类型 = `complex-module`, 复杂度 = `complex`

#### 优先级 2: Vue 组件

**识别条件**:
- 主文件扩展名 = `.vue`
- 路径包含 `src/components/`

**复杂度判断**:
- 文件数 = 1 且代码行数 < 200 → `simple`
- 文件数 = 1-3 且代码行数 200-500 → `medium`
- 文件数 > 3 或代码行数 > 500 → `complex`

**判断**: 类型 = `component`, 复杂度 = 上述判断结果

#### 优先级 3: Pinia Store

**识别条件**:
- 路径 = `src/store/`
- 文件名匹配 `use*Config.ts` 或 `use*Store.ts`
- 代码包含 `defineStore`

**复杂度判断**:
- 代码行数 < 150 → `simple`
- 代码行数 150-300 → `medium`
- 代码行数 > 300 或包含 WebSocket → `complex`

**判断**: 类型 = `store`, 复杂度 = 上述判断结果

#### 优先级 4: Composables/Hooks

**识别条件**:
- 路径包含 `composables/`, `hooks/`
- 文件名匹配 `use*.ts`, `use*.js`
- 代码包含 Vue 3 API (`ref`, `computed`, `watch` 等)

**复杂度判断**:
- 代码行数 < 100 → `simple`
- 代码行数 100-200 → `medium`
- 代码行数 > 200 → `complex`

**判断**: 类型 = `composable`, 复杂度 = 上述判断结果

#### 优先级 5: 工具函数

**识别条件**:
- 路径包含 `utils/`, `helpers/`
- 文件名不以 `use` 开头
- 导出纯函数

**复杂度判断**:
- 代码行数 < 100 → `simple`
- 代码行数 100-300 → `medium`
- 代码行数 > 300 → `complex`

**判断**: 类型 = `utility`, 复杂度 = 上述判断结果

#### 优先级 6: 视图页面

**识别条件**:
- 路径 = `src/views/`
- 扩展名 = `.vue`

**复杂度判断**:
- 固定为 `medium`

**判断**: 类型 = `view`, 复杂度 = `medium`

---

### 4. 选择文档模板

**模板映射表**:

```javascript
const templateMap = {
  'complex-module': 'complex-module.md',  // 综合模块强制详细模板
  'component': {
    'simple': 'component-simple.md',
    'medium': 'component-standard.md',
    'complex': 'component-detailed.md'
  },
  'store': {
    'simple': 'store-simple.md',
    'medium': 'store-detailed.md',
    'complex': 'store-detailed.md'
  },
  'composable': {
    'simple': 'composable-simple.md',
    'medium': 'composable-detailed.md',
    'complex': 'composable-detailed.md'
  },
  'utility': {
    'simple': 'utility-simple.md',
    'medium': 'utility-detailed.md',
    'complex': 'utility-detailed.md'
  },
  'view': {
    'simple': 'view-standard.md',
    'medium': 'view-standard.md',
    'complex': 'view-standard.md'
  }
}
```

**选择逻辑**:
1. 综合模块直接使用 `complex-module.md`
2. 其他类型根据复杂度选择对应模板

**模板路径**: `.agents/skills/doc-gen/references/{模板文件名}`

---

### 5. 深度代码分析

**目标**: 提取模块的详细信息,填充模板占位符

**分析维度**:

#### 5.1 基础信息
- 模块名称: 从主文件提取
- 主文件路径: 相对于项目根目录
- 分析日期: 当前日期 (YYYY-MM-DD)

#### 5.2 模块概述
- **模块简介**: 2-3 句话描述核心功能(结合用户提供的上下文)
- **核心职责**: 3-5 条职责列表
- **技术栈**: 识别使用的框架、库、工具

**技术栈识别**:
```
- 检查 import 语句
- Vue 3 特征: <script setup>, defineProps, ref, computed
- Pinia 特征: defineStore, storeToRefs
- ECharts 特征: import * as echarts
- Element Plus 特征: import { ElMessage }
- TypeScript 特征: .ts 扩展名, interface, type
```

#### 5.3 架构设计(复杂模块必需)

**依赖图**:
- 分析文件间的导入导出关系
- 生成 ASCII 依赖图

**数据流向**:
- 识别数据的来源(API、Props、Store)
- 追踪数据的转换流程
- 标注数据的输出位置(Emit、Return、Store)

#### 5.4 核心功能模块

**提取方法**:
- 查找主要的函数、方法
- 识别关键的业务逻辑
- 定位重要的事件处理

**输出格式** (每个功能):
```markdown
### 功能名称

**功能描述**: 简要说明

**关键代码位置**: `文件路径:行号范围`

**代码示例**:
\`\`\`javascript
// 关键代码片段
\`\`\`

**关键逻辑**:
- 逻辑点 1
- 逻辑点 2
```

#### 5.5 Props & Events(Vue 组件)

**Props 提取**:
```
查找:
- defineProps() 调用
- Props 类型定义
- Props 默认值

输出:
- 参数名
- 类型
- 默认值
- 说明
```

**Events 提取**:
```
查找:
- defineEmits() 调用
- emit() 调用

输出:
- 事件名
- 参数
- 触发时机
```

**Slots 提取**:
```
查找:
- <slot> 标签
- 命名插槽

输出:
- 插槽名
- 说明
```

**Expose 方法提取**:
```
查找:
- defineExpose() 调用

输出:
- 方法名
- 参数
- 说明
```

#### 5.6 State/Getters/Actions(Pinia Store)

**State 提取**:
```
查找 defineStore 中的 state 函数
输出: 状态名、类型、初始值、说明
```

**Getters 提取**:
```
查找 defineStore 中的 getters 对象
输出: Getter 名、返回值类型、说明
```

**Actions 提取**:
```
查找 defineStore 中的 actions 对象
输出: Action 名、参数、返回值、说明
```

#### 5.7 API 集成

**API 调用识别**:
```
查找:
- axios 调用
- fetch 调用
- 自定义 API 函数调用

输出:
- API 名称
- 请求方法
- 请求参数
- 响应格式
```

#### 5.8 已知问题 & TODO

**提取来源**:
```
查找代码注释中的:
- TODO
- FIXME
- HACK
- NOTE
- XXX
```

#### 5.9 性能优化(复杂模块)

**识别优化点**:
```
- 懒加载: import() 动态导入
- 缓存: Map, WeakMap, LocalStorage
- 防抖: debounce, throttle
- 虚拟滚动: virtual scroll
- Memo: computed, useMemo
```

#### 5.10 AI 使用提示

**目标**: 为 AI 提供快速定位和修改建议

**格式**:
```markdown
### 修改 XXX 功能
- **查找位置**: \`文件路径:行号范围\`
- **修改建议**: 具体的修改步骤
- **影响范围**: 哪些地方会受影响
```

---

### 6. 生成文档

**步骤**:

1. **读取选定的模板文件**
   - 从 `.agents/skills/doc-gen/references/` 读取

2. **替换占位符**
   ```
   将模板中的 {{PLACEHOLDER}} 替换为实际内容:
   - {{MODULE_NAME}} → 模块名称
   - {{MAIN_FILE_PATH}} → 主文件路径
   - {{ANALYSIS_DATE}} → 当前日期
   - {{MODULE_SUMMARY}} → 模块简介
   - {{CORE_RESPONSIBILITIES}} → 核心职责(列表)
   - {{TECH_STACK}} → 技术栈(列表)
   - ... (其他占位符)
   ```

3. **移除空章节**
   - 如果某个占位符的内容为空或只有占位符,删除该章节

4. **格式化 Markdown**
   - 确保列表格式正确
   - 确保代码块有语法高亮标识
   - 确保表格格式正确

5. **保存文档**
   - 文件名: 主文件名去掉扩展名 + `.md`
   - 路径: `claudedocs/modules/{文件名}.md`

**示例**:
```
主文件: src/components/shared/map/TheMap.vue
文档路径: claudedocs/modules/TheMap.md
```

---

### 7. 更新 AGENTS.md 索引

**步骤**:

1. **读取 AGENTS.md**
   - 使用 Read 工具读取全文

2. **定位索引章节**
   ```
   查找: ## 模块文档索引
   如果不存在,在文件末尾添加该章节
   ```

3. **确定分类**
   ```javascript
   const categoryMap = {
     'complex-module': {
       'src/components/shared/': '### 共享组件模块',
       'src/components/views/': '### 业务组件模块',
       'src/components/layout/': '### 布局组件模块',
       'src/views/': '### 视图页面模块'
     },
     'component': {
       'src/components/shared/': '### 共享组件模块',
       'src/components/views/': '### 业务组件模块',
       'src/components/layout/': '### 布局组件模块'
     },
     'store': '### 状态管理模块',
     'composable': '### Composables/Hooks 模块',
     'utility': '### 工具函数模块',
     'view': '### 视图页面模块'
   }

   根据模块类型和路径确定分类
   ```

4. **生成索引条目**
   ```markdown
   - **[{模块名称}](claudedocs/modules/{文件名}.md)** - {一句话描述}
     - **功能**: {核心功能,逗号分隔}
     - **技术**: {核心技术栈}
     - **特性**: {亮点特性}(复杂模块才有)
     - **最后更新**: {YYYY-MM-DD}
   ```

5. **插入或更新条目**
   ```
   如果索引中已有该模块:
     - 更新"最后更新"日期
     - 更新功能、技术、特性描述

   如果索引中没有该模块:
     - 在对应分类下添加新条目
     - 如果分类不存在,创建新分类
   ```

6. **写回 AGENTS.md**
   - 使用 Edit 工具更新文件

---

### 8. 输出结果

**展示给用户**:

```markdown
✅ 文档生成完成!

📄 **生成的文档**: claudedocs/modules/{ModuleName}.md
📑 **更新的索引**: AGENTS.md

📊 **模块信息**:
- 类型: {模块类型}
- 复杂度: {复杂度}
- 文件数: {文件数量}
- 代码行数: {总行数}

🔗 **索引条目**:
{显示生成的索引条目}

💡 **建议**:
- 请检查生成的文档是否准确
- 如有遗漏或错误,可以手动修改文档
- 修改模块代码后,记得更新文档
```

---

## 重要注意事项

### 1. 文件路径处理
- 所有文件路径必须是相对于项目根目录的相对路径
- 使用正斜杠 `/` 而非反斜杠 `\`

### 2. 代码分析边界
- 不分析 `node_modules/`
- 不分析 `.git/`
- 不分析构建产物 `dist/`

### 3. 占位符处理
- 如果某个占位符的内容无法提取,使用 `(暂无)` 而非留空
- 如果整个章节为空,移除该章节而非留下空章节

### 4. 索引更新安全性
- 更新 AGENTS.md 前,先备份(内存中保存原始内容)
- 更新失败时,恢复原始内容并提示用户手动更新

### 5. 用户交互
- 在关键步骤(模块边界识别、类型识别)后,向用户确认
- 如果用户提供的上下文与代码分析结果矛盾,询问用户

---

## 错误处理

### 错误 1: 模块路径不存在
```
提示: ❌ 错误: 模块路径不存在: {路径}
建议: 请检查路径是否正确
```

### 错误 2: 无法识别模块类型
```
提示: ⚠️ 无法自动识别模块类型
询问: 请手动选择模块类型:
  1. Vue 组件
  2. Pinia Store
  3. Composables/Hooks
  4. 工具函数
  5. 视图页面
  6. 综合模块
```

### 错误 3: 无法读取文件
```
提示: ❌ 错误: 无法读取文件: {文件路径}
建议: 请检查文件权限或文件是否存在
```

### 错误 4: AGENTS.md 更新失败
```
提示: ❌ 错误: 无法更新 AGENTS.md 索引
建议: 请手动添加以下索引条目到 AGENTS.md:
{显示索引条目}
```

---

## 执行示例

### 示例 1: 综合模块

**用户输入**:
```
/doc-gen src/components/shared/map
```

**执行流程**:
1. 扫描 `src/components/shared/map/` 目录
2. 识别到 TheMap.vue + 8 hooks + 7 utils
3. 判断为综合模块,复杂度 = complex
4. 选择模板: `complex-module.md`
5. 深度分析代码,提取所有信息
6. 生成 `claudedocs/modules/TheMap.md`
7. 在 AGENTS.md 的"### 共享组件模块"下添加索引

### 示例 2: 简单组件

**用户输入**:
```
/doc-gen src/components/shared/forms/DynamicForm.vue
```

**执行流程**:
1. 识别主文件: DynamicForm.vue
2. 扫描相关文件: 无其他文件
3. 判断为 Vue 组件,复杂度 = simple
4. 选择模板: `component-simple.md`
5. 提取 Props、Events、核心功能
6. 生成 `claudedocs/modules/DynamicForm.md`
7. 在 AGENTS.md 的"### 共享组件模块"下添加索引

### 示例 3: Pinia Store

**用户输入**:
```
/doc-gen src/store/useTreeStationConfig.ts

上下文:
根据 docs/skill创建/2026-03-06-树形站点配置-design.md 实现
```

**执行流程**:
1. 读取设计文档,理解设计意图
2. 识别主文件: useTreeStationConfig.ts
3. 判断为 Pinia Store,复杂度 = medium
4. 选择模板: `store-detailed.md`
5. 提取 State、Getters、Actions、API 集成
6. 生成 `claudedocs/modules/useTreeStationConfig.md`
7. 在 AGENTS.md 的"### 状态管理模块"下添加索引

---

## 开始执行

现在,请等待用户输入模块路径和可选的上下文描述,然后按照上述步骤执行。
