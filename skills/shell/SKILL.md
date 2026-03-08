# Shell 命令执行技能 (shell)

## 触发短语
- 执行命令/运行脚本/命令行/终端/bash/shell/跑一下
- run command/execute/terminal/系统命令

## 核心能力
通过 `exec` 工具在本机执行 Shell 命令，支持管道、环境变量、工作目录设定。

## 输入约定
- 优先接收结构化 JSON 输入，例如 `{"command":"ls -la /home","working_dir":"/tmp","timeout_secs":30}`
- `SKILL.rhai` 会优先从 `ctx.message` 解析 JSON
- 若不是 JSON，则回退为将 `user_input` 直接视为待执行命令

## 工具调用顺序

### 场景 1: 执行单条命令
1. `exec(command="命令", working_dir="/path")` — 直接执行
2. 展示 stdout/stderr 输出

### 场景 2: 运行脚本文件
1. `write_file(path="/tmp/script.sh", content="脚本内容")` — 写脚本
2. `exec(command="bash /tmp/script.sh")` — 执行

### 场景 3: 系统信息查询
- `exec(command="uname -a")` — 系统版本
- `exec(command="df -h")` — 磁盘使用
- `exec(command="free -h")` — 内存 (Linux)
- `exec(command="ps aux | head -20")` — 进程列表
- `exec(command="which 程序名")` — 查找程序

### 场景 4: 文件操作
- `exec(command="ls -la /path")` — 列目录
- `exec(command="find /path -name '*.log' -mtime -1")` — 查找文件
- `exec(command="cat /path/to/file")` — 查看文件

## 安全规则
- **危险命令**（rm -rf / dd if= 等）必须向用户二次确认
- 不在系统关键目录（/etc /sys /boot）执行写操作
- 长时间运行的命令设置 timeout（默认 30s，可调）
- 输出超过 1000 行时只展示前后各 50 行

## 输出格式
```
📟 执行命令: `{command}`
工作目录: {working_dir}

**输出**:
```
{stdout}
```

{如有错误输出}
**错误**:
```
{stderr}
```
退出码: {exit_code}
```

## 降级策略
1. 命令不存在 → 提示安装方法（brew install / apt-get install）
2. 权限不足 → 提示用 sudo（需用户确认）
3. 超时 → 建议后台运行或拆分命令
