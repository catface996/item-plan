# 用户故事映射 - 商品审核模块

## 概述

本文档将用户故事映射到两个工作单元（前端和后端）。

---

## 故事映射总览

| 故事ID | 故事名称 | 优先级 | 后端 | 前端 |
|--------|----------|--------|------|------|
| US-001 | 用户登录 | Must | ✅ | ✅ |
| US-002 | 用户登出 | Must | ✅ | ✅ |
| US-003 | 提交新商品 | Must | ✅ | ✅ |
| US-004 | 查看我的商品列表 | Must | ✅ | ✅ |
| US-005 | 查看商品审核状态 | Must | ✅ | ✅ |
| US-006 | 修改被退回商品 | Must | ✅ | ✅ |
| US-007 | 查看待初审商品列表 | Must | ✅ | ✅ |
| US-008 | 执行初审操作 | Must | ✅ | ✅ |
| US-009 | 填写审核意见 | Should | ✅ | ✅ |
| US-010 | 查看待复审商品列表 | Must | ✅ | ✅ |
| US-011 | 执行复审操作 | Must | ✅ | ✅ |
| US-012 | 查看审核历史 | Should | ✅ | ✅ |
| US-013 | 审核统计报表 | Could | ✅ | ✅ |
| US-014 | 操作日志记录 | Should | ✅ | ❌ |

---

## Unit-001: 后端服务

| 模块 | 故事 | API端点 |
|------|------|---------|
| user | US-001, US-002 | /api/auth/* |
| product | US-003-006 | /api/products/* |
| audit | US-007-013 | /api/audits/* |
| common | US-014 | AOP切面 |

## Unit-002: 前端应用

| 模块 | 故事 | 页面组件 |
|------|------|----------|
| user | US-001, US-002 | Login.tsx |
| product | US-003-006 | ProductList, ProductForm, ProductDetail |
| audit | US-007-013 | PendingList, ReviewList, AuditPanel, AuditHistory, AuditStats |

---

## 开发顺序

1. 后端核心API开发
2. 前端基于Mock数据开发UI
3. 前后端联调
