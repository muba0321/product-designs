# SRE 运维门户 - 产品需求文档

**版本：** v1.0  
**创建时间：** 2026-03-25  
**状态：** 待开发

---

## 1. 项目概述

### 1.1 项目名称

**SRE 运维门户 (SRE Portal)**

### 1.2 项目目标

构建一个统一的运维 SRE 首页，集成待办管理、日程管理、工具导航、节点监控等功能，提供统一的运维入口。

### 1.3 用户群体

- SRE 运维工程师
- 系统管理员
- DevOps 工程师

---

## 2. 功能需求

### 2.1 核心功能

#### 2.1.1 待办管理 (Todo)

- ✅ 创建/编辑/删除待办事项
- ✅ 待办优先级（高/中/低）
- ✅ 待办状态（待处理/进行中/已完成）
- ✅ 待办分类（日常/紧急/项目）
- ✅ 截止日期提醒

#### 2.1.2 日程管理 (Schedule)

- ✅ 创建/编辑/删除日程
- ✅ 日程类型（会议/值班/检查）
- ✅ 日历视图展示
- ✅ 日程提醒

#### 2.1.3 工具导航 (Tools)

- ✅ 工具卡片展示
- ✅ 工具分类（监控/部署/日志/数据库）
- ✅ 快速跳转链接
- ✅ 工具状态（正常/维护/下线）

#### 2.1.4 节点监控 (Nodes)

- ✅ 节点列表（主机名/IP/角色）
- ✅ 资源简览（CPU/内存/磁盘）
- ✅ 健康状态（正常/警告/异常）
- ✅ 监控平台跳转链接

#### 2.1.5 服务状态 (Services)

- ✅ 服务列表（Jenkins/Prometheus/Grafana 等）
- ✅ 服务状态（运行/停止/异常）
- ✅ 端口信息
- ✅ 服务监控跳转

---

## 3. 技术架构

### 3.1 技术栈

| 层级 | 技术 | 版本 |
|------|------|------|
| **前端** | Vue 2 + Element UI | 2.6.x |
| **后端** | Flask | 2.3.x |
| **数据库** | MySQL | 8.0 |
| **容器** | Docker | latest |

### 3.2 部署架构

```
┌─────────────────────────────────────────┐
│          子节点 1 (38.246.245.39)        │
│                                          │
│  ┌──────────────┐  ┌──────────────┐     │
│  │   Nginx      │  │   Flask App  │     │
│  │   :80        │  │   :5000      │     │
│  └──────┬───────┘  └──────┬───────┘     │
│         │                 │              │
│         └────────┬────────┘              │
│                  │                       │
│         ┌────────▼────────┐              │
│         │    MySQL        │              │
│         │    :3306        │              │
│         └─────────────────┘              │
└─────────────────────────────────────────┘
```

### 3.3 目录结构

```
sre-portal/
├── backend/
│   ├── app.py              # Flask 主应用
│   ├── models.py           # 数据模型
│   ├── routes/
│   │   ├── todo.py         # 待办路由
│   │   ├── schedule.py     # 日程路由
│   │   ├── tools.py        # 工具路由
│   │   └── nodes.py        # 节点路由
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── views/
│   │   │   ├── Dashboard.vue
│   │   │   ├── Todo.vue
│   │   │   └── Schedule.vue
│   │   ├── components/
│   │   └── App.vue
│   ├── package.json
│   └── Dockerfile
├── docker-compose.yml
└── README.md
```

---

## 4. 数据库设计

### 4.1 待办表 (todos)

