# Code Wiki - IT资产管理系统项目文档

## 目录

- [项目概述](#1-项目概述)
- [项目架构](#2-项目架构)
- [主要模块职责](#3-主要模块职责)
- [核心数据模型](#4-核心数据模型)
- [关键类与函数说明](#5-关键类与函数说明)
- [依赖关系](#6-依赖关系)
- [项目运行方式](#7-项目运行方式)
- [配置文件说明](#8-配置文件说明)
- [数据库设计](#9-数据库设计)

---

## 1. 项目概述

### 1.1 项目简介

本项目是一个基于Django框架开发的IT资产管理系统，主要用于管理和监控企业内部的IT基础设施，包括物理服务器、虚拟服务器、网络设备和其它设备。项目实现了设备信息扫描、用户认证、权限管理、设备分类统计等功能。

### 1.2 项目结构

```
/workspace/j26sxn/imoocc_online/
├── admin/                          # Django管理后台配置
│   ├── settings/                    # 环境配置文件
│   │   ├── settings.py            # 生产环境配置
│   │   └── local_cj.py            # 本地开发配置
│   ├── urls.py                     # 主路由配置
│   └── wsgi.py                     # WSGI配置
├── apps/                           # 业务应用模块
│   ├── users/                      # 用户管理模块
│   ├── detail/                     # 设备详情模块
│   ├── operations/                 # 操作记录模块
│   └── reports/                     # 报表模块
├── scanhosts/                      # 主机扫描模块
│   ├── lib/                        # SSH/SNMP库封装
│   ├── util/                       # 扫描工具集
│   └── modules/                    # 第三方依赖模块
├── conf/                          # 配置文件目录
├── extra_apps/                    # 扩展应用
├── templates/                     # 前端模板
├── static/                        # 静态资源
└── main.py                        # 扫描主程序入口
```

### 1.3 技术栈

- **后端框架**：Django 1.8.2
- **数据库**：MySQL
- **缓存**：Redis
- **消息队列**：MongoDB
- **扫描工具**：Nmap、SNMP
- **远程连接**：Paramiko（SSH）
- **前端**：HTML + Bootstrap

---

## 2. 项目架构

### 2.1 整体架构图

```
┌─────────────────────────────────────────────────────────┐
│                    Web 客户端层                          │
│         (浏览器 ←→ Django Admin ←→ 用户界面)             │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                   Django 应用层                          │
│  ┌──────────┬──────────┬──────────┬──────────┐        │
│  │  Users   │  Detail  │Operations│  Reports │        │
│  └──────────┴──────────┴──────────┴──────────┘        │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                   扫描服务层                             │
│  ┌──────────┬──────────┬──────────┬──────────┐        │
│  │  Nmap    │  Paramiko │  SNMP    │ Filters  │        │
│  └──────────┴──────────┴──────────┴──────────┘        │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                   数据存储层                             │
│  ┌──────────┬──────────┬──────────┬──────────┐        │
│  │   MySQL  │   Redis  │  MongoDB │   Logs   │        │
│  └──────────┴──────────┴──────────┴──────────┘        │
└─────────────────────────────────────────────────────────┘
```

### 2.2 模块依赖关系

```
users (用户认证)
  ↓ 依赖
detail (设备管理)
  ↓ 依赖
operations (操作记录)
  ↓ 依赖
scanhosts (扫描服务)
  ↓ 依赖
reports (报表统计)
```

---

## 3. 主要模块职责

### 3.1 用户管理模块 (users)

**模块路径**：`/workspace/j26sxn/imoocc_online/apps/users/`

**核心功能**：
- 用户注册与登录
- 用户权限验证
- 会话管理
- 密码加密存储

**关键文件**：
- `models.py` - 用户数据模型
- `views.py` - 登录注册视图
- `middleware.py` - URL中间件
- `utils/verify.py` - 验证工具

**权限等级**：
- `0`：普通用户
- `1`：后台管理员
- `2`：超级管理员

### 3.2 设备详情模块 (detail)

**模块路径**：`/workspace/j26sxn/imoocc_online/apps/detail/`

**核心功能**：
- 物理服务器信息管理
- 虚拟服务器信息管理
- 网络设备信息管理
- 设备分类统计
- 设备详情展示

**关键文件**：
- `models.py` - 设备数据模型
- `views.py` - 设备管理视图
- `utils/handdles.py` - 设备分类处理
- `utils/machines.py` - 设备管理工具

### 3.3 操作记录模块 (operations)

**模块路径**：`/workspace/j26sxn/imoocc_online/apps/operations/`

**核心功能**：
- 设备操作记录
- 操作历史查询
- 设备状态追踪

**关键文件**：
- `models.py` - 操作记录模型

### 3.4 主机扫描模块 (scanhosts)

**模块路径**：`/workspace/j26sxn/imoocc_online/scanhosts/`

**核心功能**：
- 网络主机发现
- SSH远程连接
- SNMP信息查询
- 虚拟机类型识别
- 宿主机关系梳理

**关键文件**：
- `main.py` - 扫描主程序入口
- `models.py` - 扫描结果模型
- `util/nmap_all_server.py` - Nmap扫描封装
- `util/j_filter.py` - 过滤规则处理
- `util/get_pv_relation.py` - 宿主机关系获取
- `lib/J_do.py` - SSH执行封装

---

## 4. 核心数据模型

### 4.1 用户模型 (UserInfo)

**文件位置**：`apps/users/models.py`

```python
class UserInfo(models.Model):
    user_name = models.CharField(max_length=15)      # 用户名
    user_passwd = models.CharField(max_length=40)  # 密码(SHA1加密)
    user_level = models.CharField(max_length=2)     # 权限等级
```

**表名**：`userinfo`

### 4.2 连接信息模型 (ConnectionInfo)

**文件位置**：`apps/detail/models.py`

```python
class ConnectionInfo(models.Model):
    ssh_username = models.CharField()   # SSH用户名
    ssh_userpasswd = models.CharField() # SSH密码(加密存储)
    ssh_hostip = models.CharField()    # 主机IP
    ssh_host_port = models.CharField() # SSH端口
    ssh_rsa = models.CharField()       # SSH私钥
    ssh_status = models.IntegerField() # 连接状态(0失败/1成功)
    ssh_type = models.IntegerField()   # 连接类型
    sn_key = models.CharField()         # 唯一设备ID
```

**表名**：`connectioninfo`

### 4.3 物理服务器模型 (PhysicalServerInfo)

**文件位置**：`apps/detail/models.py`

```python
class PhysicalServerInfo(models.Model):
    server_ip = models.CharField()       # 服务器IP
    machine_brand = models.CharField()    # 服务器品牌
    system_ver = models.CharField()       # 操作系统版本
    sys_hostname = models.CharField()     # 主机名
    mac = models.CharField()             # MAC地址
    sn = models.CharField()              # SN序列号
    vir_type = models.CharField()        # 宿主机类型
    ser_cabin = models.ForeignKey()      # 关联机柜
    conn_phy = models.ForeignKey()       # 关联连接信息
```

**表名**：`physicalserverinfo`

### 4.4 虚拟服务器模型 (VirtualServerInfo)

**文件位置**：`apps/detail/models.py`

```python
class VirtualServerInfo(models.Model):
    server_ip = models.CharField()        # 虚拟服务器IP
    server_type = models.CharField()       # 服务器类型(KVM/Docker/VMware)
    system_ver = models.CharField()        # 操作系统版本
    sys_hostname = models.CharField()      # 主机名
    mac = models.CharField()              # MAC地址
    sn = models.CharField()               # SN序列号
    vir_phy = models.ForeignKey()         # 关联宿主机
    conn_vir = models.ForeignKey()        # 关联连接信息
```

**表名**：`virtualserverinfo`

### 4.5 网络设备模型 (NetWorkInfo)

**文件位置**：`apps/detail/models.py`

```python
class NetWorkInfo(models.Model):
    host_ip = models.CharField()        # 设备IP
    host_name = models.CharField()       # 设备名称
    sn = models.CharField()            # SN序列号
    net_cab = models.ForeignKey()       # 关联机柜
```

**表名**：`networkinfo`

### 4.6 机柜模型 (CabinetInfo)

**文件位置**：`apps/detail/models.py`

```python
class CabinetInfo(models.Model):
    cab_name = models.CharField()      # 机柜编号
    cab_lever = models.CharField()     # 机器U数(1-10)
```

**表名**：`cabinetinfo`

### 4.7 操作记录模型 (MachineOperationsInfo)

**文件位置**：`apps/operations/models.py`

```python
class MachineOperationsInfo(models.Model):
    number = models.IntegerField()      # 操作序号
    person = models.CharField()         # 操作人
    time = models.DateTimeField()       # 操作时间
    sn_key = models.CharField()         # 设备唯一标识
    machine_type = models.CharField()   # 设备类型
    operation = models.TextField()      # 操作内容
    state = models.IntegerField()       # 设备状态
    remark = models.TextField()         # 备注
```

**表名**：`machineoperationsinfo`

---

## 5. 关键类与函数说明

### 5.1 用户认证相关

#### 5.1.1 UrlMiddelware 中间件

**文件位置**：`apps/users/middleware.py`

```python
class UrlMiddelware:
    def process_view(self, request, *args, **kwargs):
        # 记录用户访问路径，用于登录后重定向
        if request.path not in ["/user/register/", "/user/login/", ...]:
            request.session['path'] = request.get_full_path()
```

**功能**：记录用户访问路径，实现登录后重定向功能。

#### 5.1.2 login_haddle 登录处理

**文件位置**：`apps/users/views.py`

```python
def login_haddle(request):
    # 1. 获取POST提交的表单数据
    # 2. SHA1加密密码
    # 3. 查询数据库验证用户
    # 4. 创建session会话
    # 5. 设置Cookie(记住用户名功能)
    # 6. 重定向到原访问页面
```

### 5.2 设备管理相关

#### 5.2.1 Machines 设备管理类

**文件位置**：`apps/detail/utils/machines.py`

```python
class Machines:
    def all_machines(self, model_class):
        # 返回指定模型的所有记录
        
    def filter_machines(self, model_class, **kwargs):
        # 根据条件过滤设备
        
    def filter_phy_servers(self, Vir_Type):
        # 过滤物理服务器(支持KVM/Docker/ESXi)
        
    def filter_vir_servers(self, Vir_Phy_Id):
        # 过滤虚拟服务器
        
    def get_all_count(self):
        # 获取所有设备的统计数量
```

#### 5.2.2 ServersClassify 服务器分类

**文件位置**：`apps/detail/utils/handdles.py`

```python
class ServersClassify:
    def __init__(self):
        self.phy = {'phy_num': 0, 'kvm': 0, 'docker': 0, 'esxi': 0, 'other': 0}
        self.vir = {'vir_num': 0, 'kvm': 0, 'docker': 0, 'vmware': 0}
    
    def phy_classify(self, phy_servers):
        # 分类统计物理服务器
        # vir_type: 0=KVM, 1=Docker, 2=ESXi
        
    def vir_classify(self, vir_servers):
        # 分类统计虚拟服务器
        # server_type前缀: VM=VMware, Do=Docker, KV=KVM
```

#### 5.2.3 PageDeal 分页处理

**文件位置**：`apps/detail/utils/handdles.py`

```python
class PageDeal:
    def pages_deal(self, num, objects):
        # 处理页码，确保在有效范围内
        
class PaginatorDeal:
    def paginator_obj(self, obj_list):
        # 创建分页器，每页10条
        
    def page_obj(self, obj_list, num):
        # 获取指定页的数据
```

### 5.3 扫描服务相关

#### 5.3.1 FilterRules 过滤规则

**文件位置**：`scanhosts/util/j_filter.py`

```python
class FilterRules:
    def run(self):
        # 主流程：
        # 1. 获取所有扫描数据
        # 2. 初始化唯一标识(优先SN，其次MAC)
        # 3. 插入连接信息表
        # 4. 分类过滤(物理/虚拟/网络)
        
    def return_key_mac(self, mac_address):
        # 提取可用作唯一标识的MAC地址
        
    def init_uniq_key(self, key_obj_list):
        # 初始化唯一键映射
        
    def insert_db_conn_info(self):
        # 向ConnectionInfo表插入连接数据
        
    def filter_type(self):
        # 根据类型分类插入对应表
```

#### 5.3.2 GetHostType 宿主机类型识别

**文件位置**：`scanhosts/util/get_pv_relation.py`

```python
class GetHostType:
    def get_host_type(self, pv_dict):
        # 识别宿主机上运行的虚拟化类型
        # 类型映射:
        #   'qemu-system-x86_64' → KVM宿主机
        #   'docker-containerd' → Docker宿主机
        #   'vmx' → ESXi宿主机
        
        # 通过SSH执行命令检测进程
        # 返回: 宿主机IP列表
```

#### 5.3.3 J_ssh_do SSH执行封装

**文件位置**：`scanhosts/lib/J_do.py`

```python
class J_ssh_do:
    def run(self, host_ip, commands):
        # SSH远程执行命令
        # 支持密码和密钥认证
        # 返回执行结果字典
```

### 5.4 视图函数说明

#### 5.4.1 detail模块视图

**文件位置**：`apps/detail/views.py`

```python
def index(request):
    # 首页视图
    # 返回：设备统计信息、机柜列表、网络设备列表等

@login
def machine_list(request):
    # 设备列表视图
    # 支持类型过滤：cb(机柜)、p(物理)、v(虚拟)、n(网络)、c(连接)等

@login
@level
def detail(request, types):
    # 设备详情视图
    # 根据类型展示不同设备的详细信息

@login
def classify(request):
    # 分类列表视图
    # 支持KVM、Docker、ESXi、VMware等分类筛选
```

---

## 6. 依赖关系

### 6.1 Python依赖包

```
Django>=1.8.2
djangorestframework>=3.7
paramiko>=1.9.0          # SSH客户端
python-nmap>=0.6.1        # Nmap扫描封装
PyYAML>=3.12              # YAML配置解析
MySQL-python>=1.2.5       # MySQL数据库驱动
redis>=2.10.6             # Redis客户端
pymongo>=3.6.0            # MongoDB客户端
python-crontab>=0.21.0    # 定时任务
crispy-forms>=1.7.2        # 表单美化
```

### 6.2 系统依赖

```
nmap>=7.0                 # 网络扫描工具
snmp                      # SNMP工具包
sshpass                   # SSH密码认证工具
```

### 6.3 外部服务依赖

```yaml
MySQL:
  Host: 127.0.0.1
  Port: 3306
  Database: imoocc
  
Redis:
  Host: 192.168.1.108
  Port: 6379
  DB: 3
  
MongoDB:
  Host: 192.168.1.108
  Port: 27017
```

---

## 7. 项目运行方式

### 7.1 环境准备

```bash
# 1. 创建Python虚拟环境
python3 -m venv venv
source venv/bin/activate  # Linux
# 或 venv\Scripts\activate  # Windows

# 2. 安装依赖
pip install Django==1.8.2
pip install paramiko python-nmap PyYAML
pip install MySQL-python redis pymongo

# 3. 配置数据库
mysql -u root -p
CREATE DATABASE imoocc CHARACTER SET utf8mb4;
CREATE USER 'imoocc'@'localhost' IDENTIFIED BY 'imoocccom';
GRANT ALL PRIVILEGES ON imoocc.* TO 'imoocc'@'localhost';
FLUSH PRIVILEGES;
```

### 7.2 初始化项目

```bash
cd /workspace/j26sxn/imoocc_online

# 1. 数据库迁移
python manage.py makemigrations
python manage.py migrate

# 2. 创建超级用户
python manage.py createsuperuser

# 3. 收集静态文件
python manage.py collectstatic
```

### 7.3 启动服务

```bash
# 开发环境启动
python manage.py runserver 0.0.0.0:8000

# 生产环境启动(使用gunicorn)
pip install gunicorn
gunicorn admin.wsgi:application --bind 0.0.0.0:8000 --workers 4
```

### 7.4 运行主机扫描

```bash
# 配置扫描参数
vim conf/scanhosts.yaml

# 执行扫描程序
python main.py
```

### 7.5 定时任务配置

```python
# settings.py 中配置
INSTALLED_APPS = (
    'django_crontab',
    # ... 其他应用
)

CRONJOBS = [
    # 每天凌晨2点执行主机扫描
    ('0 2 * * *', 'django.core.management.call_command', ['main']),
]
```

---

## 8. 配置文件说明

### 8.1 主配置文件 (settings.py)

**文件位置**：`admin/settings/settings.py`

**关键配置项**：

```python
# 数据库配置
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'imoocc',
        'USER': 'imoocc',
        'PASSWORD': 'imoocccom',
        'HOST': '127.0.0.1',
    }
}

# 邮件配置
EMAIL_HOST = "smtp.163.com"
EMAIL_HOST_USER = "xxx@email.com"
EMAIL_HOST_PASSWORD = "123456"

# MongoDB配置
MONGO_HOST = '192.168.1.108'
MONGO_PORT = '27017'

# Redis配置
REDSI_KWARGS_LPUSH = {"host": '192.168.1.108', 'port': 6379, 'db': 3}
```

### 8.2 扫描配置文件 (scanhosts.yaml)

**文件位置**：`conf/scanhosts.yaml`

```yaml
# 主机扫描配置
hostsinfo:
  nets: ['192.168.1']           # 扫描网段
  ports: "22,222,2222"          # SSH端口列表
  ssh_key_file: ['/opt/key/first_rsa']  # SSH密钥文件
  ssh_pass: ['123456']          # SSH密码列表
  syscmd_list:                  # 系统信息获取命令
    - "cat /etc/issue"
    - "hostname"
    - "cat /sys/class/net/[^vtlsb]*/address"
    - "dmidecode -s system-serial-number"
  black_list: []                # 黑名单IP

# Docker配置
dockerinfo:
  ssh_pass: ['123456']

# 网络设备配置
netinfo:
  sysname_oid: "1.3.6.1.2.1.1.5.0"  # SNMP OID
  sn_oids: ["1.3.6.1.2.1.47.1.1.1.1.11.1"]
  community: "public"            # SNMP社区名
  login_enable: 'True'           # 启用登录
  backup_enable: 'True'         # 启用备份
```

### 8.3 路由配置 (urls.py)

**文件位置**：`admin/urls.py`

```python
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^user/', include('users.urls')),
    url(r'^detail/', include('detail.urls')),
    url(r'^operations/', include('operations.urls')),
]
```

---

## 9. 数据库设计

### 9.1 ER关系图

```
┌──────────────┐     1:N      ┌──────────────┐
│ CabinetInfo │──────────────│PhysicalServer│
│   (机柜)    │              │    Info      │
└──────────────┘              │  (物理服务器) │
       │                       └──────────────┘
       │ 1:N                        │
       ↓                           │ N:1
┌──────────────┐                   ↓
│ NetWorkInfo │              ┌──────────────┐
│  (网络设备)  │              │ConnectionInfo│
└──────────────┘              │   (连接信息)  │
                              └──────────────┘
                                    │
                                    │ 1:N
                                    ↓
┌──────────────┐              ┌──────────────┐
│VirtualServer │──────────────│ MachineOpera │
│    Info     │              │ tionsInfo    │
│ (虚拟服务器) │              │  (操作记录)  │
└──────────────┘              └──────────────┘
```

### 9.2 主要数据表说明

| 表名 | 说明 | 主要字段 |
|------|------|----------|
| `userinfo` | 用户信息表 | user_name, user_passwd, user_level |
| `connectioninfo` | 连接信息表 | ssh_username, ssh_hostip, sn_key |
| `cabinetinfo` | 机柜信息表 | cab_name, cab_lever |
| `physicalserverinfo` | 物理服务器表 | server_ip, machine_brand, vir_type |
| `virtualserverinfo` | 虚拟服务器表 | server_ip, server_type, vir_phy_id |
| `networkinfo` | 网络设备表 | host_ip, host_name, sn |
| `netconnectioninfo` | 网络设备连接表 | tel_username, tel_hostip |
| `othermachineinfo` | 其它设备表 | ip, machine_name, reson_str |
| `hostloginifo` | 扫描初始化表 | ip, hostname, sn, mathine_type |
| `machineoperationsinfo` | 操作记录表 | sn_key, operation, time |
| `statisticsrecord` | 统计记录表 | all_count, pyh_count, vir_count |

### 9.3 外键关系

```sql
-- 物理服务器关联机柜和连接信息
PhysicalServerInfo.ser_cabin_id → CabinetInfo.id
PhysicalServerInfo.conn_phy_id → ConnectionInfo.id

-- 虚拟服务器关联物理服务器和连接信息
VirtualServerInfo.vir_phy_id → PhysicalServerInfo.id
VirtualServerInfo.conn_vir_id → ConnectionInfo.id

-- 网络设备关联机柜
NetWorkInfo.net_cab_id → CabinetInfo.id

-- 网络设备连接关联网络设备
NetConnectionInfo.dev_info_id → NetWorkInfo.id
```

---

## 附录

### A. 常用命令

```bash
# 数据库迁移
python manage.py makemigrations
python manage.py migrate

# 创建应用
python manage.py startapp app_name

# 超级用户管理
python manage.py createsuperuser
python manage.py changepassword username

# 静态文件
python manage.py collectstatic
```

### B. 目录结构说明

```
/workspace/
├── CookBook_and_code-master/     # Python学习教程(教学项目)
│   ├── 1-python基础/             # Python基础
│   ├── 11-AirPlane/              # Tkinter飞机大战
│   ├── 12-多线程/                # 多线程编程
│   ├── 13-协程/                  # 协程编程
│   ├── 14-Spider/                # 网络爬虫
│   ├── 17-Django/                # Django框架
│   └── 18-DRF/                   # Django REST Framework
│
├── Linux初级课程课件/             # Linux学习资料
│
└── j26sxn/                       # IT资产管理系统(生产项目)
    └── imoocc_online/            # 主应用
```

### C. 注意事项

1. **安全性**：生产环境需修改`SECRET_KEY`和数据库密码
2. **性能**：大规模扫描时注意Nmap频率限制
3. **日志**：定期清理日志文件，避免占用过多磁盘空间
4. **备份**：定期备份数据库和配置文件

---

**文档版本**：v1.0  
**生成时间**：2026-05-21  
**维护建议**：
- 每次模块更新后同步文档
- 记录重大架构变更
- 保持示例代码与实际代码同步
