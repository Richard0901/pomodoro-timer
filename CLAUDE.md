# 番茄时钟 — 项目文档

## 项目概述

番茄时钟（Pomodoro Timer）是一款基于 Tauri v2 的桌面应用，提供番茄工作法计时、任务管理和数据统计功能。

## 技术栈

### 前端（TypeScript）
- **框架**: React 19 + TypeScript 6
- **构建工具**: Vite 8
- **样式**: Tailwind CSS v4（含 `@tailwindcss/vite` 插件）
- **状态管理**: Zustand
- **图表**: Recharts
- **包管理**: npm

### 后端（Rust）
- **桌面框架**: Tauri 2.x
- **数据库**: SQLite（通过 sqlx + tauri-plugin-sql）
- **额外插件**:
  - `tauri-plugin-notification` — 系统通知
  - `tauri-plugin-window-state` — 窗口状态持久化
  - `tauri-plugin-log` — 开发日志
- **时间处理**: chrono
- **UUID**: uuid v4
- **序列化**: serde / serde_json

## 项目目录结构

```
pomodoro/
├── src/                          # React 前端代码
│   ├── App.tsx                   # 根组件
│   ├── main.tsx                  # 入口文件
│   ├── index.css                 # 全局样式 + Tailwind 主题配置
│   ├── components/
│   │   ├── timer/                # 计时器组件
│   │   ├── tasks/                # 任务列表组件
│   │   ├── stats/                # 统计图表组件
│   │   ├── settings/             # 设置页面组件
│   │   └── ui/                   # 通用 UI 组件
│   ├── hooks/                    # 自定义 Hooks
│   ├── lib/                      # 工具函数
│   ├── store/                    # Zustand 状态仓库
│   └── assets/                   # 静态资源
├── src-tauri/                    # Tauri/Rust 后端代码
│   ├── src/
│   │   ├── main.rs               # Rust 入口
│   │   ├── lib.rs                # Tauri 应用初始化、插件注册、命令注册
│   │   ├── db.rs                 # SQLite 数据库初始化 & 迁移
│   │   ├── state.rs              # 全局状态（DbPool）
│   │   └── commands/
│   │       ├── mod.rs            # 命令模块声明
│   │       ├── tasks.rs          # 任务 CRUD 命令
│   │       ├── sessions.rs       # 番茄钟会话命令
│   │       └── stats.rs          # 统计查询命令
│   └── Cargo.toml                # Rust 依赖
├── index.html                    # HTML 入口
├── vite.config.ts                # Vite 配置
├── tsconfig.json                 # TypeScript 根配置
├── tsconfig.app.json             # 前端 TypeScript 配置
├── tsconfig.node.json            # Node 端 TypeScript 配置
├── eslint.config.js              # ESLint 配置
└── package.json                  # 前端依赖 & 脚本
```

## 数据库

### 数据库文件
应用数据目录下的 `pomodoro.db`（SQLite），通过 `tauri::path::app_data_dir()` 定位。

### 表结构

#### `tasks` — 任务表
| 列 | 类型 | 说明 |
|---|---|---|
| id | TEXT PK | UUID v4 |
| title | TEXT NOT NULL | 任务标题 |
| description | TEXT | 任务描述 |
| estimated_pomodoros | INTEGER | 预估番茄数 |
| completed_pomodoros | INTEGER | 已完成番茄数 |
| is_completed | INTEGER(0/1) | 是否完成 |
| created_at | TEXT | 创建时间 (RFC3339) |
| completed_at | TEXT | 完成时间 (RFC3339) |

#### `sessions` — 会话表
| 列 | 类型 | 说明 |
|---|---|---|
| id | TEXT PK | UUID v4 |
| task_id | TEXT FK | 关联任务 ID |
| session_type | TEXT | `work` / `short_break` / `long_break` |
| duration_seconds | INTEGER | 计划时长（秒） |
| started_at | TEXT | 开始时间 (RFC3339) |
| completed_at | TEXT | 完成时间 (RFC3339) |
| was_completed | INTEGER(0/1) | 是否自然完成 |

