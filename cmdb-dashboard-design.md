# CMDB 虚拟机信息展示平台设计方案

**版本：** v1.0  
**创建时间：** 2026-03-26  
**状态：** 待复核

---

## 📋 目录

1. [项目概述](#项目概述)
2. [功能需求](#功能需求)
3. [技术架构](#技术架构)
4. [API 设计](#api-设计)
5. [UI 设计](#ui-设计)
6. [数据库设计](#数据库设计)
7. [部署方案](#部署方案)
8. [开发计划](#开发计划)

---

## 项目概述

### 背景
将 NetBox CMDB 中的虚拟机信息在 Web 页面展示，支持查看、编辑和管理。

### 目标
- 首页直观展示所有虚拟机状态
- 支持虚拟机信息的增删改查
- 美观的 UI 界面
- 高效、可扩展的后端架构

### 数据源
- **NetBox CMDB:** http://cmdb.mubai.top
- **API Token:** `1f9e13213206`

---

## 功能需求

### 1. 首页仪表盘

| 模块 | 功能 | 说明 |
|------|------|------|
| **统计卡片** | 虚拟机总数、运行中、已停止、故障 | 顶部展示关键指标 |
| **Infrastructure 分组** | 7 台基础设施服务器 | 按标签分组展示 |
| **Application 分组** | 5 个应用服务 | 按标签分组展示 |
| **快速操作** | 新增、导入、刷新 | 常用功能快捷入口 |

### 2. 虚拟机列表

| 字段 | 类型 | 说明 |
|------|------|------|
| 名称 | 文本 | 虚拟机名称 |
| 状态 | 标签 | 🟢运行中 / 🟡已停止 / 🔴故障 |
| 标签 | 标签 | Infrastructure / Application |
| CPU | 数字 | CPU 核心数 |
| 内存 | 文本 | 内存大小 (GB) |
| 磁盘 | 文本 | 磁盘大小 (GB) |
| IP 地址 | 文本 | 内网/外网 IP |
| 负责人 | 文本 | 负责人姓名 |
| 操作 | 按钮 | 编辑/删除/详情 |

### 3. 编辑功能

- ✅ 基本信息编辑（名称、状态、标签）
- ✅ 资源配置编辑（CPU、内存、磁盘）
- ✅ 网络信息编辑（IP 地址）
- ✅ 负责人分配
- ✅ 批量操作（批量修改状态、标签）

### 4. 高级功能

- 🔍 搜索过滤（按名称、标签、状态）
- 📊 排序（按名称、状态、创建时间）
- 📥 数据同步（从 NetBox 刷新数据）
- 📤 导出（CSV/Excel）

---

## 技术架构

### 架构图

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   浏览器        │────▶│   Flask 后端     │────▶│  NetBox CMDB    │
│   Vue.js        │◀────│   REST API      │◀────│  (数据源)       │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                       │
        │                       ▼
        │              ┌─────────────────┐
        │              │   SQLite/MySQL  │
        │              │   (本地缓存)    │
        │              └─────────────────┘
        │
        ▼
┌─────────────────┐
│   Nginx         │
│   (反向代理)    │
└─────────────────┘
```

### 技术栈

| 层级 | 技术 | 说明 |
|------|------|------|
| **前端** | Vue 2.x + Element UI | 基于现有 portal-vue 项目 |
| **后端** | Flask 2.x + Python 3.10 | 轻量级 REST API |
| **数据库** | SQLite (默认) / MySQL (可选) | 本地缓存 |
| **部署** | Docker + Docker Compose | 容器化部署 |
| **代理** | Nginx | 反向代理 + 静态资源 |

---

## API 设计

### 基础信息

- **Base URL:** `/api/v1`
- **认证:** JWT Token
- **格式:** JSON

### 接口列表

#### 1. 获取虚拟机列表

```http
GET /api/v1/virtual-machines
```

**请求参数:**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | int | 否 | 页码 (默认 1) |
| page_size | int | 否 | 每页数量 (默认 20) |
| tag | string | 否 | 标签过滤 (infrastructure/application) |
| status | string | 否 | 状态过滤 (active/stopped/offline) |
| search | string | 否 | 搜索关键词 |

**响应:**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "total": 12,
    "page": 1,
    "page_size": 20,
    "list": [
      {
        "id": 1,
        "name": "ser493590849885",
        "status": "active",
        "status_label": "运行中",
        "tags": ["infrastructure"],
        "vcpus": 8,
        "memory": 32768,
        "disk": 500,
        "primary_ip": "38.246.245.32",
        "owner": "沐白",
        "created": "2026-03-25T10:00:00Z",
        "updated": "2026-03-26T08:00:00Z"
      }
    ]
  }
}
```

#### 2. 获取虚拟机详情

```http
GET /api/v1/virtual-machines/:id
```

#### 3. 创建虚拟机

```http
POST /api/v1/virtual-machines
```

**请求体:**

```json
{
  "name": "new-vm-001",
  "status": "active",
  "tags": ["application"],
  "vcpus": 4,
  "memory": 8192,
  "disk": 100,
  "primary_ip": "10.0.1.100",
  "owner": "张三"
}
```

#### 4. 更新虚拟机

```http
PUT /api/v1/virtual-machines/:id
```

#### 5. 删除虚拟机

```http
DELETE /api/v1/virtual-machines/:id
```

#### 6. 同步 NetBox 数据

```http
POST /api/v1/sync/netbox
```

#### 7. 统计信息

```http
GET /api/v1/dashboard/stats
```

**响应:**

```json
{
  "code": 200,
  "data": {
    "total": 12,
    "active": 10,
    "stopped": 1,
    "offline": 1,
    "infrastructure_count": 7,
    "application_count": 5
  }
}
```

---

## UI 设计

### 首页布局

```
┌─────────────────────────────────────────────────────────────────┐
│  CMDB 虚拟机管理平台                              [刷新] [同步] [+] │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ 总数     │  │ 运行中   │  │ 已停止   │  │ 故障     │       │
│  │   12     │  │   10     │  │    1     │  │    1     │       │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │
├─────────────────────────────────────────────────────────────────┤
│  [搜索框...]  [标签▼]  [状态▼]                    [导出] [批量操作]│
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────────┐ │
│  │ Infrastructure (7)                                        │ │
│  ├───────────────────────────────────────────────────────────┤ │
│  │ [列表/卡片视图]                                           │ │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐│ │
│  │ │ VM1 │ │ VM2 │ │ VM3 │ │ VM4 │ │ VM5 │ │ VM6 │ │ VM7 ││ │
│  │ │ 🟢  │ │ 🟢  │ │ 🟢  │ │ 🟢  │ │ 🟢  │ │ 🟢  │ │ 🟢  ││ │
│  │ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘│ │
│  └───────────────────────────────────────────────────────────┘ │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │ Application (5)                                           │ │
│  ├───────────────────────────────────────────────────────────┤ │
│  │ [列表/卡片视图]                                           │ │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐                 │ │
│  │ │ App1│ │ App2│ │ App3│ │ App4│ │ App5│                 │ │
│  │ │ 🟢  │ │ 🟢  │ │ 🟡  │ │ 🟢  │ │ 🔴  │                 │ │
│  │ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘                 │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### 配色方案

| 元素 | 颜色 | 说明 |
|------|------|------|
| **运行中** | `#67C23A` (绿色) | 正常状态 |
| **已停止** | `#E6A23C` (橙色) | 已关闭 |
| **故障** | `#F56C6C` (红色) | 异常状态 |
| **主色** | `#409EFF` (蓝色) | Element UI 默认 |
| **背景** | `#F5F7FA` (浅灰) | 页面背景 |

### 卡片视图设计

```
┌─────────────────────────────┐
│  ser493590849885            │
│  ─────────────────────────  │
│  🟢 运行中                   │
│  ─────────────────────────  │
│  💾 8 核 / 32GB / 500GB     │
│  🌐 38.246.245.32           │
│  👤 沐白                    │
│  ─────────────────────────  │
│  [编辑] [详情] [更多]       │
└─────────────────────────────┘
```

### 编辑弹窗

```
┌─────────────────────────────────────┐
│  编辑虚拟机                      [×] │
├─────────────────────────────────────┤
│  名称 *    [___________________]    │
│  状态 *    [运行中 ▼]               │
│  标签 *    [Infrastructure ▼]       │
│  ─────────────────────────────────  │
│  CPU 核心  [____] 核                 │
│  内存      [____] GB                 │
│  磁盘      [____] GB                 │
│  ─────────────────────────────────  │
│  IP 地址   [___________________]    │
│  负责人    [___________________]    │
│  ─────────────────────────────────  │
│              [取消]  [保存]         │
└─────────────────────────────────────┘
```

---

## 数据库设计

### 本地缓存表 (SQLite)

```sql
-- 虚拟机表
CREATE TABLE virtual_machines (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    netbox_id INTEGER UNIQUE,          -- NetBox 中的 ID
    name VARCHAR(255) NOT NULL,        -- 虚拟机名称
    status VARCHAR(20) NOT NULL,       -- active/stopped/offline
    tags JSON,                         -- 标签数组
    vcpus INTEGER,                     -- CPU 核心数
    memory INTEGER,                    -- 内存 (MB)
    disk INTEGER,                      -- 磁盘 (GB)
    primary_ip VARCHAR(45),            -- 主 IP 地址
    owner VARCHAR(100),                -- 负责人
    description TEXT,                  -- 描述
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    synced_at DATETIME                 -- 最后同步时间
);

-- 操作日志表
CREATE TABLE operation_logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    vm_id INTEGER,
    action VARCHAR(50),                -- create/update/delete/sync
    old_value JSON,
    new_value JSON,
    operator VARCHAR(100),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (vm_id) REFERENCES virtual_machines(id)
);

-- 创建索引
CREATE INDEX idx_vm_status ON virtual_machines(status);
CREATE INDEX idx_vm_tags ON virtual_machines(tags);
CREATE INDEX idx_vm_name ON virtual_machines(name);
```

---

## 部署方案

### Docker Compose

```yaml
version: '3.8'

services:
  cmdb-web:
    build:
      context: ./cmdb-web
      dockerfile: Dockerfile
    container_name: cmdb-web
    restart: unless-stopped
    expose:
      - "3000"
    networks:
      - cmdb-net

  cmdb-api:
    build:
      context: ./cmdb-api
      dockerfile: Dockerfile
    container_name: cmdb-api
    restart: unless-stopped
    expose:
      - "5000"
    volumes:
      - ./data:/app/data          # SQLite 数据
      - ./logs:/app/logs          # 日志
    environment:
      - NETBOX_URL=http://cmdb.mubai.top
      - NETBOX_TOKEN=1f9e13213206
      - DATABASE_URL=sqlite:///data/cmdb.db
    depends_on:
      - cmdb-web
    networks:
      - cmdb-net

  nginx:
    image: nginx:alpine
    container_name: cmdb-nginx
    restart: unless-stopped
    ports:
      - "8082:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - cmdb-web
      - cmdb-api
    networks:
      - cmdb-net

networks:
  cmdb-net:
    driver: bridge
```

### 目录结构

```
/data/cmdb-dashboard/
├── cmdb-web/                 # 前端项目
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── Dockerfile
├── cmdb-api/                 # 后端项目
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── models.py
│   │   ├── routes.py
│   │   └── services/
│   ├── requirements.txt
│   └── Dockerfile
├── data/                     # 数据目录
│   └── cmdb.db
├── logs/                     # 日志目录
├── nginx.conf                # Nginx 配置
└── docker-compose.yml        # Docker Compose 配置
```

---

## 开发计划

### 阶段 1：后端开发 (2 天)

| 任务 | 工时 | 说明 |
|------|------|------|
| 项目初始化 | 2h | Flask 项目结构、依赖安装 |
| 数据模型 | 2h | SQLAlchemy 模型定义 |
| NetBox 同步服务 | 4h | API 对接、数据同步 |
| CRUD API | 4h | 增删改查接口 |
| 认证授权 | 2h | JWT Token 认证 |
| 单元测试 | 2h | API 测试 |

### 阶段 2：前端开发 (2 天)

| 任务 | 工时 | 说明 |
|------|------|------|
| 项目初始化 | 2h | 基于 portal-vue 扩展 |
| 首页布局 | 4h | 统计卡片、分组展示 |
| 列表组件 | 4h | 表格/卡片视图切换 |
| 编辑弹窗 | 3h | 表单验证、提交 |
| API 对接 | 3h | Axios 封装、状态管理 |
| UI 优化 | 2h | 响应式、动画效果 |

### 阶段 3：联调测试 (1 天)

| 任务 | 工时 | 说明 |
|------|------|------|
| 接口联调 | 3h | 前后端对接 |
| 功能测试 | 2h | 完整流程测试 |
| 性能优化 | 2h | 加载速度、缓存 |
| Bug 修复 | 1h | 问题修复 |

### 阶段 4：部署上线 (0.5 天)

| 任务 | 工时 | 说明 |
|------|------|------|
| Docker 构建 | 1h | 镜像构建 |
| 部署配置 | 1h | 环境配置 |
| 验收测试 | 2h | 最终验收 |

**总计：** 5.5 天

---

## 待确认事项

### 需与后端开发助手确认

1. [ ] Flask 版本选择 (2.x vs 3.x)
2. [ ] 数据库选择 (SQLite vs MySQL)
3. [ ] 认证方式 (JWT vs Session)
4. [ ] 日志方案 (文件 vs ELK)
5. [ ] API 文档工具 (Swagger vs Redoc)

### 需与前端开发助手确认

1. [ ] UI 框架 (Element UI vs Element Plus)
2. [ ] 状态管理 (Vuex vs Pinia)
3. [ ] 构建工具 (Webpack vs Vite)
4. [ ] 图表库 (ECharts vs Chart.js)
5. [ ] 是否复用现有 portal-vue 项目

---

## 附录

### A. NetBox API 参考

- **文档:** https://docs.netbox.dev/en/stable/api/
- **虚拟机端点:** `/api/virtualization/virtual-machines/`
- **标签端点:** `/api/extras/tags/`

### B. 参考项目

- vue-element-admin: https://github.com/PanJiaChen/vue-element-admin
- Flask 官方文档: https://flask.palletsprojects.com/

---

**文档结束**

**下一步：** 请 @后端开发助手 和 @前端开发助手 复核此方案，确认技术选型和接口设计。
