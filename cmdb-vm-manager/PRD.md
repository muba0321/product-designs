# CMDB 虚拟机管理系统 - 产品需求文档

**版本：** v1.0  
**创建时间：** 2026-03-26  
**状态：** 开发中  
**优先级：** P0

---

## 1. 项目概述

### 1.1 项目名称

**CMDB 虚拟机管理系统 (CMDB VM Manager)**

### 1.2 项目背景

当前运维环境中有多台虚拟机分布在不同的宿主机上，缺乏统一的管理和展示平台。需要将数据库中的虚拟机信息同步到 CMDB 系统，并提供可视化的管理界面。

### 1.3 项目目标

- ✅ 从数据库同步虚拟机信息到 CMDB
- ✅ 提供虚拟机信息展示页面（首页）
- ✅ 支持页面修改虚拟机信息
- ✅ 前后端分离架构，高效可扩展

### 1.4 用户群体

- SRE 运维工程师
- 系统管理员
- 运维开发人员

---

## 2. 功能需求

### 2.1 核心功能

#### 2.1.1 虚拟机信息展示（首页）

- ✅ 虚拟机列表展示（表格形式）
- ✅ 关键信息卡片（总数/运行中/已停止/异常）
- ✅ 搜索和过滤功能
- ✅ 分页展示

#### 2.1.2 虚拟机信息管理

- ✅ 编辑虚拟机信息（名称/IP/CPU/内存/磁盘/状态等）
- ✅ 批量操作支持
- ✅ 操作日志记录

#### 2.1.3 数据同步