#### `settings` — 设置表
| 列 | 类型 | 说明 |
|---|---|---|
| key | TEXT PK | 设置键名 |
| value | TEXT | 设置值 |

### 迁移
迁移在 `db.rs` 的 `MIGRATIONS` 常量中定义，应用启动时自动执行。

## Tauri 命令注册

在 `lib.rs` 中通过 `invoke_handler` 注册，所有 Tauri 命令通过 `#[tauri::command]` 定义。

### 任务命令（`commands::tasks`）
- `get_tasks` — 获取所有任务（按创建时间倒序）
- `create_task` — 创建任务
- `update_task` — 更新任务（支持部分更新）
- `delete_task` — 删除任务（级联删除关联会话）
- `increment_task_pomodoros` — 任务番茄数 +1

### 会话命令（`commands::sessions`）
- `create_session` — 创建会话
- `complete_session` — 将会话标记为已完成
- `get_sessions` — 查询会话列表（支持分页和类型筛选）
- `get_incomplete_session` — 获取未完成的会话（用于恢复）

### 统计命令（`commands::stats`）
- `get_stats_summary` — 获取今日/本周/总计/连续天数
- `get_daily_summary` — 每日统计（近 90 天）
- `get_task_breakdown` — 各任务番茄分布
- `get_session_history` — 历史记录（同 `get_daily_summary`）

## 窗口配置

- 默认尺寸: 420×700
- 最小尺寸: 360×600
- 支持缩放
- 启动时居中
- 含窗口标题栏装饰

## 开发命令

```bash
# 启动开发模式（前端 dev server + Tauri 窗口）
npm run tauri dev

# 构建前端代码
npm run build

# 构建桌面应用
npm run tauri build

# ESLint 检查
npm run lint

# 仅启动 Vite 前端预览
npm run dev

# 预览构建产物
npm run preview
```

## 样式规范

使用 Tailwind CSS v4 的自定义主题，通过 `@theme` 指令定义：

| Token | 值 | 用途 |
|---|---|---|
| `surface-0` | `oklch(18% 0.01 15)` | 最深背景（主背景） |
| `surface-1` | `oklch(22% 0.01 15)` | 中层背景 |
| `surface-2` | `oklch(28% 0.01 15)` | 最浅背景 |
| `text-primary` | `oklch(92% 0.01 90)` | 主文字 |
| `text-secondary` | `oklch(65% 0.01 90)` | 次要文字 |
| `tomato` | `oklch(68% 0.21 25)` | 番茄红（主色调） |
| `tomato-bright` | `oklch(72% 0.23 25)` | 亮番茄红（悬停/激活） |
| `amber` | `oklch(75% 0.14 80)` | 琥珀色（休息/提示） |
| `teal` | `oklch(68% 0.09 195)` | 青色（完成/成功） |
| `rose` | `oklch(60% 0.17 10)` | 玫瑰红 |

字体: 系统原生 `-apple-system, BlinkMacSystemFont, SF Pro Text, Helvetica Neue`
无用户选择 (`user-select: none`)

## 代码规范

- TypeScript: `strict` 模式，`verbatimModuleSyntax` 启用
- React: JSX 使用 `react-jsx` runtime
- 导入路径: 使用相对路径（无别名配置）
- ESLint: typescript-eslint + react-hooks + react-refresh 规则
- Rust: 2021 edition，error 统一通过 `e.to_string()` 返回 `String`

## 注意事项

- 前端组件目录目前为空，App.tsx 为占位组件（显示"番茄时钟加载中..."）
- 后端 Rust 后端已完整实现（数据库、命令、统计逻辑）
- 应用使用 `app.preventClose()` 进行窗口状态管理（通过 window-state 插件）
