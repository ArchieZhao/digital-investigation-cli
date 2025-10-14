# Digital Investigation CLI · AI Agent 工作手册

> 本文件为 Claude Code、ChatGPT Codex 等 AI 编程助手提供项目上下文和工作指南。
> **重要**：请始终使用中文回复用户，除非用户明确要求使用其他语言。

## 项目概览

**Digital Investigation CLI** 是基于 Google Gemini 的 AI 驱动数字调查工具，专注于快速高效完成电子数据调查分析任务。

- **上游项目**：Fork 自 [Gemini CLI](https://github.com/google-gemini/gemini-cli)
- **核心定位**：将 LLM AI AGENT 的知识和能力应用于数字调查场景
- **主要用户**：安全分析师、调查人员、事件响应团队、竞赛选手
- **许可证**：Apache 2.0

### 核心原则

**🎯 以完成调查分析为王** - 快速找到答案，解决问题，不被形式约束拖慢节奏

### 关键特性

- 🔍 手机取证、服务器分析、日志查询、数据恢复
- 🗄️ 数据库提取、聊天记录分析、密码破解
- 🛠️ Docker容器分析、网站重构、API逆向
- 🔌 MCP (Model Context Protocol) 扩展支持
- 💻 终端优先设计，支持交互和非交互模式

### 工具定位

**AI Agent 职责**：

- ✅ **快速响应** - 优先提供能解决问题的方法，不纠结形式
- ✅ **灵活变通** - 多种方法并举，黑盒白盒结合，怎么快怎么来
- ✅ **实战导向** - 以找到答案为目标，服务于实际调查需求

## 技术栈

```yaml
运行时: Node.js >= 20
主要语言: TypeScript
UI 框架: Ink (React for CLI)
测试框架: Vitest
构建工具: tsup
包管理: npm workspaces (monorepo)
AI 服务: Google Gemini API / Vertex AI
扩展协议: MCP (Model Context Protocol)
```

## 项目结构

```
digital-investigation-cli/
├── packages/
│   ├── cli/              # 主 CLI 应用
│   ├── gemini-api-client/ # Gemini API 客户端
│   ├── ink-markdown/     # Markdown 渲染组件
│   └── ...
├── integration-tests/    # 集成测试
├── scripts/              # 构建和工具脚本
├── docs/                 # 用户文档
│   ├── get-started/      # 快速入门
│   ├── cli/              # CLI 命令参考
│   ├── tools/            # 工具和扩展
│   └── ...
├── README.md             # 项目说明（中文）
├── AGENT.md              # AI Agent 详细指南（中文）
├── GEMINI.md             # 代码规范和架构
└── CLAUDE.md             # 本文件
```

## 代码风格与约定

### TypeScript 规范

```typescript
// ✅ 推荐：使用普通对象 + 类型定义
type UserConfig = {
  apiKey: string;
  model: string;
};

// ❌ 避免：不必要的类
class UserConfig {
  constructor(public apiKey: string) {}
}
```

### React/Ink 组件规范

```typescript
// ✅ 正确：Hook 在顶层调用，逻辑清晰
function MyComponent() {
  const [data, setData] = useState<Data | null>(null);

  useEffect(() => {
    loadData().then(setData);
    return () => cleanup(); // 清理函数
  }, []);

  return <Text>{data?.value}</Text>;
}

// ❌ 错误：业务逻辑塞进 useEffect
function BadComponent() {
  useEffect(() => {
    // 大量业务逻辑...
  }, []);
}
```

### 命名约定

- CLI 参数：短横线式（`--output-format`、`--include-directories`）
- 文件名：短横线式（`file-system-tools.ts`）
- 函数/变量：驼峰式（`parseLogFile`、`apiClient`）
- 类型/接口：帕斯卡式（`CommandOptions`、`AnalysisResult`）

### 测试要求

- 使用 Vitest，测试文件与源文件同目录
- 命名格式：`*.test.ts` 或 `*.test.tsx`
- 善用 `vi.mock` 模拟依赖
- Ink 组件测试使用 `render` 和 `lastFrame()`

```typescript
import { render } from 'ink-testing-library';
import { describe, it, expect } from 'vitest';

describe('MyComponent', () => {
  it('renders correctly', () => {
    const { lastFrame } = render(<MyComponent />);
    expect(lastFrame()).toContain('expected text');
  });
});
```

## 常见调查任务类型

参考 `Forensics_Notes/WP - 取证比赛WP/` 中的实战案例，digital investigation 的典型任务包括：

### 手机取证

#### iOS 取证
**常见任务**：
- iTunes备份解析（`/AppDomain-*/`目录结构）
- 聊天记录提取：WhatsApp/Signal/微信数据库
- 照片库分析（`Photos.sqlite`）：EXIF、位置、创建时间
- 备忘录破解（加密备忘录密码爆破）
- App数据提取：iBooks、地图、MTR、导航App
- 飞行记录分析（DJI App `FlightRecords`）
- AirDrop传输记录（`com.apple.sharingd`）

**AirDrop取证**（重点！新兴技术）：
```bash
# macOS 13+ 日志位置
# 使用控制台查看 system_logs.logarchive

# 可提取信息（从 sharingd 进程日志）
- 发送人姓名（Unicode编码，需解码）
- 设备名称
- AppleID邮箱
- 手机号SHA256哈希（前后各10位可见）
- 文件名（.pvt格式）
- 传输识别码（UUID）
- 时间戳

# 手机号爆破（重要技巧！）
# 原理：SHA256(区号+手机号)
# 已知部分哈希值时，可暴力枚举
```

**Python爆破脚本示例**：
```python
from hashlib import sha256

# 中国手机号：86 + 运营商前缀 + 8位号码
area_code = "86"
begins = ["133","135","136","137","138","139",  # 移动
          "186","187","188","189",              # 移动
          "130","131","132","145","155","156",  # 联通
          "180","181","185",                    # 电信
          "133","153","177","173","189"]        # 电信

# 假设已知哈希的前5位和后5位
known_prefix = "eeb48"  # 示例
known_suffix = "2d99d"

for begin in begins:
    for i in range(10000000):  # 0000000-9999999
        phone = area_code + begin + str(i).zfill(8)
        h = sha256(phone.encode()).hexdigest()
        if h[:5] == known_prefix and h[-5:] == known_suffix:
            print(f"找到号码: {phone[2:]}")  # 去掉86
            break
```

**快速定位**：
```bash
# WhatsApp聊天数据库
AppDomainGroup-group.net.whatsapp.WhatsApp.shared/ChatStorage.sqlite

# 照片库
AppDomain-com.apple.mobileslideshow/Media/PhotoData/Photos.sqlite

# 备忘录
AppDomainGroup-group.com.apple.notes/NoteStore.sqlite

# 位置记录
grep -r "Latitude\|Longitude" .
```

#### Android 取证
**常见任务**：
- 设备信息（`build.prop`、UserAgent）
- Gmail邮件分析（`gmail.db`，注意zlib压缩格式）
- 照片EXIF和GPS信息
- 删除文件恢复（DCIM缓存、Google Photos缓存）
- 应用数据库（`/data/data/包名/databases/`）
- 快递单/截图OCR识别

**技巧**：
- 缩略图在，原图可能在缓存：`find . -name "*cache*" -type f -size +100k`
- 文件扩展名改变但签名未变：用火眼"特征分析"
- Google Photos缓存：`/data/com.google.android.apps.photos/cache/glide_cache/`

### 服务器分析

#### 宝塔面板（重点）
**绕密登录**：
```bash
# 方法1：清除限制
cp -r /www/backup/panel/ /root/ && \
cp -r /www/server/panel/data/ /root && \
rm -f /www/server/panel/data/close.pl && \
bt 23 && bt 11 && bt 12 && bt 13 && bt 24 && bt 5

# 方法2：查看端口和安全路径
bt 14
```

**关键位置**：
```bash
# 宝塔日志
/www/server/panel/logs/request/*.json.gz
/www/server/panel/data/default.db  # SQLite数据库

# 网站配置
/www/server/panel/vhost/nginx/*.conf
/www/wwwlogs/网站.log  # 访问日志，可找后台地址

# 数据库配置
/www/server/panel/data/default.db  # 数据库密码
```

**典型任务**：
- 最早登录IP：查看`default.db`的`logs`表
- 网站域名：`nginx -T | grep server_name`
- 数据库密码：宝塔面板数据库管理

#### Web应用分析
**Java应用（jar包）**：
```bash
# 1. 提取jar包
find /web /data /var/www -name "*.jar"

# 2. 使用jd-gui或jadx反编译
jadx-gui application.jar

# 3. 查看关键配置
# Spring: application.yml / application.properties
# 数据库配置、Redis配置、API密钥

# 4. 分析加密逻辑
# 找LoginController、UserService等关键类
```

**PHP应用**：
```bash
# 配置文件
find /www/wwwroot -name "config*.php" -o -name ".env"

# 查看密码加密逻辑
grep -r "password" app/  | grep -i "md5\|sha\|crypt"

# 找后台地址
grep -r "admin\|login" /www/wwwlogs/*.log

# 数据库配置常见文件
/config/database.php
/application/database.php  # ThinkPHP
/.env
```

**典型解密**：
```php
// base64编码的配置
<?php
$config = array(
    'DB_HOST' => base64_decode('bG9jYWxob3N0'),
    'DB_USER' => base64_decode('cm9vdA=='),
    'DB_PWD'  => base64_decode('cGFzc3dvcmQ='),
);
```

#### 数据库取证
**MySQL快速操作**：
```sql
-- 1. 查看所有表
SHOW TABLES;

-- 2. 查看表结构和注释（重要！）
SHOW CREATE TABLE users;
DESC users;

-- 3. 统计类查询
SELECT COUNT(*) FROM users WHERE status='冻结';
SELECT SUM(amount) FROM orders WHERE status='成功';

-- 4. 用户关系追踪
SELECT * FROM users WHERE parent_id=(SELECT id FROM users WHERE username='张三');

-- 5. 时间筛选
SELECT * FROM logs WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31';

-- 6. 开启general_log（查看实时SQL）
SET GLOBAL general_log = ON;
SET GLOBAL general_log_file='/tmp/general.log';
-- 然后：tail -f /tmp/general.log

-- 7. 用户关系链追踪（传销网络分析）
-- 找出所有下级（递归查询）
WITH RECURSIVE user_tree AS (
    SELECT id, username, parent_id, 1 as level FROM users WHERE id=12345
    UNION ALL
    SELECT u.id, u.username, u.parent_id, ut.level+1
    FROM users u INNER JOIN user_tree ut ON u.parent_id=ut.id
)
SELECT * FROM user_tree ORDER BY level;

-- 8. 统计每个用户的下级数量
SELECT parent_id, COUNT(*) as downline_count
FROM users
GROUP BY parent_id
ORDER BY downline_count DESC;

-- 9. 金额统计（充值、提现、余额）
SELECT
    SUM(CASE WHEN type='recharge' THEN amount ELSE 0 END) as total_recharge,
    SUM(CASE WHEN type='withdraw' THEN amount ELSE 0 END) as total_withdraw,
    SUM(amount) as balance
FROM transactions WHERE user_id=12345;

-- 10. 时间范围内活跃用户
SELECT user_id, COUNT(*) as login_count
FROM login_logs
WHERE login_time BETWEEN '2024-01-01' AND '2024-12-31'
GROUP BY user_id
HAVING login_count > 10;
```

**binlog分析**（删除数据恢复）：
```bash
# 查看binlog
mysqlbinlog --base64-output=decode-rows -v /var/lib/mysql/mysql-bin.000001 > binlog.txt

# 搜索DELETE操作
grep -A 10 "DELETE FROM" binlog.txt
```

**MongoDB**：
```bash
# 连接
mongo --host IP --port 27017

# 查看数据库和集合
show dbs
use rocketchat
show collections

# 查询
db.users.find({"roles": "admin"})
```

#### Docker容器分析
**快速命令**：
```bash
# 1. 查看所有容器（包括停止的）
docker ps -a
podman ps -a

# 2. 查看容器详细信息（重点！）
docker inspect <container_id> | grep -i "password\|mysql\|port"

# 3. 查看容器日志
docker logs <container_id>

# 4. 进入容器
docker exec -it <container_id> /bin/bash

# 5. 查看端口映射
docker port <container_id>

# 6. 查看镜像
docker images

# 7. 导出容器文件系统
docker export <container_id> > container.tar
```

**典型场景**：
- MySQL容器：找`MYSQL_ROOT_PASSWORD`环境变量
- Redis容器：找端口映射和密码
- Web容器：找代码路径和配置

**Docker高级技巧**：

1. **容器密码重置**（MySQL/PostgreSQL等）：
```bash
# MySQL容器跳过权限验证
docker exec -it <container_id> bash
echo "skip-grant-tables" >> /etc/mysql/my.cnf
docker restart <container_id>

# 进入修改密码
docker exec -it <container_id> mysql
mysql> UPDATE mysql.user SET authentication_string='' WHERE user='root';
mysql> FLUSH PRIVILEGES;

# 删除skip-grant-tables配置并重启
```

2. **多容器环境分析**：
```bash
# 查看容器间网络
docker network ls
docker network inspect bridge

# 查看容器连接关系
docker inspect <container_id> | grep -A 20 "Networks"

# 导出容器完整文件系统
docker export <container_id> > container.tar
tar -xf container.tar

# 查看容器创建时间和启动命令
docker inspect <container_id> | grep Created
docker inspect <container_id> | grep -A 5 "Cmd"
```

3. **容器配置和环境变量提取**（重要！）：
```bash
# 查看完整配置（JSON格式）
docker inspect <container_id> > container_config.json

# 提取环境变量（密码常在这里）
docker inspect <container_id> | grep -A 50 "Env"

# 提取挂载点（找数据存储位置）
docker inspect <container_id> | grep -A 20 "Mounts"

# 示例：提取MySQL密码
docker inspect mysql_container | grep MYSQL_ROOT_PASSWORD
```

4. **停止容器分析**：
```bash
# 查看停止容器的日志
docker logs <stopped_container_id>

# 启动停止的容器
docker start <container_id>

# 提交容器为镜像（便于分析）
docker commit <container_id> analysis_image:v1
```

#### RAID重组（R-Studio）
**步骤**：
1. 加载所有磁盘镜像（.dsk/.E01）
2. 工具栏 → "创建虚拟快RAID和自动检测"
3. 拖拽磁盘 → 自动检测参数
4. 应用 → 右键"虚拟块RAID" → 创建镜像
5. 选择"逐字节镜像"导出

**注意**：记录RAID参数（左/右同步、块大小）作为答案

#### 系统分析技巧
```bash
# history分析
cat ~/.bash_history
cat /www/server/panel/config/ssh_info/*/history.pl

# 开机启动项
chkconfig --list
systemctl list-unit-files --state=enabled

# 网络配置
cat /etc/sysconfig/network-scripts/ifcfg-*
ip addr

# 时区
timedatectl
cat /etc/timezone

# 系统版本
cat /etc/*-release
uname -a
```

#### NAS服务器取证
**常见NAS系统**：群晖（Synology）、威联通（QNAP）、FreeNAS

**关键取证点**：

1. **系统配置和用户**：
```bash
# 群晖DSM
cat /etc/shadow  # 用户密码哈希
cat /etc.defaults/synoinfo.conf  # 系统信息
cat /etc/VERSION  # 系统版本

# 用户配置
cat /etc/passwd
cat /usr/syno/etc/preference/*/user.config

# 共享文件夹配置
cat /usr/syno/etc/smb.conf
```

2. **应用和服务**：
```bash
# Docker容器（NAS上常运行Docker）
docker ps -a
docker inspect <container_id>

# 数据库（MariaDB/MySQL）
/volume1/@database/  # 数据库数据目录
/var/services/homes/  # 用户家目录

# Web应用
/var/services/web/  # Web服务目录
/var/log/nginx/  # Web访问日志
```

3. **日志分析**：
```bash
# 系统日志
/var/log/messages
/var/log/synolog/  # 群晖专用日志目录

# 登录记录
/var/log/auth.log
last -f /var/log/wtmp

# 应用日志
/var/log/synopkg.log  # 套件安装日志
/volume1/@appstore/  # 应用数据
```

4. **网络配置**：
```bash
# 网络接口
cat /etc/sysconfig/network-scripts/ifcfg-*
ip addr show

# 路由和防火墙
iptables -L -n
cat /etc/iptables/rules.v4
```

5. **虚拟化和嵌套环境**（重要！）：
```bash
# 虚拟机（Virtual Machine Manager）
/var/packages/Virtualization/target/
find /volume1 -name "*.vmdk" -o -name "*.qcow2"

# 嵌套Docker容器分析
# 技巧：NAS → Docker → MySQL → 数据库数据
# 层层深入，不要遗漏任何一层
```

**实战技巧**：
- NAS常作为犯罪团伙的数据中心
- 优先查看Docker容器（Web应用常部署在容器中）
- 注意虚拟机嵌套（NAS上的VM可能运行更多服务）
- 用户家目录可能有大量敏感数据

### 密码破解与绕密

#### Hash爆破
**hashcat常用模式**：
```bash
# MD5
hashcat -m 0 hash.txt wordlist.txt

# SHA1
hashcat -m 100 hash.txt wordlist.txt

# bcrypt（Linux shadow）
hashcat -m 3200 hash.txt wordlist.txt

# bcrypt（常见Web）
hashcat -m 3200 hash.txt wordlist.txt

# 掩码攻击（6位小写字母）
hashcat -m 0 hash.txt -a 3 ?l?l?l?l?l?l

# MySQL
hashcat -m 300 hash.txt wordlist.txt
```

**典型场景**：
```bash
# Linux用户密码
unshadow passwd shadow > unshadowed.txt
john unshadowed.txt
hashcat -m 1800 shadow.txt -a 0 passwords.txt

# Web后台密码（通过日志找到加密逻辑）
# 例：md5(salt + md5(password) + username)
echo -n "9c64c47febe686ce847d3b4b8c3477a00659c7992e268962384eb17fafe88364test" | md5sum
```

#### SSH/服务爆破
```bash
# SSH
hydra -l root -P passwords.txt ssh://IP

# MySQL
hydra -l root -P passwords.txt mysql://IP:3306

# FTP
hydra -l ftpuser -P passwords.txt ftp://IP
```

#### BitLocker/VeraCrypt
**方法1：找密码线索**
```bash
# 检查常见位置
- Documents目录下的txt文件（密码字典）
- 图片隐写（xxd image.jpg | tail）
- 浏览器保存的密码
- 备忘录/便签
```

**方法2：提取恢复密钥**
```bash
# BitLocker恢复密钥
# Windows检材中搜索: "BitLocker恢复密钥"
# 或在加密容器中找相关txt文件
```

#### 网站后台绕密
**方法1：修改数据库密码字段**
```sql
-- 生成bcrypt密文（在线工具或Python）
-- 然后替换数据库中的password字段
UPDATE admin SET password='$2a$10$...' WHERE username='admin';
```

**方法2：修改登录逻辑**
```php
// 找到登录验证代码
if (md5($input_pwd) !== $db_pwd) {  // 原逻辑
if (md5($input_pwd) === $db_pwd) {  // 修改为===（取反）
```

**方法3：添加测试账号**
```sql
-- 用日志中抓到的密码格式，插入测试账号
INSERT INTO admin (username, password, allow) VALUES ('test', 'hash', 'your_ip');
```

#### 密码逻辑逆向（重要！）
**步骤**：
1. 尝试登录 → 抓取请求（浏览器F12或查看日志）
2. 找到密码字段的值（已加密）
3. 搜索源码中的登录逻辑：
```bash
grep -r "password" app/ | grep -i "md5\|sha\|bcrypt"
find . -name "*Login*.php" -o -name "*User*.php"
```
4. 还原加密算法：
```php
// 例：ThinkPHP某项目
$encrypted = md5(C('cfg_adminkey') . md5($password) . $username);
// 翻译成: md5(配置中的key + md5(明文密码) + 用户名)
```
5. 验证：用相同逻辑加密已知密码，看是否匹配

### Windows PC取证

#### 浏览器取证
**Chrome/Edge**：
```
用户数据位置：
C:\Users\用户名\AppData\Local\Google\Chrome\User Data\Default\

关键数据库：
- History（浏览历史）
- Cookies
- Login Data（保存的密码，需解密）
- Web Data（自动填充）
```

**查询技巧**：
```sql
-- 火眼直接查看，或用SQLite工具
-- 搜索关键词：admin、login、config、password
SELECT url, title, datetime(last_visit_time/1000000-11644473600, 'unixepoch', 'localtime')
FROM urls ORDER BY last_visit_time DESC;
```

#### PowerShell历史
```powershell
# 查看历史记录
C:\Users\用户名\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

#### 加密货币钱包
**Electrum**：
- 钱包文件：`AppData\Roaming\Electrum\wallets\`
- 查看地址和交易

**比特币地址特征**：以`1`、`3`或`bc1`开头

#### Signal桌面版
```
数据位置：
C:\Users\用户名\AppData\Roaming\Signal\

关键文件：
- db.sqlite（聊天数据库）
- config.json（配置）
```

**解密**：需要密钥（通常在安装时设置）

### 虚拟化环境

#### ESXi虚拟机
**绕密方法**：
1. FTK挂载ESXI镜像（选"逻辑"挂载）
2. 找到250MB分区的`state.tgz`
3. 解压：`state.tgz → local.tgz → /etc/shadow`
4. 清空root密码：`::`替换密码部分
5. 重新压缩并替换原文件
6. VMware创建虚拟机，使用挂载的物理磁盘
7. 启动后空密码登录

**注意**：勾选"Intel VT-x/AMD-V"（嵌套虚拟化）

#### VMware虚拟机分析
```bash
# 查看虚拟机配置
cat *.vmx

# 挂载vmdk
# 方法1：火眼证据分析直接加载
# 方法2：7-Zip解压vmdk
# 方法3：挂载到Linux: losetup/mount
```

### 社交媒体分析

#### WhatsApp
**iOS**：`AppDomainGroup-group.net.whatsapp.WhatsApp.shared/ChatStorage.sqlite`
**Android**：`/data/data/com.whatsapp/databases/msgstore.db`

**关键表**：
- `ZWAMESSAGE`（消息）
- `ZWACHATSESSION`（会话）
- `ZWAGROUPMEMBER`（群成员）

#### 微信
**数据库**：`EnMicroMsg.db`（加密）
**解密密钥**：IMEI + UIN（需要专用工具）

#### Signal
**加密数据库**：需要密码
**爆破**：使用[apple_cloud_notes_parser](https://github.com/threeplanetssoftware/apple_cloud_notes_parser)类似方法

#### 加密聊天数据库（野火IM、SQLCipher等）
**野火IM（Wildfire IM）**：
```bash
# 数据库位置（Android）
/data/data/cn.wildfirechat.chat/databases/wfc.db

# 加密方式：SQLCipher
# 已知密钥格式：固定字节序列（竞赛题常见）
```

**SQLCipher解密方法**：
```bash
# 方法1：使用DB Browser for SQLite（SQLCipher版）
# 工具 → SQLCipher密码 → 输入密钥

# 方法2：命令行
sqlcipher wfc.db
sqlite> PRAGMA key = "x'00112233445566778899AABBCCDDEEFF'";
sqlite> .tables

# 方法3：Python脚本
from pysqlcipher3 import dbapi2 as sqlite
conn = sqlite.connect('wfc.db')
conn.execute("PRAGMA key='0x00112233...'")
```

**常见加密IM密钥规律**：
- 野火IM：固定16字节模式（`0x00,0x11,0x22...`）
- 微信EnMicroMsg.db：`MD5(IMEI + UIN)[:7]`
- Telegram：local.key文件（需提取）

**密钥获取技巧**：
```bash
# 1. 查看应用源码（jadx反编译APK）
grep -r "PRAGMA key" .
grep -r "SQLCipher" .

# 2. 内存dump（需root或调试权限）
strings memory.dump | grep -i "key"

# 3. 已知规律暴力破解（参考历史比赛题）
```

### 恶意软件分析

#### Metasploit/Meterpreter
**痕迹**：
- `.msf4/logs/`目录（Linux/Mac）
- `sessions/`子目录（交互日志）
- 命令：`geolocate`（获取位置）、`download`（下载文件）

#### PowerShell Empire
**日志**：`/var/lib/powershell-empire/empire/client/downloads/logs/empire_client.log`
**关键信息**：任务ID、下载文件名、时间戳

#### Android恶意软件（APK勒索软件）
**分析步骤**：

1. **反编译APK**：
```bash
# 使用jadx反编译
jadx-gui malware.apk

# 或命令行
jadx -d output_dir malware.apk
```

2. **关键代码定位**：
```bash
# 查找关键类和方法
find . -name "*.java" | xargs grep -l "encrypt"
find . -name "*.java" | xargs grep -l "ransom"
find . -name "*.java" | xargs grep -l "lock"

# 查找MainActivity和Service
grep -r "MainActivity" .
grep -r "extends Service" .
```

3. **加密逻辑分析**（常见模式）：
```java
// 示例：文件加密勒索
public void encryptFiles() {
    String key = "hardcoded_key";  // 硬编码密钥（重点！）
    String algorithm = "AES/CBC/PKCS5Padding";

    // 遍历目录加密文件
    File[] files = getExternalStorageDirectory().listFiles();
    for (File file : files) {
        encrypt(file, key);
        file.renameTo(new File(file.getPath() + ".locked"));
    }
}
```

4. **提取密钥和解密**：
```bash
# 从反编译代码中找密钥
grep -r "key\s*=\s*\"" . | grep -i "encrypt\|decrypt"

# 常见位置
- MainActivity.java（主逻辑）
- Utils.java / CryptoUtils.java（工具类）
- strings.xml（资源文件）
- Native代码（.so文件，需IDA分析）
```

5. **动态分析技巧**：
```bash
# 使用frida hook关键函数
frida -U -f com.malware.app -l hook.js

# hook.js示例
Java.perform(function() {
    var Utils = Java.use("com.malware.CryptoUtils");
    Utils.encrypt.implementation = function(data, key) {
        console.log("Key: " + key);  // 打印密钥
        return this.encrypt(data, key);
    };
});
```

**真实案例模式**：
- 密钥硬编码在代码中（70%）
- Base64编码存储在资源文件（20%）
- 从C&C服务器获取（10%，需网络分析）

### 数据恢复

#### 删除文件恢复
**方法**：
1. 火眼"特征分析" → 识别文件类型
2. 按文件头搜索（JPEG: `FFD8FF`、PDF: `25504446`）
3. 检查缓存目录：
   - `AppData\Local\Temp`
   - Google Photos缓存
   - 浏览器缓存

#### 图片隐写分析
```bash
# 查看十六进制尾部
xxd image.jpg | tail -n 50

# 常见隐写位置
- 文件尾部附加数据
- EXIF注释字段
- LSB隐写（工具：stegsolve）
```

### AI生成图片取证（新兴领域）

#### Stable Diffusion取证
**分析目标**：确定AI生成图片的参数、模型、提示词

**关键文件位置**（以Windows为例）：
```bash
# Stable Diffusion WebUI
stable-diffusion-webui/outputs/
stable-diffusion-webui/models/Stable-diffusion/  # 模型文件
stable-diffusion-webui/log/  # 日志文件

# 图片元数据
# PNG文件中包含生成参数（在PNG chunks中）
```

**提取生成参数**：

1. **从PNG元数据提取**：
```python
from PIL import Image

img = Image.open('generated_image.png')
metadata = img.info
print(metadata.get('parameters'))  # Stable Diffusion参数

# 输出示例：
# prompt: "car on fire, high speed, highway"
# negative_prompt: "people, low quality"
# steps: 20
# sampler: Euler a
# cfg_scale: 7
# seed: 957419862
# size: 512x512
# model_hash: abc123def
```

2. **从exiftool提取**：
```bash
exiftool generated_image.png | grep -i "parameters\|prompt\|seed"
```

3. **从日志文件追踪**：
```bash
# SD WebUI日志包含完整生成历史
cat stable-diffusion-webui/outputs/txt2img-images/2024-03-13/log.txt

# 查找特定图片
grep "00036-957419862.png" log.txt -A 10
```

**模型文件分析**：
```bash
# 模型文件通常是.ckpt或.safetensors格式
# 计算SHA256哈希值
sha256sum model.ckpt

# 常见模型识别
- sd-v1-4.ckpt (Stable Diffusion 1.4)
- v1-5-pruned.ckpt (Stable Diffusion 1.5)
- custom models (自定义训练模型)
```

**典型调查任务**：
```
题目：某嫌疑人使用AI生成"燃烧的汽车"图片，请找出：
1. 使用的模型SHA256哈希值 → models/目录下计算
2. 正向提示词包含哪些 → PNG元数据或日志
3. 生成种子（seed） → PNG元数据parameters字段
4. 监听端口 → 进程列表或配置文件
```

**快速分析流程**：
```bash
# 1. 找SD安装目录
find / -name "stable-diffusion-webui" 2>/dev/null

# 2. 查看生成的图片目录
ls outputs/txt2img-images/2024-*/

# 3. 提取图片参数
exiftool 00036-957419862.png

# 4. 计算模型哈希
sha256sum models/Stable-diffusion/*.ckpt

# 5. 查看端口（默认7860）
netstat -tuln | grep 7860
# 或查看配置
cat webui-user.bat | grep port
```

### 网络取证

#### OpenWRT
**配置文件**：`/etc/config/`目录
**关键配置**：
```bash
/etc/config/network  # 网络配置、静态IP
/etc/config/passwall2  # VPN配置、节点列表
/etc/config/firewall  # 防火墙规则
```

**Docker密码重置**：
```bash
# 进入容器修改MySQL配置
podman exec -it <container> bash
echo "skip-grant-tables" >> /etc/mysql/my.cnf
# 重启后无密码登录修改密码
```

### 实战工作流

**标准流程**（以"找管理员密码"为例）：
```
1. 并行尝试多种方法：
   ├─ 浏览器保存的密码
   ├─ 配置文件搜索（grep -r "password"）
   ├─ 数据库配置文件
   ├─ 日志中的明文密码
   └─ 密码字典爆破

2. 找到密文后：
   ├─ 识别加密算法（MD5/bcrypt/自定义）
   ├─ 在线解密/本地爆破
   └─ 逆向加密逻辑

3. 无法破解：
   ├─ 绕密登录（修改代码/数据库）
   ├─ 添加测试账号
   └─ 重置密码
```

**时间管理**：
- ✅ 先做确定能拿分的题（配置文件、数据库查询）
- ✅ 并行操作（一边仿真一边分析其他检材）
- ✅ 不要死磕一道题超过15分钟
- ✅ 记录所有尝试过的方法（避免重复）

**工具组合**：
- 火眼证据分析（主力，自动取证）
- R-Studio（RAID重组、数据恢复）
- jadx/jd-gui（jar包逆向）
- hashcat（密码爆破）
- SQLite Browser（数据库分析）
- FTK Imager（挂载镜像）
- 火眼仿真（虚拟机快速启动）

## 常见任务工作流

### 1. 理解用户需求

**步骤**：

1. 用中文复述用户目标和约束条件
2. 确认输入、输出和预期行为
3. 明确标注不确定的信息，主动提问

**模板**：

```
我理解您的需求是：[复述需求]

前提条件：
- [列出已知条件]

需要确认：
- [列出待确认项]
```

### 2. 代码调研与定位

**工具选择**：

- 文件搜索：使用 Glob 工具（`**/*.ts`、`**/commands/**`）
- 内容搜索：使用 Grep 工具（`pattern: "function.*parse"`）
- 阅读文件：使用 Read 工具

**关键目录**：

- `packages/cli/src/` - CLI 核心逻辑
- `packages/gemini-api-client/` - API 客户端
- `docs/` - 用户文档
- `integration-tests/` - 集成测试示例

### 3. 设计方案

**原则**：简洁实用，快速迭代

**输出格式**（简化版）：

```markdown
## 方案

### 目标
[一句话说明要达成什么]

### 思路
1. [方法1] - 最快/最直接
2. [方法2] - 备选方案
3. [方法3] - 如果前面都不行

### 可能的坑
- [问题] → [怎么绕过]
```

**注意**：不需要过度设计，边做边调整，能用就行

### 4. 实施与验证

**实施原则**：

- 优先编辑现有文件，避免创建新文件
- 保持代码风格一致
- 添加必要的类型定义
- 复杂逻辑添加简短注释

**验证流程**：

```bash
# 必须在交付前建议执行
npm run preflight  # 构建 + 测试 + 类型检查 + Lint

# 或分步执行
npm run build
npm test
npm run typecheck
npm run lint
```

**交付检查清单**（灵活掌握）：

- [ ] 代码能运行，基本符合项目风格
- [ ] 核心功能有测试（复杂逻辑优先）
- [ ] 重要变更建议执行 `npm run preflight`
- [ ] 使用中文提供变更说明
- [ ] 明确标注"未测试"或"需验证"的部分

**注意**：优先保证功能可用，代码规范可以后续优化

### 5. 交付总结

**模板**：

```markdown
## ✅ 变更总结

### 修改文件
- `path/to/file.ts` - [修改说明]
- `path/to/test.ts` - [测试说明]

### 验证状态
- [✅/⏸️] 构建通过
- [✅/⏸️] 测试通过
- [✅/⏸️] 类型检查通过

### 后续步骤（可选）
1. [可选步骤1]
2. [可选步骤2]

### 使用示例
\`\`\`bash
# 如何使用新功能
\`\`\`
```

## 调查实战技巧

### 常用破解/绕密方法

**密码爆破**：
```bash
# Hash破解（hashcat）
hashcat -m 1800 shadow.txt -a 3 ?l?l?l?l?l?l  # 6位小写字母
hashcat -m 0 hash.txt wordlist.txt           # 字典攻击

# SSH爆破（hydra）
hydra -l root -P passwords.txt ssh://IP

# MySQL爆破
hydra -l root -P passwords.txt mysql://IP
```

**数据库快速查询**：
```sql
-- 统计类
SELECT COUNT(*) FROM users WHERE status='冻结';
SELECT SUM(amount) FROM transactions WHERE type='充值';

-- 关系追踪
SELECT * FROM users WHERE parent_id=12345;

-- 时间筛选
SELECT * FROM logs WHERE created_at BETWEEN '2024-01-01' AND '2024-01-31';
```

**Docker容器分析**：
```bash
# 查看容器列表和ID
docker ps -a

# 查看容器详细信息
docker inspect <container_id>

# 进入容器
docker exec -it <container_id> /bin/bash

# 查看容器IP/端口
docker inspect <container_id> | grep IPAddress
docker port <container_id>
```

**日志快速分析**：
```bash
# 宝塔面板日志
/www/server/panel/logs/request/
cat /www/server/panel/data/default.db  # SQLite数据库

# 查找关键信息
grep -r "password" /path/to/config/
grep -r "mysql" /var/log/
```

### 快速定位技巧

1. **配置文件优先** - Web应用看 `.env`、`application.yml`、`database.php`
2. **浏览器记录** - Windows检材优先查看浏览器保存的密码和历史
3. **命令历史** - Linux看 `.bash_history`、PowerShell历史
4. **文本搜索** - 关键词：`password`、`admin`、`secret`、`config`
5. **并行分析** - 同时查看数据库、配置文件、日志，交叉验证

### 实战案例示例

**案例1：查找网站数据库密码**

目标：获取涉案网站的MySQL数据库密码

快速思路：
1. 查看Web应用配置文件（`.env`、`application.yml`）
2. 查看宝塔面板配置
3. 查看Docker容器环境变量

实施：
```bash
# 方法1：搜索配置文件
find /data -name "*.env" -o -name "application*.yml" | xargs grep -i "password"

# 方法2：查看宝塔数据库
cat /www/server/panel/data/default.db | grep -i mysql

# 方法3：查看Docker容器
docker inspect <container_id> | grep -i "MYSQL_ROOT_PASSWORD"
```

**案例2：统计充值总额**

目标：计算所有成功充值的总金额

快速思路：
1. 找到数据库表名（recharge/payment/order）
2. 确认状态字段含义（status=1表示成功）
3. SQL求和

实施：
```sql
-- 先看表结构
SHOW TABLES LIKE '%recharge%';
DESC lc_recharge;

-- 统计
SELECT SUM(money) FROM lc_recharge WHERE status='1';
```

**案例3：破解加密容器**

目标：解密VeraCrypt容器获取内部文件

快速思路：
1. 找密码字典（Documents目录、桌面）
2. 找密码提示（图片隐写、文本文件）
3. 暴力破解

实施：
```bash
# 1. 查看图片尾部（隐写常见方法）
xxd image.jpg | tail

# 2. 搜索密码文件
find /Users -name "*pwd*" -o -name "*password*"

# 3. 使用找到的密码挂载
veracrypt --mount container.tc --password=qwerasdfzxcv
```

**关键要点**：
- 多路径并行尝试，不要死磕一个方向
- 优先用最直接的方法（配置文件、浏览器密码）
- 善用搜索和grep，快速定位关键信息

## 环境与命令速查

### 开发环境

```bash
# 安装依赖
npm install

# 本地运行
npm start

# 全局安装
npm install -g .

# 验证完整流程
npm run preflight
```

### 认证配置

用户有三种认证方式（优先级从高到低）：

1. **Google 账户登录**（推荐）

   - 60 请求/分钟，1,000 请求/天
   - 自动使用 Gemini 2.5 Pro
2. **Gemini API Key**

   ```bash
   export GEMINI_API_KEY="YOUR_KEY"
   ```
3. **Vertex AI**（企业）

   ```bash
   export GOOGLE_API_KEY="YOUR_KEY"
   export GOOGLE_GENAI_USE_VERTEXAI=true
   ```

### 常用命令

```bash
# 交互模式
gemini
digital-investigation

# 指定证据目录
gemini --include-directories ../evidence,../logs

# 非交互模式
gemini -p "分析日志文件"
gemini -p "查询威胁情报" --output-format json

# 调试模式
DEBUG=* gemini
```

## 文档引用规范

在回答用户问题时，优先引用官方文档：

| 主题       | 文档路径                               |
| ---------- | -------------------------------------- |
| 快速开始   | `docs/get-started/index.md`          |
| 身份验证   | `docs/get-started/authentication.md` |
| 命令参考   | `docs/cli/commands.md`               |
| 自定义命令 | `docs/cli/custom-commands.md`        |
| MCP 集成   | `docs/tools/mcp-server.md`           |
| 企业部署   | `docs/cli/enterprise.md`             |
| 故障排除   | `docs/troubleshooting.md`            |

**引用示例**：

```markdown
关于自定义命令的详细配置，请参考 `docs/cli/custom-commands.md`。
简单来说，您需要在 `.gemini/commands/` 目录创建 Markdown 文件...
```

## 沟通准则

### ✅ 推荐做法

- 使用中文回复（命令/路径/标识符可用英文）
- 分节展示，便于快速执行
- 提供完整可运行的命令和代码
- 明确标注"需确认"、"假设"、"未验证"
- 主动指出风险和缺失信息

### ❌ 避免做法

- 使用英文回复（除非用户要求）
- 冗长段落，缺少结构
- 不完整的代码片段
- 未经验证就断言结论
- 过度关注形式而忽略实际任务需求

## 响应模板

### 简单问题回复

```markdown
[直接回答]

相关命令：
\`\`\`bash
[可执行命令]
\`\`\`

参考文档：`docs/path/to/doc.md`
```

### 调查分析任务回复

```markdown
## 目标
[要查什么/找什么]

## 快速思路
1. [方法1] - 直接查配置文件
2. [方法2] - 数据库查询
3. [方法3] - 日志分析

## 实施
[命令/查询/步骤]

## 结果
[找到的答案/数据]

## 备注
[其他发现/注意事项]
```

### 实现功能任务回复

```markdown
## 实现方案
[简要说明]

## 代码
[代码 + 关键注释]

## 测试
\`\`\`bash
# 快速验证
npm test
\`\`\`

## 注意
[已知问题/待优化点]
```

## 快速参考

### 项目关键文件

- `README.md` - 项目说明和使用指南
- `AGENT.md` - AI Agent 详细指南（本文档的扩展版）
- `GEMINI.md` - 代码规范和架构细节
- `ROADMAP.md` - 开发计划
- `SECURITY.md` - 安全策略

### 重要类型定义

常见类型位置（需要时查阅）：

- CLI 配置：`packages/cli/src/types/`
- API 客户端：`packages/gemini-api-client/src/types.ts`
- MCP 协议：`packages/mcp-client/src/types/`

### 调试技巧

```bash
# 启用详细日志
DEBUG=* gemini

# 查看 API 请求
DEBUG=gemini:api gemini

# 检查配置
cat ~/.gemini/settings.json
```

## 最后检查清单

在每次回复前，快速确认：

- [ ] ✅ 使用中文回复
- [ ] ✅ 提供了快速可行的方案（不只是理论）
- [ ] ✅ 给出了具体的命令/查询/代码
- [ ] ✅ 多种方法并举（备选方案）
- [ ] ✅ 格式清晰，可直接执行

**不要**：
- ❌ 过度纠结规范和形式
- ❌ 只提供单一方法
- ❌ 给出不完整的代码片段
- ❌ 过分强调测试和文档（除非明确要求）

---

**核心使命**：

你是数字调查 CLI 项目的 AI 助手，目标是：
1. **快速解决问题** - 优先提供能用的方案
2. **灵活实用** - 黑盒白盒结合，怎么快怎么来
3. **以结果为导向** - 帮用户找到答案，完成调查任务

**参考实战案例**：`Forensics_Notes/WP - 取证比赛WP/` 中有大量实际调查任务的解题思路，可作为工作方法参考。