- ✅ 从源数据库同步虚拟机信息
- ✅ 手动触发同步
- ✅ 同步状态展示

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
│  │   :80        │  │   :5001      │     │
│  └──────┬───────┘  └──────┬───────┘     │
│         │                 │              │
│         └────────┬────────┘              │
│                  │                       │
│         ┌────────▼────────┐              │
│         │    MySQL        │              │
│         │    :3306        │              │
│         └─────────────────┘              │
│                  │                       │
│         ┌────────▼────────┐              │
│         │   源数据库       │              │
│         │   (同步源)      │              │
│         └─────────────────┘              │
└─────────────────────────────────────────┘
```

### 3.3 目录结构

```
cmdb-vm-manager/
├── backend/
│   ├── app.py              # Flask 主应用
│   ├── models.py           # 数据模型
│   ├── routes/
│   │   ├── vm.py           # 虚拟机路由
│   │   └── sync.py         # 同步路由
│   ├── services/
│   │   └── db_sync.py      # 数据库同步服务
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── views/
│   │   │   ├── Dashboard.vue    # 首页
│   │   │   └── VMList.vue       # 虚拟机列表
│   │   ├── components/
│   │   └── App.vue
│   ├── package.json
│   └── Dockerfile
├── docker-compose.yml
└── README.md
```

---

## 4. 数据库设计

### 4.1 虚拟机表 (virtual_machines)

```sql
CREATE TABLE virtual_machines (
  id INT PRIMARY KEY AUTO_INCREMENT,
  vm_name VARCHAR(100) NOT NULL COMMENT '虚拟机名称',
  vm_id VARCHAR(50) UNIQUE COMMENT '虚拟机唯一标识',
  ip_address VARCHAR(50) COMMENT 'IP 地址',
  mac_address VARCHAR(50) COMMENT 'MAC 地址',
  host_name VARCHAR(100) COMMENT '宿主机名称',
  cpu_cores INT DEFAULT 0 COMMENT 'CPU 核心数',
  memory_gb DECIMAL(10,2) DEFAULT 0 COMMENT '内存 (GB)',
  disk_gb DECIMAL(10,2) DEFAULT 0 COMMENT '磁盘 (GB)',
  os_type VARCHAR(50) COMMENT '操作系统类型',
  os_version VARCHAR(50) COMMENT '操作系统版本',
  status ENUM('running', 'stopped', 'suspended', 'error') DEFAULT 'stopped' COMMENT '运行状态',
  environment ENUM('production', 'staging', 'development', 'test') DEFAULT 'development' COMMENT '环境',
  owner VARCHAR(100) COMMENT '负责人',
  department VARCHAR(100) COMMENT '所属部门',
  created_date DATETIME COMMENT '创建日期',
  last_sync_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后同步时间',
  remarks TEXT COMMENT '备注',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_status (status),
  INDEX idx_host (host_name),
  INDEX idx_environment (environment)
);
```

### 4.2 同步日志表 (sync_logs)

```sql
CREATE TABLE sync_logs (
  id INT PRIMARY KEY AUTO_INCREMENT,
  sync_type VARCHAR(50) NOT NULL COMMENT '同步类型',
  status ENUM('success', 'partial', 'failed') DEFAULT 'success' COMMENT '同步状态',
  total_count INT DEFAULT 0 COMMENT '总记录数',
  success_count INT DEFAULT 0 COMMENT '成功数',
  failed_count INT DEFAULT 0 COMMENT '失败数',
  error_message TEXT COMMENT '错误信息',
  started_at DATETIME NOT NULL COMMENT '开始时间',
  completed_at DATETIME COMMENT '完成时间',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 5. API 设计

### 5.1 虚拟机 API

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/vms | 获取虚拟机列表（支持分页、搜索、过滤） |
| GET | /api/vms/stats | 获取虚拟机统计信息 |
| GET | /api/vms/{id} | 获取单个虚拟机详情 |
| POST | /api/vms | 创建虚拟机记录 |
| PUT | /api/vms/{id} | 更新虚拟机信息 |
| DELETE | /api/vms/{id} | 删除虚拟机记录 |
| POST | /api/vms/batch-update | 批量更新虚拟机 |

### 5.2 同步 API

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | /api/sync/trigger | 触发数据同步 |
| GET | /api/sync/status | 获取同步状态 |
| GET | /api/sync/logs | 获取同步日志 |

---

## 6. 页面设计

### 6.1 首页布局（Dashboard）

```
┌─────────────────────────────────────────────────────────┐
│  CMDB 虚拟机管理                          [同步] [设置]  │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │ 虚拟机总数│ │ 运行中    │ │ 已停止    │ │ 异常     │  │
│  │   128    │ │   96     │ │   28     │ │    4     │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘  │
│                                                          │
│  ┌─────────────────────────────────────────────────┐   │
│  │  搜索：[________________] 环境：[全部▼] 状态：[全部▼] │
│  └─────────────────────────────────────────────────┘   │
│                                                          │
│  ┌─────────────────────────────────────────────────┐   │
│  │ 虚拟机列表                                       │   │
│  │ ┌─────┬──────────┬──────────┬──────┬──────┬───┐│   │
│  │ │选择 │ 名称      │ IP 地址   │ 状态  │ 环境  │...││   │
│  │ ├─────┼──────────┼──────────┼──────┼──────┼───┤│   │
│  │ │ ☐   │ vm-web-01│ 10.0.1.1 │ 🟢运行│ 生产  │...││   │
│  │ │ ☐   │ vm-db-01 │ 10.0.1.2 │ 🟢运行│ 生产  │...││   │
│  │ │ ☐   │ vm-test-1│ 10.0.2.1 │ 🔴停止│ 测试  │...││   │
│  │ └─────┴──────────┴──────────┴──────┴──────┴───┘│   │
│  │                                    [1] [2] [3]  │   │
│  └─────────────────────────────────────────────────┘   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 6.2 编辑虚拟机弹窗

```
┌─────────────────────────────────────────┐
│  编辑虚拟机：vm-web-01              [×]  │
├─────────────────────────────────────────┤
│                                          │
│  名称：    [vm-web-01____________]       │
│  IP 地址：  [10.0.1.1____________]        │
│  宿主机：  [host-01______________]       │
│  CPU:      [4____] 核心                  │
│  内存：    [8____] GB                    │
│  磁盘：    [100___] GB                   │
│  状态：    [运行中 ▼]                    │
│  环境：    [生产环境 ▼]                  │
│  负责人：  [____________________]        │
│  部门：    [____________________]        │
│  备注：    [____________________]        │
│            [____________________]        │
│                                          │
│              [取消]  [保存]              │
└─────────────────────────────────────────┘
```

---

## 7. 实施计划

### 阶段一：后端基础架构（当前执行）
- [ ] 创建项目目录结构
- [ ] 设计数据库表结构
- [ ] 实现 Flask 后端 API
- [ ] 实现数据库同步服务
- [ ] 编写单元测试

### 阶段二：前端界面开发
- [ ] 搭建 Vue 前端项目
- [ ] 实现首页 Dashboard
- [ ] 实现虚拟机列表页
- [ ] 实现编辑功能
- [ ] 实现搜索过滤

### 阶段三：集成与部署
- [ ] 前后端联调
- [ ] Docker 容器化
- [ ] 部署到子节点
- [ ] 性能测试与优化

---

## 8. 验收标准

- [ ] 后端 API 完整可用（Postman 测试通过）
- [ ] 前端页面美观易用
- [ ] 数据同步功能正常
- [ ] 支持页面修改虚拟机信息
- [ ] 容器化部署成功
- [ ] 响应式适配

---

## 9. 风险与注意事项

### 9.1 数据同步风险
- 源数据库结构可能变化，需保持同步逻辑灵活
- 同步过程需保证数据一致性，避免脏数据

### 9.2 性能考虑
- 虚拟机数量较多时需优化查询性能
- 考虑添加缓存层（Redis）

### 9.3 安全考虑
- API 需添加认证授权
- 敏感操作需记录日志

---

**文档版本：** v1.0  
**创建人：** OpenClaw Agent  
**创建时间：** 2026-03-26  
**最后更新：** 2026-03-26
