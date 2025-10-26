<!--
Version: 1.0.0 (Initial ratification)
Sync Impact Report:
- Version change: none → 1.0.0
- Modified principles: N/A (initial creation)
- Added sections: All (initial creation)
- Removed sections: N/A
- Templates requiring updates:
  ⚠ plan-template.md: To be created
  ⚠ spec-template.md: To be created
  ⚠ tasks-template.md: To be created
  ⚠ commands/*.md: To be created
- Follow-up TODOs: Create all dependent template files
-->

# 项目宪章 (Project Constitution)

**项目名称**: 极简项目管理工具 (Minimalist Project Management Tool)

**宪章版本**: 1.0.0

**批准日期**: 2025-10-26

**最后修订日期**: 2025-10-26

---

## 宪章目的 (Purpose)

本宪章为"极简项目管理工具"项目确立了核心开发原则、架构约束和质量标准。所有设计决策、代码贡献和技术选型必须符合本宪章规定的原则。本文档旨在确保项目在整个生命周期内保持技术一致性、代码质量和架构清晰度。

---

## 核心原则 (Core Principles)

### 原则 1: 极简主义设计 (Minimalist Design)

**原则声明**:
所有用户界面、功能特性和代码实现必须遵循极简主义设计哲学。复杂性是敌人，简洁性是目标。

**具体要求**:
- 用户界面必须遵循 Apple 设计风格：清晰的字体层级、充足的留白、柔和的圆角、有限的色板（不超过 3-4 种主色）
- 每个功能模块必须有明确的单一职责，避免功能堆砌
- UI 组件必须使用 shadcn/ui 库，保持设计系统的一致性
- 移除所有非必要的动画、装饰和视觉元素
- 交互流程必须在 3 步以内完成核心任务（创建任务、更新状态等）

**设计理由**:
目标用户是 20-30 人的小型团队，需要工具快速上手、零学习成本。市面上工具（Jira、ClickUp）因功能臃肿导致使用障碍，极简设计能最大化团队生产力。

---

### 原则 2: 安全优先 (Security First)

**原则声明**:
系统必须在数据库层、API 层和 UI 层实现多层安全防护。权限控制不是可选项，而是架构的核心。

**具体要求**:
- 所有数据库表必须配置 Supabase Row Level Security (RLS) 策略
- 禁止在客户端代码中绕过 RLS 直接访问数据库
- 所有 Server Actions 和 API Routes 必须验证用户角色和权限
- 敏感操作（用户管理、项目删除、角色变更）必须有二次确认机制
- 富文本编辑器（Wiki）必须对用户输入进行 XSS 防护和 HTML 清理
- 禁止在代码中硬编码凭证、API 密钥或敏感配置
- 用户会话必须使用 Supabase Auth 管理，禁止自定义会话机制

**设计理由**:
项目管理系统包含敏感的商业信息（成本、项目进度、内部文档）。基于角色的访问控制（RBAC）是核心需求，安全漏洞可能导致数据泄露和合规风险。

---

### 原则 3: 强制质量保证 (Mandatory Quality Assurance)

**原则声明**:
代码质量不是建议，而是交付标准。每个功能模块必须通过测试和代码审查才能视为完成。

**具体要求**:
- 每个功能模块开发完成后，必须执行以下流程（按顺序）：
  1. **单元测试**: 使用专门的测试工具或子代理进行单元测试
     - 测试所有新增的函数和组件
     - 验证边界条件、错误处理和异常情况
     - 确保测试覆盖率达到合理水平（关键业务逻辑 ≥ 80%）
  2. **代码审查**: 测试通过后，使用 `feature-dev:code-reviewer` 子代理进行代码审查
     - 检查代码质量、潜在安全漏洞和最佳实践遵守情况
     - 验证是否符合项目架构约定和设计模式
     - 审查权限控制实现是否正确（特别是 RLS 策略和角色检查）
  3. **问题修复**: 如果发现问题，必须立即修复并重新测试和审查
  4. **完成确认**: 只有测试和审查都通过后，该功能模块才被视为完成
- 禁止跳过任何质量保证步骤，即使时间紧迫

**设计理由**:
小型团队无法承受因代码质量问题导致的系统故障或安全事故。强制质量保证流程能在开发早期发现问题，降低维护成本。

---

### 原则 4: 类型安全与可维护性 (Type Safety & Maintainability)

**原则声明**:
代码必须是类型安全的、可读的、可维护的。使用 TypeScript 的严格模式，避免 `any` 类型的滥用。

**具体要求**:
- 所有 TypeScript 代码必须启用 `strict` 模式（`tsconfig.json` 中配置）
- 禁止使用 `any` 类型，除非有充分理由并添加注释说明
- 数据库类型定义必须使用 Supabase CLI 自动生成（`supabase gen types typescript`）
- 所有组件的 Props 必须定义接口（Interface）或类型别名（Type Alias）
- Server Actions 的输入输出必须使用 Zod 或类似库进行运行时验证
- 函数和组件必须有清晰的命名，遵循语义化命名约定（如 `createTask`, `TaskCard`）
- 复杂逻辑必须添加注释说明意图，特别是权限检查和数据转换逻辑

**设计理由**:
TypeScript 的类型系统能在编译时捕获大量错误，减少运行时问题。可维护性对小型团队尤为重要，团队成员可能同时负责多个模块。

---

### 原则 5: 性能优先与渐进增强 (Performance First & Progressive Enhancement)

**原则声明**:
系统必须快速响应、低延迟。优先使用服务端渲染（SSR）和静态生成（SSG），谨慎使用客户端渲染（CSR）。

**具体要求**:
- 默认使用 Next.js Server Components 进行数据获取和页面渲染
- 仅在必要时使用 Client Components（表单交互、拖拽、富文本编辑器等）
- 使用 `loading.tsx` 和 `Suspense` 边界提供加载状态，避免白屏
- 图片必须使用 Next.js `<Image>` 组件进行优化（懒加载、尺寸优化）
- 大列表（任务列表、项目列表）必须实现虚拟滚动或分页
- 数据库查询必须使用索引，避免全表扫描
- 禁止在组件渲染循环中发起网络请求
- 使用 Vercel 的 Edge Functions 和 CDN 加速静态资源

**设计理由**:
用户期望工具快速响应（如拖拽任务卡片的即时反馈）。性能问题会直接影响用户体验和工具的采用率。

---

### 原则 6: 数据一致性与可靠性 (Data Consistency & Reliability)

**原则声明**:
系统必须保证数据的完整性和一致性。关键操作必须是原子的、可追溯的。

**具体要求**:
- 数据库表必须定义适当的外键约束（Foreign Key Constraints）
- 级联删除（CASCADE）必须谨慎使用，关键数据（如项目、任务）删除前必须有确认机制
- 用户操作（创建、编辑、删除）必须有审计日志（可选：V1 不强制，V2 实现）
- 并发写入（如多人同时编辑任务）必须使用乐观锁或数据库事务
- 所有表必须有 `created_at` 和 `updated_at` 时间戳字段
- 数据迁移脚本（Supabase Migrations）必须可逆，提供 `up` 和 `down` 路径

**设计理由**:
项目管理数据（任务、成本、文档）是业务关键信息，数据丢失或不一致会导致严重后果。可靠性是企业工具的基础。

---

## 技术栈约束 (Technology Stack Constraints)

以下技术栈已确定且不可变更（除非宪章修订）：

- **前端框架**: Next.js (App Router, React 18+)
- **后端/数据库**: Supabase (PostgreSQL 15+, Auth, RLS)
- **UI 库**: Tailwind CSS 3+ + shadcn/ui
- **部署平台**: Vercel
- **语言**: TypeScript 5+（禁止使用 JavaScript）

**允许的额外依赖**:
- 拖拽库: dnd-kit（推荐）或 react-beautiful-dnd
- 富文本编辑器: Tiptap 或 Lexical（需符合安全原则）
- 表单验证: Zod + React Hook Form
- 日期处理: date-fns（禁止使用 moment.js）

**禁止的技术**:
- Redux、MobX 等复杂状态管理库（优先使用 React Server Components 和 Server Actions）
- jQuery 或其他传统 DOM 操作库
- 自建认证系统（必须使用 Supabase Auth）

---

## 架构约束 (Architecture Constraints)

### 文件组织 (File Organization)
```
/app                  # Next.js App Router 页面和布局
  /(auth)            # 认证相关页面（登录、注册）
  /(dashboard)       # 主应用页面（仪表盘、项目详情）
  /api               # API Routes（仅用于 Webhooks 等特殊场景）
/components          # 可复用 UI 组件
  /ui                # shadcn/ui 组件
  /features          # 功能特定组件（TaskCard, ProjectList）
/lib                 # 工具函数、常量、类型定义
  /supabase          # Supabase 客户端配置
  /actions           # Server Actions
  /types             # TypeScript 类型定义
/hooks               # 自定义 React Hooks
/public              # 静态资源
/supabase            # Supabase 配置和迁移脚本
  /migrations        # 数据库迁移文件
```

### 数据流 (Data Flow)
- **读取数据**: Server Components → Supabase Client → RLS → Database
- **写入数据**: Client Components → Server Actions → RLS → Database
- **实时更新**: Supabase Realtime Subscriptions（可选，V2）

---

## 治理与修订 (Governance & Amendment)

### 修订流程
1. **提议**: 任何团队成员可提议宪章修订（通过 Issue 或讨论）
2. **评估**: 技术负责人评估修订的必要性和影响范围
3. **审批**: 修订必须获得项目负责人或管理员的批准
4. **实施**: 修订后更新版本号，同步更新所有依赖模板

### 版本控制规则
- **MAJOR 版本升级**: 移除或重新定义核心原则，不兼容现有架构
- **MINOR 版本升级**: 新增原则、技术栈约束或架构要求
- **PATCH 版本升级**: 澄清措辞、修正错误、补充示例

### 合规性审查
- 每个 Sprint/迭代结束时，技术负责人应检查代码是否符合宪章
- 发现违反宪章的代码必须在下一个迭代中修复
- 重复违反宪章可能需要修订宪章或调整开发流程

---

## 签署与承诺 (Signature & Commitment)

本宪章代表项目团队对技术卓越、代码质量和用户体验的共同承诺。所有贡献者在提交代码前应熟读本宪章并遵守其规定。

**批准人**: 项目管理员
**生效日期**: 2025-10-26
