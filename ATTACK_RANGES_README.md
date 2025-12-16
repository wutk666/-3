# 多攻击类型检测系统 - 使用指南

## 系统概述

这是一个全面的 Web 安全攻击测试平台，支持多种攻击类型的检测、防御和演示。系统采用分类架构，将攻击类型分为三大类别。

## 攻击分类体系

### 💉 注入类攻击 (Injection Attacks)
通过注入恶意代码来操纵应用程序行为

#### 1. XSS 跨站脚本攻击
- **反射型 XSS**: URL 参数直接回显到页面
- **存储型 XSS**: 恶意脚本保存到数据库
- **DOM型 XSS**: 纯客户端 JavaScript 操作 DOM
- **严重程度**: HIGH
- **测试页面**: `/test_xss`

#### 2. SQL 注入攻击
- **经典注入**: `OR 1=1`, `AND 1=2`
- **注释绕过**: `admin' --`, `admin' #`
- **UNION 注入**: 联合查询获取数据
- **堆叠查询**: 执行多条SQL语句
- **时间盲注**: `SLEEP()`, `BENCHMARK()`
- **布尔盲注**: 基于真假判断
- **严重程度**: CRITICAL
- **测试页面**: `/test_sqli`
- **靶场沙箱**: `/attack/sqli/sandbox`

#### 3. 命令注入攻击
- **分号注入**: `; command`
- **管道符注入**: `| command`
- **逻辑运算符**: `&& command`, `|| command`
- **反引号执行**: `` `command` ``
- **命令替换**: `$(command)`
- **重定向注入**: `> file`, `< file`
- **严重程度**: CRITICAL
- **测试页面**: `/test_cmdi`

### 🚫 访问控制攻击 (Access Control)
试图访问未授权的资源或文件

#### 4. 目录遍历攻击
- **相对路径遍历**: `../` 或 `..\`
- **绝对路径访问**: `/etc/passwd`
- **URL编码绕过**: `%2e%2e%2f`
- **双重编码**: `%252e%252e%252f`
- **混合编码**: `..%2f`
- **Unicode编码**: `..%c0%af`
- **严重程度**: HIGH
- **测试页面**: `/test_path_traversal`

#### 5. 权限提升 (开发中)
- 垂直权限提升
- 水平权限提升

#### 6. 文件上传 (开发中)
- 恶意文件上传
- WebShell 测试

### ⏱️ 行为分析攻击 (Behavioral Analysis)
通过异常行为模式进行的攻击

#### 7. 暴力破解 (开发中)
- 密码爆破
- 频率限制测试

#### 8. DoS 攻击 (开发中)
- 拒绝服务
- 资源耗尽测试

#### 9. 爬虫检测 (开发中)
- 恶意爬虫
- Bot 检测

## 数据库模型

### AttackLog (攻击日志)
```python
- id: 主键
- ip: 攻击者IP
- payload: 攻击载荷
- timestamp: 时间戳
- blocked: 是否被拦截
- attack_type: 攻击类型 (xss, sqli, cmdi, path_traversal, etc.)
- attack_category: 攻击分类 (injection, access_control, behavioral)
- severity: 严重程度 (low, medium, high, critical)
- target_url: 目标URL
- user_agent: User-Agent信息
```

### VulnerableUser (SQL注入靶场用户)
```python
- id: 主键
- username: 用户名
- email: 邮箱
- password: 密码（明文，仅用于演示）
- role: 角色 (admin, user, moderator)
- created_at: 创建时间
```

### VulnerableFile (目录遍历靶场文件)
```python
- id: 主键
- filename: 文件名
- filepath: 文件路径
- content: 文件内容
- is_sensitive: 是否敏感文件
- created_at: 创建时间
```

### RateLimitLog (频率限制日志)
```python
- id: 主键
- ip: IP地址
- endpoint: 访问端点
- timestamp: 时间戳
- request_count: 请求次数
```

## 安装和启动

### 1. 安装依赖
```bash
pip install -r requirements.txt
```

### 2. 初始化数据库
```bash
python init_attack_ranges.py
```

### 3. 启动应用
```bash
python run.py
```

### 4. 访问系统
- 登录页面: http://localhost:5000/login
- 控制台: http://localhost:5000/console
- 攻击测试中心: http://localhost:5000/attack_hub

## 使用指南

### 测试 SQL 注入

1. 访问 `/test_sqli`
2. 在搜索框输入测试 payload，例如：
   - `admin' OR '1'='1`
   - `admin' --`
   - `' UNION SELECT username,password,email FROM vulnerable_user --`
