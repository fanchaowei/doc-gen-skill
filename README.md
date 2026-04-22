# 模块文档生成器 Skill

## 功能

智能分析 Vue 3 + TypeScript 项目中的代码模块,生成易于 AI 阅读的结构化文档。

## 使用方法

### 基础用法

```bash
/doc-gen <模块路径>
```

### 带上下文的用法

```bash
/doc-gen <模块路径> --context "<上下文描述>"
```

## 使用场景

### 场景 1: 有设计/实施文档

```bash
/doc-gen src/store/useTreeStationConfig.ts

上下文:
这是根据 docs/skill创建/2026-03-06-树形站点配置-design.md 实现的模块
```

### 场景 2: 手动修改代码

```bash
/doc-gen src/components/shared/tree

上下文:
我重构了树形组件,将逻辑拆分为 8 个 hooks,支持懒加载、右键菜单、拖拽排序
```

### 场景 3: AI 完成实现后

```bash
/doc-gen src/views/real-time-video

上下文:
刚完成实施,包含主视图 + 2 个 hooks + 私有子组件
```

## 支持的模块类型

1. **综合模块** - 多文件组合(组件 + hooks + utils)
2. **Vue 组件** - 单个或简单组件
3. **Pinia Store** - 状态管理
4. **Composables/Hooks** - 组合式函数
5. **工具函数** - 纯函数工具
6. **视图页面** - 路由级别组件

## 输出

- 生成文档: `claudedocs/modules/{ModuleName}.md`
- 更新索引: `CLAUDE.md` 的"模块文档索引"部分

## 文档质量

生成的文档包含(根据模块复杂度自适应):
- 📋 模块概述
- 🏗️ 架构设计
- 🔧 核心功能
- 📊 数据结构
- 🔌 API 接口
- 🎯 Props & Events
- ⚠️ 已知问题
- 🚀 性能优化
- 💡 AI 使用提示
