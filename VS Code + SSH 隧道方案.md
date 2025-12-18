文档重点强调了SSH 隧道配置以及最后必须重载窗口的关键步骤。



# 局域网服务器 AI 开发环境配置指南 (VS Code + SSH 隧道方案)

## 1. 背景与目标
在内网/局域网服务器无法访问外网的情况下，通过 **SSH 反向隧道** 技术，将本地电脑（有网）的代理流量转发给服务器。

**实现效果**：
1. 服务器终端可使用 `curl`, `wget`, `git`, `pip`, `npm` 等命令访问外网。
2. VS Code 中的 AI 插件（Trae, CodeBuddy, Copilot 等）可在服务器环境正常联网工作。

## 2. 前置准备
* **本地电脑**：已安装 VS Code，已运行代理软件（如 V2Ray）。
* **远程服务器**：已开启 SSH 服务。

---

## 3. 配置步骤

### 第一步：确认本地代理端口
1. 打开本地电脑的代理软件（如 V2RayN / V2RayU）。
2. 查看底部或设置中的 **HTTP 端口**。
   * *假设本地 HTTP 端口为 `10809` (Windows 常见) 或 `1087` (Mac 常见)。下文以 `10809` 为例。*

### 第二步：配置 SSH 反向隧道 (本地电脑)
1. 在 VS Code 中按 `F1` (或 `Ctrl+Shift+P`)。
2. 输入并选择 `Remote-SSH: Open SSH Configuration File`。
3. 编辑对应的服务器配置 (`Host`)，添加 `RemoteForward` 指令：

```ssh
Host my-dev-server
    HostName 192.168.x.x
    User root
    # 核心配置：将服务器的 7890 端口流量转发到本地的 10809 端口
    # 格式：RemoteForward [服务器端口] 127.0.0.1:[本地代理端口]
    RemoteForward 7890 127.0.0.1:10809


(保存文件 Ctrl+S)
第三步：配置服务器环境变量 (服务器端)
为了让服务器的所有命令默认走代理，建议将配置写入 .bashrc。
连接服务器终端，编辑配置文件：
Bash
nano ~/.bashrc


在文件末尾添加以下内容：
Bash
# SSH Tunnel Proxy Settings
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
# 只有当 SSH 隧道建立时，上述代理才生效


保存退出 (Ctrl+X -> Y -> Enter)。
使配置立即生效（仅本次终端）：
Bash
source ~/.bashrc


4. 关键步骤：重启连接 (至关重要)
修改 SSH 配置文件 (config) 后，必须彻底重启 VS Code 的远程连接进程，否则隧道不会建立，服务器会报 Connection refused 错误。
操作方法：
保持 VS Code 界面。
按 F1 (或 Ctrl+Shift+P)。
输入并执行：Developer: Reload Window (开发人员: 重新加载窗口)。
或者：点击左下角绿色远程图标 -> 关闭远程连接 -> 重新点击连接。
5. 验证是否成功
连接成功后，在 VS Code 的服务器终端执行以下命令验证：
1. 检查端口监听

Bash


netstat -anp | grep 7890
# 成功输出示例：tcp  0  0  127.0.0.1:7890   0.0.0.0:* LISTEN   sshd: root


2. 测试网络连通性

Bash


curl -I https://www.google.com
curl -I https://www.baidu.com
# 成功输出示例：HTTP/1.1 200 OK (或 301 Moved Permanently)


6. 常见问题排查
报错 Connection refused：
原因：SSH 隧道未建立。
解决：请务必执行第 4 步的“Reload Window”操作。
报错 Empty reply from server：
原因：本地代理端口填错了（例如填成了 Socks 端口而不是 HTTP 端口）。
解决：检查 V2Ray 界面显示的 HTTP 端口号，修正 RemoteForward 配置。
AI 插件仍无法使用：
解决：在 VS Code 设置 (Remote) 中搜索 Proxy，手动填入 http://127.0.0.1:7890。