3. 点击"执行查询"查看结果
4. 系统会显示是否检测到攻击并拦截
5. 点击"查看靶场"可在隔离环境中查看完整数据库

### 测试命令注入

1. 访问 `/test_cmdi`
2. 在目标主机输入框输入测试 payload，例如：
   - `127.0.0.1; cat /etc/passwd`
   - `127.0.0.1 && ls -la`
   - `127.0.0.1 | whoami`
3. 点击"执行 Ping"查看结果
4. 系统会显示是否检测到命令注入并拦截

### 测试目录遍历

1. 访问 `/test_path_traversal`
2. 在文件路径输入框输入测试 payload，例如：
   - `../../../etc/passwd`
   - `../../etc/shadow`
   - `%2e%2e%2f%2e%2e%2fetc%2fpasswd`
3. 点击"查看文件"查看结果
4. 系统会显示是否检测到路径遍历并拦截

## 检测器架构

### AttackDetectorManager
统一管理所有攻击检测器

```python
detector_manager = AttackDetectorManager(app, db)
detection = detector_manager.detect_all(content)
```

### 单个检测器
- `XSSDetector`: XSS 攻击检测
- `SQLInjectionDetector`: SQL 注入检测
- `CommandInjectionDetector`: 命令注入检测
- `PathTraversalDetector`: 目录遍历检测
- `RateLimitDetector`: 频率限制检测

### 检测结果格式
```python
{
    'detected': bool,
    'attacks': [
        {
            'type': str,           # 攻击类型
            'category': str,       # 攻击分类
            'severity': str,       # 严重程度
            'description': str     # 攻击描述
        }
    ]
}
```

## API 端点

### 攻击测试 API
- `POST /attack/sqli/search` - SQL注入搜索测试
- `POST /attack/sqli/reset` - 重置SQL注入靶场
- `GET /attack/sqli/database` - 获取完整数据库
- `POST /attack/cmdi/ping` - 命令注入Ping测试
- `POST /attack/path/view` - 目录遍历文件查看

### 统计 API
- `GET /api/attack/stats` - 攻击统计信息
- `GET /api/stats/attacks` - 攻击趋势
- `GET /api/stats/types` - 攻击类型分布
- `GET /api/stats/top_ips` - Top攻击IP

## 扩展开发

### 添加新的攻击类型

1. **创建检测器类** (`app/attack_detectors.py`)
```python
class NewAttackDetector(AttackDetector):
    def __init__(self):
        super().__init__()
        self.attack_type = "new_attack"
        self.attack_category = "category"
        self.severity = "high"
        self.patterns = [
            (r'pattern1', 'description1'),
            (r'pattern2', 'description2'),
        ]
    
    def detect(self, content: str) -> Tuple[bool, Optional[str]]:
        # 实现检测逻辑
        pass
```

2. **注册到管理器**
```python
self.detectors['new_attack'] = NewAttackDetector()
```

3. **创建测试页面** (`app/templates/test_new_attack.html`)

4. **添加路由** (`app/routes.py`)
```python
@app.route('/test_new_attack')
def test_new_attack():
    return render_template('test_new_attack.html')

@app.route('/attack/new_attack/test', methods=['POST'])
def new_attack_test():
    # 实现测试逻辑
    pass
```

## 安全注意事项

⚠️ **重要警告**：
1. 本系统仅用于教育和安全研究目的
2. 所有靶场数据都是模拟的，不包含真实敏感信息
3. 请勿在生产环境中使用
4. 请勿用于非法攻击测试
5. 使用前请确保已获得适当授权

## 技术栈

- **后端**: Flask, SQLAlchemy
- **前端**: Bootstrap 5, Bootstrap Icons
- **数据库**: SQLite (可切换到其他数据库)
- **检测**: 正则表达式匹配
- **日志**: 完整的攻击日志记录

## 未来计划

- [ ] 实现暴力破解检测
- [ ] 实现 DoS 攻击检测
- [ ] 添加文件上传漏洞测试
- [ ] 添加权限提升测试
- [ ] 实现 CSRF 攻击测试
- [ ] 添加 XXE 注入测试
- [ ] 实现 SSRF 攻击测试
- [ ] 添加反序列化漏洞测试
- [ ] 机器学习增强检测
- [ ] 实时攻击可视化

## 贡献指南

欢迎提交 Issue 和 Pull Request！

## 许可证

本项目仅用于教育目的。
