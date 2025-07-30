
## 环境准备
- 操作系统: macOS 10.15+、Ubuntu 20.04+/Debian 10+，windows 或WSL的windows
- 内存:  最少 4GB RAM
- 软件: Node.js 18+

##  claude 
我这里用的WSL的windows安装的ubuntu24 ，打开windows终端 ，进入ubuntu系统，安装Claude Code CLI

![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20250730224500.png)

```
sudo apt update
sudo apt install nodejs npm -y
sudo npm install -g @anthropic-ai/claude-code
或者
sudo npm config set registry https://registry.npmmirror.com && sudo npm install -g @anthropic-ai/claude-code 
```
##  kimi 
我们先去Moonshot AI - 开放平台，进行注册，默认会送15元额度
https://platform.moonshot.cn/console/api-keys

## claude接入kimi2
首先我们先选择新建一个目录作为项目目录 ，重点强调下，下面安装我们会遇到429的问题，因为默认并发是1，并发太小
所以需要冲 50 块钱 ，增加并发才可以使用 

```shell 
# mkdir -pv /mnt/d/claude-projects  && cd /mnt/d/claude-projects 
# export export  ANTHROPIC_AUTH_TOKEN=sk-HKKxsg2zxxxxxxxxxpxqRhAyawVYvwk1Tjr084H9E
# export ANTHROPIC_BASE_URL=https://api.moonshot.cn/anthropic/

root@bdser-home:/mnt/d/claude-projects# claude
╭──────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                      │
│ Detected a custom API key in your environment                                        │
│                                                                                      │
│ ANTHROPIC_API_KEY: sk-ant-...hAyawVYvwk1Tjr084H9E                                    │
│                                                                                      │
│ Do you want to use this API key?                                                     │
│                                                                                      │
│ ❯ 1. Yes                                                                             │
│   2. No (recommended)✔                                                               │
│                                                                                      │
╰──────────────────────────────────────────────────────────────────────────────────────╯
   Enter to confirm ․ Esc to cancel
╭───────────────────────────────────────────────────╮
│ ✻ Welcome to Claude Code!                         │
│                                                   │
│   /help for help, /status for your current setup  │
│                                                   │
│   cwd: /mnt/d/claude-projects                     │
│                                                   │
│   ─────────────────────────────────────────────── │
│                                                   │
│   Overrides (via env):                            │
│                                                   │
│   • API Key: sk-ant-…1myHpxqRhAyawVYvwk1Tjr084H9E │
│   • API Base URL:                                 │
│   https://api.moonshot.cn/anthropic/              │
╰───────────────────────────────────────────────────╯
 ※ Tip: Send messages to Claude while it works to steer Claude in real-time

> 请问你是谁，你能为我做什么？

● 我是Claude Code，Anthropic官方的CLI工具。我可以帮助你完成各种软件开发任务，包括：

  - 编写和修改代码
  - 调试和修复bug
  - 代码重构和优化
  - 运行测试和构建项目
  - 分析代码库结构
  - 创建和管理git提交
  - 搜索和理解现有代码
  - 实现新功能

  请告诉我你需要什么帮助，我会直接为你完成。

╭──────────────────────────────────────────────────────────────────────────────────────╮
│ >                                                                                    │
╰──────────────────────────────────────────────────────────────────────────────────────╯
  -- INSERT --

```

并发和限速如图: 
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20250730224052.png)

## 配置环境变量
```shell
# cat << 'EOF' >> /etc/profile.d/claude.sh 
# claude code
export ANTHROPIC_BASE_URL=https://api.moonshot.cn/anthropic/
export ANTHROPIC_API_KEY=$YOUR_API_KEY
EOF
# source  /etc/profile.d/claude.sh 
```
## 客户端使用
- vscode 安装cline模块，这里不做演示
## 总结
相对比算是穷鬼套餐，体验还不错，首次注册送15，但是想要使用还需要冲50，资本家的游戏果然充满着诱惑和套路，大家自行参考吧~ 
