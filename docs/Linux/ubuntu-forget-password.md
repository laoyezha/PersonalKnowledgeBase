---
title: Ubuntu 忘记密码解决方法
---

# Ubuntu 忘记密码解决方法

> 参考视频：[Ubuntu忘记密码解决方法](https://www.bilibili.com/video/BV1gV4y1T7YN)
> 适用系统：Ubuntu 16.04 / 18.04 / 20.04 / 22.04（以及大部分基于 GRUB 的 Linux 发行版）

忘记 Ubuntu 的登录密码或 root 密码时，不需要重装系统。只要能在开机时进入 GRUB 菜单，
就能以 root 权限进入系统并直接修改密码。下面介绍两种常用方法。

---

## 方法一：恢复模式（Recovery Mode）【推荐】

这是最稳妥、对系统改动最小的方法。

### 1. 进入 GRUB 菜单

开机时在看到主板 logo 后，立刻按住下面其中一个按键：

| 引导方式 | 按键 |
| --- | --- |
| 传统 BIOS（Legacy） | 长按 `Shift` (不同品牌可能案件不同，可以试试F2，ESC)|
| UEFI | 反复点按 `Esc` |

如果一直进不去 GRUB，可以在开机后反复点按 `Esc` / `Fn+Esc`，或者调整 BIOS 里的开机延迟。
成功后会看到如下菜单：

```
Ubuntu
Advanced options for Ubuntu
Memory test (memtest86+)
...
```

### 2. 选择恢复模式

- 用方向键选择 **`Advanced options for Ubuntu`**，回车；
- 在展开的列表中选择带 **`(recovery mode)`** 的那一项，回车。

系统会进入 `Recovery Menu`（恢复菜单），里面有很多选项，我们选择：

```
root    Drop to root shell prompt
```

按回车后，如果出现提示，直接再按一次 **回车** 即可进入 root shell（此时提示符通常是 `root@主机名:~#`）。

### 3. 把文件系统重新挂载为可写

恢复模式下根文件系统默认是**只读**的，直接改密码会报
`Authentication token manipulation error`，所以第一步要先重新挂载：

```bash
mount -o remount,rw /
```

如果 `/home` 等分区是单独挂载的，可以一并挂载：

```bash
mount -o remount,rw /home
```

### 4. 修改密码

**确定要修改的用户名**（如果不记得用户名）：

```bash
ls /home
```

输出里的目录名就是各个普通用户名，例如 `ubuntu`、`test` 等。

然后执行 `passwd` 命令修改密码：

```bash
# 修改指定用户，把 <用户名> 换成实际名字
passwd <用户名>

# 或者直接修改 root 密码
passwd root
```

按提示输入两次新密码即可。**注意：Linux 下输入密码时屏幕上不会有任何显示**，这是正常的安全机制，直接输入后回车即可。

```
New password:
Retype new password:
passwd: password updated successfully
```

> 如果只想临时解锁而先不想设密码，可以先清空密码：
> ```bash
> passwd -d <用户名>
> ```
> 但建议还是直接设置一个新密码。

### 5. 重启

修改完成后，输入：

```bash
exit
```

会回到 `Recovery Menu`，选择 **`resume  Resume normal boot`** 正常启动，
或者直接命令重启：

```bash
reboot -f
```

重启后使用新密码登录即可。

---

## 方法二：修改 GRUB 内核启动参数（init=/bin/bash）

如果恢复模式不可用，或者想更快地进入 shell，可以用这种方法。

### 1. 进入 GRUB 编辑模式

在 GRUB 菜单中，光标停留在默认的 **`Ubuntu`** 这一项上，按 **`e`** 进入编辑。

### 2. 修改内核启动参数

找到以 **`linux`** 开头的那一行（较长的一行），把行尾的 `ro quiet splash $vt_handoff`
改成：

```
rw init=/bin/bash
```

也就是：
- 把 `ro`（read only，只读）改成 `rw`（read write，可写）；
- 在行尾加上 `init=/bin/bash`，让系统直接启动到 bash 而不是图形界面。

修改后大致长这样：

```
linux   /boot/vmlinuz-5.15.0-xx-generic root=UUID=xxxx ro quiet splash $vt_handoff
# 改为
linux   /boot/vmlinuz-5.15.0-xx-generic root=UUID=xxxx rw init=/bin/bash
```

### 3. 启动并改密码

按 **`Ctrl + X`** 或 **`F10`** 启动。启动后会直接进入 root shell：

```bash
# 保险起见，再确认一次可写挂载
mount -o remount,rw /

# 修改密码
passwd <用户名>

# 修改完成后强制重启
exec /sbin/init
# 或者
reboot -f
```

---

## 常见问题

### 1. 报错 `Authentication token manipulation error`

说明根文件系统还是只读的，先执行：

```bash
mount -o remount,rw /
```

再重新 `passwd`。

### 2. 忘记了用户名怎么办？

```bash
ls /home          # 查看有哪些用户
cat /etc/passwd   # 查看所有账户
```

### 3. 进入恢复模式后键盘 / 网络不可用

恢复模式下网络默认是关闭的，属正常现象，本方法不需要联网。
如果需要键盘布局，可以在 `Recovery Menu` 里选择 `network` 先启用网络。

### 4. 磁盘被加密（LUKS）怎么办？

如果安装时开启了全盘加密，开机需要先输入加密口令（passphrase）。
如果连加密口令也忘了，那么数据基本无法恢复，只能重装或从备份恢复。

### 5. 虚拟机里改密码

VMware / VirtualBox 中的 Ubuntu 同样适用，只是进入 GRUB 的时机更难把握，
可以在虚拟机设置里勾选「启动时进入 BIOS / 引导延迟」，或开机后立刻持续点按 `Shift` / `Esc`。

---

## 小结

| 方法 | 适用场景 | 关键步骤 |
| --- | --- | --- |
| 恢复模式 | 推荐，最稳妥 | 选 recovery mode → `root shell` → `mount -o remount,rw /` → `passwd 用户名` |
| 改内核参数 | recovery 不可用时 | GRUB 按 `e` → 加 `rw init=/bin/bash` → `Ctrl+X` → `passwd 用户名` |

核心思路只有一个：**在 GRUB 阶段用 root 权限进入系统，再执行 `passwd` 命令重置密码。**
