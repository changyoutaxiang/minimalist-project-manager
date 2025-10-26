# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 重要提示：沟通语言

**始终使用中文与用户沟通。** 所有回复、说明、注释和文档都应使用中文。

## Project Overview

This is a minimalist project management tool designed for internal teams of 20-30 people. The application follows Apple's design philosophy with a focus on simplicity, clarity, and efficiency.

## Technology Stack

- **Frontend**: Next.js (App Router)
- **Backend/Database**: Supabase (PostgreSQL, Auth, Row Level Security)
- **UI Library**: Tailwind CSS + shadcn/ui
- **Deployment**: Vercel

## User Roles & Access Control

The system implements Role-Based Access Control (RBAC) with four user types:

1. **管理员 (Admin)**: System administrator with full access
2. **观察员 (Observer)**: High-level view-only access to all projects
3. **项目负责人 (Project Lead)**: Project managers who can manage assigned projects
4. **项目成员 (Project Member)**: Task executors with limited permissions

### Permission Matrix

| Feature | Admin | Observer | Project Lead | Member |
|---------|-------|----------|--------------|--------|
| User Management | Full CRUD | View | - | - |
| All Projects | Full CRUD | View | Create | - |
| Assigned Projects | - | - | Edit/Delete | View |
| Tasks | - | - | Full CRUD & Assign | View/Update Status |
| Cost Management | - | - | Edit | View |
| Documents/Wiki | - | - | Full CRUD | View |

## Core Features

### F-AUTH-01: Authentication
- Email/password login
- Session management
- Admin-only user invitations (V1)

### F-DASH-01: Role-based Dashboard
- **Admin/Observer**: View all projects with progress and leads
- **Lead/Member**: "My Tasks" (cross-project) + "My Projects" lists

### F-TASK-01: Kanban Board
- Three columns: 待办 (To Do), 进行中 (In Progress), 已完成 (Done)
- Drag-and-drop task cards with: title, assignee avatar/name, due date
- Task creation form for Leads/Admins

### F-COST-01: Cost Management
- Per-project budget and spent tracking
- Lead/Admin edit, all members view

### F-DOCS-01 & F-DOCS-02: Document Library
- **Module A**: External links (Feishu, Figma, etc.)
- **Module B**: Built-in Wiki with WYSIWYG editor

### F-PPL-01: User Management (Admin only)
- Invite users via email
- Manage roles and permissions
- User list with CRUD operations

## Architecture Considerations

### Database Schema (Supabase)
When implementing the database:
- Use Supabase Row Level Security (RLS) policies to enforce role-based permissions
- Main tables: users, projects, tasks, project_members, documents, wiki_pages, costs
- Foreign key relationships: project_members links users to projects with roles

### Next.js Structure
- Use App Router with server components for initial data fetching
- Implement middleware for auth checks and role-based route protection
- Client components for interactive features (drag-and-drop, forms, editors)

### State Management
- Use React Server Components where possible to reduce client-side state
- Client state for UI interactions (modals, forms)
- Supabase real-time subscriptions for collaborative features (optional in V1)

## UI/UX Guidelines

- **UI-01**: Follow Apple design principles - minimal, modern, intuitive
- **UI-02**: Clear typography, generous whitespace, soft corners, limited color palette
- **UI-03**: Responsive design prioritizing desktop experience, basic mobile support

Use shadcn/ui components consistently:
- Buttons, Cards, Dialogs for modals
- Form components with proper validation
- Data tables for user lists
- Drag-and-drop libraries compatible with Tailwind (e.g., dnd-kit)

## Development Workflow

When implementing features:
1. Start with database schema and RLS policies in Supabase
2. Create server actions for data mutations
3. Build UI components using shadcn/ui
4. Implement role-based access checks at both API and UI levels
5. Test permission boundaries for each role

### 质量保证流程

**每完成一个功能模块的开发后，必须执行以下流程：**

1. **单元测试**：使用 Task tool 调用专门的测试子代理进行单元测试
   - 测试所有新增的函数和组件
   - 验证边界条件和错误处理
   - 确保测试覆盖率达标

2. **代码审查**：测试通过后，使用 Task tool 调用 `feature-dev:code-reviewer` 子代理进行代码审查
   - 检查代码质量、安全漏洞和最佳实践
   - 验证是否符合项目约定和设计模式
   - 审查权限控制实现是否正确

3. **问题修复**：如果代码审查发现问题，必须立即修复并重新进行测试和审查，直到通过为止

4. **完成确认**：只有在测试和审查都通过后，才能认为该功能模块开发完成

## Security Notes

- Never bypass RLS policies - always test with different role contexts
- Validate user permissions on both client and server
- Use Supabase Auth for session management
- Implement CSRF protection for form submissions
- Sanitize user inputs, especially in Wiki rich text editor
