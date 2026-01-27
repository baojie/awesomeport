# Ubuntu 上 GitHub SSH 被 systemd 劫持的完整诊断复盘

> **关键词**：GitHub / SSH / systemd-ssh-proxy / 198.18.0.0/15 / port 22 / port 443
>
> **一句话结论**：
> 在部分 Ubuntu 系统中，`systemd-ssh-proxy` 会强制劫持 SSH 连接，将 GitHub 的 SSH 请求重写到 `198.18.x.x:22`，导致 GitHub SSH 永远失败。最终解决方案是 **强制 Git 使用 SSH 443 端口**。

> **说明**：本文中的 `john` 和 `REPO` 均为化名，用于保护真实用户和项目信息。

---

## 一、问题背景

在一台 Ubuntu 机器上，对 GitHub 仓库执行常规操作：

```bash
git pull
git push
```

却始终失败，错误信息为：

```text
Connection closed by 198.18.0.xxx port 22
fatal: Could not read from remote repository
```

但与此同时：

- 浏览器可以正常访问 GitHub
- 更换为手机 Wi‑Fi 仍然失败
- SSH key 已确认配置正确

这说明：**问题不在 GitHub、不在权限、不在网络本身**。

---

## 二、第一阶段：确认 SSH 行为是否异常

### 1. 使用详细模式观察 SSH 连接

```bash
ssh -vvv git@github.com
```

关键输出：

```text
Including file /etc/ssh/ssh_config.d/20-systemd-ssh-proxy.conf
Connecting to github.com [198.18.0.155] port 22
```

### 2. 关键异常点

- `198.18.0.0/15` 是 **RFC 定义的测试网段**，不是 GitHub
- SSH **无视用户配置**，始终走 22 端口
- 系统层注入了 `systemd-ssh-proxy`

➡️ **结论**：SSH 流量在本机就被 systemd 劫持了。

---

## 三、验证假设：SSH 443 是否可用？

GitHub 官方支持 **SSH over HTTPS (443)**。

### 手动直连测试：

```bash
ssh -T -p 443 git@ssh.github.com
```

返回：

```text
Hi john! You've successfully authenticated
```

✅ 证明：

- SSH key 没问题
- GitHub 没问题
- 网络没问题

---

## 四、为什么 Git 还是失败？

即便将 Git remote 改为：

```text
git@ssh.github.com:john/REPO.git
```

执行：

```bash
git pull
```

仍然失败。

### 再次观察 Git 调用 SSH 的真实行为

```bash
GIT_SSH_COMMAND="ssh -vvv" git pull
```

关键输出：

```text
resolving "ssh.github.com" port 22
Connecting to ssh.github.com [198.18.0.191] port 22
```

➡️ **systemd-ssh-proxy 已经进入"完全接管模式"**：

- 不管 host 是什么
- 不管 SSH config
- 一律重写为 `198.18.x.x:22`

---

## 五、最终解法（关键转折点）

### 解法核心思想

> **不要试图说服 systemd，直接绕过它。**

Git 提供了一个隐藏但非常强大的配置项：`core.sshCommand`。

---

## 六、最终解决方案

### 1️⃣ 临时验证（立即生效）

```bash
GIT_SSH_COMMAND="ssh -p 443" git pull
```

结果：

```text
当前分支 main 是最新的。
```

🎯 成功。

---

### 2️⃣ 永久修复（推荐）

```bash
git config --global core.sshCommand "ssh -p 443"
```

之后：

```bash
git pull
git push
```

全部正常。

Git push 成功日志：

```text
To ssh.github.com:john/REPO.git
   xxxx..yyyy  main -> main
```

---

## 七、为什么这是"最优解"？

- ✅ 不修改系统文件
- ✅ 不依赖 DNS / systemd 行为
- ✅ 对所有 GitHub SSH 仓库生效
- ✅ 适合公司网络 / VPN / 内网 / 云主机
- ✅ 完全可逆

相比之下：

- 禁用 `systemd-ssh-proxy`：侵入性更强
- 改 hosts / DNS：不可控
- 切 HTTPS：需要 PAT，体验差

---

## 八、经验总结（给未来的自己）

### 当你看到这些症状时：

- GitHub 浏览器正常
- SSH 永远连不上
- IP 是 `198.18.x.x`
- 日志里出现 `systemd-ssh-proxy`

👉 **直接跳到结论**：

```bash
git config --global core.sshCommand "ssh -p 443"
```

无需再 debug 30 分钟。

---

## 九、附录：一行速查卡片

```bash
ssh -T -p 443 git@ssh.github.com
git remote set-url origin git@ssh.github.com:john/REPO.git
git config --global core.sshCommand "ssh -p 443"
```

---

**这不是使用者的问题，而是 systemd 的"智能代理"在开发者场景下的失败设计。**

如果你在 Ubuntu / WSL / 云主机环境里工作，这个结论值得长期记住。
