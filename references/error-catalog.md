# 错误目录 — 常见报错诊断与修复速查

遇到报错时，按错误关键词在下表中定位类型，再执行对应修复。修复后必须重新运行命令验证，不可只改不验。

## 目录

1. [Python 相关错误](#1-python-相关错误)
2. [Node.js / npm 相关错误](#2-nodejs--npm-相关错误)
3. [文件与路径错误](#3-文件与路径错误)
4. [权限与沙箱错误](#4-权限与沙箱错误)
5. [网络与下载错误](#5-网络与下载错误)
6. [端口与进程错误](#6-端口与进程错误)
7. [命令行工具错误](#7-命令行工具错误)
8. [Windows 特定错误](#8-windows-特定错误)
9. [通用修复策略](#9-通用修复策略)

---

## 1. Python 相关错误

| 错误关键词 | 诊断 | 修复方案 |
|---|---|---|
| `ModuleNotFoundError: No module named 'xxx'` | 缺少 Python 包 | `pip install xxx`；若是项目内模块，确认在项目根目录运行 |
| `ImportError: DLL load failed` | 二进制依赖缺失或版本不兼容 | 检查 Python 位数与包匹配；`pip install --upgrade xxx`；必要时装 Visual C++ Redistributable |
| `SyntaxError: invalid syntax` | 代码语法错误 | 读取出错行号，检查缩进、括号、冒号、Python 版本兼容性 |
| `IndentationError` | 缩进不一致 | 统一用 4 空格或 Tab，不要混用 |
| `PermissionError: [Errno 13]` | 写入位置无权限 | 检查目标路径；Windows 下 `Program Files` 等目录需管理员权限 |
| `UnicodeDecodeError` | 读文件编码不对 | 打开文件时指定 `encoding='utf-8'` 或 `'gbk'`（中文 Windows 常见） |
| `MemoryError` | 内存不足 | 分批处理数据；减少并行度；用生成器替代列表 |
| `RecursionError` | 递归过深 | 改迭代；或 `sys.setrecursionlimit()` 谨慎提高 |
| `pip` 安装慢/超时 | 网络问题 | 换清华/阿里镜像：`pip install xxx -i https://pypi.tuna.tsinghua.edu.cn/simple` |

## 2. Node.js / npm 相关错误

| 错误关键词 | 诊断 | 修复方案 |
|---|---|---|
| `Cannot find module 'xxx'` | 未安装依赖 | `npm install`（在项目根目录）；单独装：`npm install xxx` |
| `EACCES` | npm 目录权限问题 | 不用 sudo 改 npm 全局目录；项目内安装优先 |
| `EADDRINUSE` | 端口被占用 | 见[端口错误](#6-端口与进程错误) |
| `node-gyp` 编译失败 | 缺少 C++ 编译工具 | Windows 装 windows-build-tools 或 Visual Studio Build Tools |
| `npm ERR! network` | 网络问题 | 换淘宝镜像：`npm config set registry https://registry.npmmirror.com` |
| `ERESOLVE` 依赖冲突 | 版本冲突 | 删 `node_modules` 和 `package-lock.json` 后重装；或 `npm install --legacy-peer-deps` |

## 3. 文件与路径错误

| 错误关键词 | 诊断 | 修复方案 |
|---|---|---|
| `FileNotFoundError` / `No such file or directory` | 文件不存在或路径错 | 先用 `ls`/`Glob` 确认实际路径；注意相对路径基于当前工作目录 |
| `IsADirectoryError` | 把目录当文件打开了 | 检查路径指向的是文件还是目录 |
| `NotADirectoryError` | 把文件当目录用了 | 同上 |
| 中文路径乱码 | 编码问题 | Windows PowerShell 读取文件时指定 `-Encoding utf8` |
| 路径中有空格 | 未加引号 | 所有含空格的路径用双引号包裹 |

**排查路径问题的标准动作：**
1. 用 `ls` 列出上级目录，确认文件实际叫什么。
2. 用 `pwd`（或 PowerShell `Get-Location`）确认当前工作目录。
3. 相对路径基于工作目录，不是脚本所在目录。

## 4. 权限与沙箱错误

| 错误现象 | 诊断 | 修复方案 |
|---|---|---|
| `Permission denied`（读/写） | 当前用户无权限 | 确认文件 ACL；若需系统目录写入，使用 `sandbox_permissions: require_escalated` |
| `Access is denied`（Windows） | 文件被占用或权限不足 | 检查文件是否被其他程序打开；检查是否需要管理员 |
| Bash 写入被拒但目录看似可写 | 沙箱限制 | 检查是否在授权目录内；必要时用 `require_escalated` |
| `.sh` 脚本无执行权限 | Unix 风格权限 | Windows 下直接 `bash script.sh`，不需要 chmod |
| 环境变量读不到 | 未设置或作用域不对 | 用 `echo $VAR`（Linux）或 `$env:VAR`（PowerShell）确认 |

## 5. 网络与下载错误

| 错误关键词 | 诊断 | 修复方案 |
|---|---|---|
| `ETIMEDOUT` / `Connection timed out` | 网络超时 | 重试 1 次；仍失败则检查网络/代理 |
| `Connection refused` | 目标端口未开放或服务未启动 | 确认服务是否在运行；检查地址和端口 |
| `SSL: CERTIFICATE_VERIFY_FAILED` | 证书校验失败 | 公司代理/自签名证书环境；谨慎使用 `-k`（curl）或 `verify=False`（Python）作为临时方案 |
| `403 Forbidden` | 无权限访问 | 检查是否需要认证/token/登录态 |
| `404 Not Found` | URL 错误或资源不存在 | 核对 URL；检查拼写和大小写 |
| `ProxyError` / 代理相关 | 代理配置问题 | 检查 `HTTP_PROXY`/`HTTPS_PROXY` 环境变量 |
| 下载速度极慢 | 镜像源问题 | 换国内镜像源（pip/npm/git 均有对应镜像） |

## 6. 端口与进程错误

| 错误关键词 | 诊断 | 修复方案 |
|---|---|---|
| `EADDRINUSE` / `Address already in use` | 端口被占 | Windows: `netstat -ano \| findstr :PORT` 找 PID，再 `taskkill /PID <pid> /F`；或换一个端口 |
| `port is already allocated`（Docker） | 容器端口冲突 | 停掉旧容器或换宿主端口映射 |
| `connection refused` 连本地服务 | 服务未启动或端口错 | 先确认服务真的在跑；再核对端口号 |
| 进程不退出 | 子进程挂住 | 用 TaskStop 终止后台任务；或 `taskkill /F /T /PID <pid>` 杀进程树 |

## 7. 命令行工具错误

| 错误关键词 | 诊断 | 修复方案 |
|---|---|---|
| `command not found` / `not recognized` | 工具未安装或不在 PATH | 先确认是否已安装；安装后重启 shell；或用完整路径 |
| `unknown option` / `invalid option` | 命令参数写错 | 查看 `--help` 确认正确参数名和格式 |
| `too many arguments` | 参数传多了 | 检查引号是否配对、路径是否含空格未转义 |
| `exit code 1`（通用） | 命令失败 | 读 stderr 找具体原因；退出码本身不含信息 |
| Bash timeout | 命令跑太久 | 改后台运行；或优化命令（加 `--quiet`、分批处理） |

## 8. Windows 特定错误

| 错误现象 | 诊断 | 修复方案 |
|---|---|---|
| 中文输出乱码 | 编码不是 UTF-8 | PowerShell 中执行 `chcp 65001`；读文件指定 UTF-8 |
| 执行策略限制 | PowerShell 脚本被禁 | 对当前会话：`Set-ExecutionPolicy -Scope Process Bypass` |
| `bash: command not found` 但 Windows 有 | 环境是 PowerShell 不是 bash | 用 PowerShell 语法；或显式调用 `bash -c "..."` |
| 路径混用斜杠 | Windows 接受 `/` 和 `\`，但混用偶尔出问题 | 统一用 `\`（PowerShell）或在引号内使用 `/` |
| 长路径问题 | Windows 260 字符限制 | 启用长路径支持；或缩短路径 |

## 9. 通用修复策略

当错误类型不明确时，按以下顺序排查：

1. **看最后一行**——大多数错误信息的根因在最后几行，不要从头找。
2. **复制关键错误关键词去搜索**——如果不确定，用错误关键词在上下文中推理。
3. **最小化复现**——把复杂命令拆到最小可复现单元，排除干扰。
4. **检查最近改动**——刚改了什么导致的？回滚那一步试试。
5. **换工具/路径**——同一功能换一种方式实现（如用 Python 替代 bash 解析）。
6. **清理重来**——删 `node_modules` / `__pycache__` / 临时文件后重装重跑（仅当确认是缓存损坏时）。

**不要做的事：**
- 不要在未读错误信息前就重试。
- 不要连续两次用完全相同的命令重试。
- 不要为了绕过错误而禁用安全检查（如 SSL 校验），除非明确是测试环境。
- 不要忽略 warning——很多 warning 是后续报错的前兆。