```sql
CREATE TABLE todos (
  id INT PRIMARY KEY AUTO_INCREMENT,
  title VARCHAR(200) NOT NULL,
  description TEXT,
  priority ENUM("high", "medium", "low") DEFAULT "medium",
  status ENUM("pending", "in_progress", "completed") DEFAULT "pending",
  category VARCHAR(50),
  due_date DATETIME,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### 4.2 日程表 (schedules)

```sql
CREATE TABLE schedules (
  id INT PRIMARY KEY AUTO_INCREMENT,
  title VARCHAR(200) NOT NULL,
  description TEXT,
  type ENUM("meeting", "duty", "check") DEFAULT "meeting",
  start_time DATETIME NOT NULL,
  end_time DATETIME,
  reminder BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 4.3 工具表 (tools)

```sql
CREATE TABLE tools (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  url VARCHAR(500),
  category VARCHAR(50),
  status ENUM("active", "maintenance", "offline") DEFAULT "active",
  icon VARCHAR(50),
  description TEXT,
  sort_order INT DEFAULT 0
);
```

### 4.4 节点表 (nodes)

```sql
CREATE TABLE nodes (
  id INT PRIMARY KEY AUTO_INCREMENT,
  hostname VARCHAR(100) NOT NULL,
  ip VARCHAR(50),
  role VARCHAR(50),
  cpu_usage DECIMAL(5,2),
  memory_usage DECIMAL(5,2),
  disk_usage DECIMAL(5,2),
  health_status ENUM("healthy", "warning", "critical") DEFAULT "healthy",
  monitor_url VARCHAR(500),
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## 5. API 设计

### 5.1 待办 API

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/todos | 获取待办列表 |
| POST | /api/todos | 创建待办 |
| PUT | /api/todos/{id} | 更新待办 |
| DELETE | /api/todos/{id} | 删除待办 |

### 5.2 日程 API

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/schedules | 获取日程列表 |
| POST | /api/schedules | 创建日程 |
| PUT | /api/schedules/{id} | 更新日程 |
| DELETE | /api/schedules/{id} | 删除日程 |

### 5.3 节点 API

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/nodes | 获取节点列表 |
| GET | /api/nodes/stats | 获取节点统计 |
| PUT | /api/nodes/{id}/health | 更新健康状态 |

---

## 6. 页面设计

### 6.1 首页布局

```
┌─────────────────────────────────────────────────────────┐
│  SRE 运维门户                              [用户] [设置]  │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │  待办事项    │  │  日程安排    │  │  工具导航    │     │
│  │  (5 待处理)  │  │  (3 今天)    │  │  (12 个)     │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│                                                          │
│  ┌─────────────────────────────────────────────────┐   │
│  │              节点健康状态                         │   │
│  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐           │   │
│  │  │master│ │node1 │ │node2 │ │bastion│          │   │
│  │  │ 🟢   │ │ 🟢   │ │ 🟡   │ │ 🟢   │           │   │
│  │  └──────┘ └──────┘ └──────┘ └──────┘           │   │
│  └─────────────────────────────────────────────────┘   │
│                                                          │
│  ┌─────────────────────────────────────────────────┐   │
│  │              服务状态                             │   │
│  │  Jenkins:🟢  Prometheus:🟢  Grafana:🟢  MySQL:🟢 │   │
│  └─────────────────────────────────────────────────┘   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 7. 部署配置

### 7.1 Docker Compose

```yaml
version: "3.8"
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: sre2026
      MYSQL_DATABASE: sre_portal
    volumes:
      - mysql_data:/var/lib/mysql
    ports:
      - "3306:3306"
  
  backend:
    build: ./backend
    ports:
      - "5000:5000"
    depends_on:
      - mysql
    environment:
      DATABASE_URL: mysql://root:sre2026@mysql:3306/sre_portal
  
  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - backend

volumes:
  mysql_data:
```

---

## 8. 验收标准

- [ ] 待办功能完整可用
- [ ] 日程功能完整可用
- [ ] 工具导航正常跳转
- [ ] 节点状态正确显示
- [ ] 服务状态正确显示
- [ ] 容器化部署成功
- [ ] 页面响应式适配

---

**文档版本：** v1.0  
**创建人：** OpenClaw Agent  
**创建时间：** 2026-03-25
