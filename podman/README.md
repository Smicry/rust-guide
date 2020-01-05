# [podman](https://github.com/containers/libpod)
* `Podman` 原来是 `CRI-O` 项目的一部分，后来被分离成一个单独的项目叫 `libpod`。
* `Podman` 的使用体验和 `Docker` 类似，`Podman CLI` 里面 87% 的指令都和 `Docker CLI` 相同。

## 和docker区别
* `Docker CLI` 会通过 `gRPC API` 去跟 `Docker Engine` 说「我要启动一个容器」，然后 `Docker Engine` 才会通过 `OCI Container runtime`（默认是 `runc` ）来启动一个容器。这就意味着容器的进程不可能是 Docker CLI 的子进程，而是 Docker Engine 的子进程。
* 因为 `docke` 有 `docker daemon`，所以 `docker` 启动的容器支持 `--restart` 策略，但是 `podman` 不支持，如果在 `k8s` 中就不存在这个问题，我们可以设置 `pod` 的重启策略，在系统中我们可以采用编写systemd服务来完成自启动。
* `docker` 需要使用 `root` 用户来创建容器，但是 `podman` 不需要。
* `Podman` 不使用 `Daemon`，而是直接通过 `OCI runtime`（默认也是 `runc` ）来启动容器，所以容器的进程是 `podman` 的子进程。这比较像 `Linux` 的 `fork/exec` 模型，而 `Docker` 采用的是 `C/S`（客户端/服务器）模型。与 `C/S` 模型相比，`fork/exec` 模型有很多优势，比如：
    * 系统管理员可以知道某个容器进程到底是谁启动的。
    * 如果利用 cgroup 对 podman 做一些限制，那么所有创建的容器都会被限制。
    * SD_NOTIFY : 如果将 podman 命令放入 systemd 单元文件中，容器进程可以通过 podman 返回通知，表明服务已准备好接收任务。
    * socket 激活 : 可以将连接的 socket 从 systemd 传递到 podman，并传递到容器进程以便使用它们。

## 替代docke
* `bash` 中加入 `alias docker=podman` 完成替换

## 设置自启动
* `Systemd` 来实现 `Podman` 开机重启容器，建立一个 Systemd 服务配置文件：
```
$ vim /etc/systemd/system/nginx_podman.service

[Unit]
Description=Podman Nginx Service
After=network.target
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/podman start -a nginx
ExecStop=/usr/bin/podman stop -t 10 nginx
Restart=always

[Install]
WantedBy=multi-user.target
```
* 接下来，启用这个 Systemd 服务
```
$ sudo systemctl daemon-reload
$ sudo systemctl enable nginx_podman.service
$ sudo systemctl start nginx_podman.service
```

## 跟踪登录 UID
* 登录 `UID`（loginuid），存储在 `/proc/self/loginuid` 中，它是系统上每个进程的 proc 结构的一部分。该字段只能设置一次；设置后，内核将不允许任何进程重置它。当登录系统时，登录程序会为我的登录过程设置 `loginuid` 字段。从初始登录过程 `fork` 并 `exec` 的每个进程都会自动继承 `loginuid`。
### 对比
* `podman` 甚至容器进程也保留了 `loginuid`。
```
sudo podman run fedora cat /proc/self/loginuid
3267
```
*  但是在 `Docker` 中则不是。
```
sudo docker run fedora cat /proc/self/loginuid 
4294967295
```
### 原因
* 进程的默认 `loginuid`（在设置 `loginuid` 之前）是 `4294967295`（LCTT 译注：2^32 - 1）。由于容器是 `Docker` 守护程序的后代，而 `Docker` 守护程序是 `init` 系统的子代，所以，我们看到 `systemd`、`Docker` 守护程序和容器进程全部具有相同的 `loginuid`：4294967295，审计系统视其为未设置审计 UID。
```
cat /proc/1/loginuid 
4294967295
```
### 影响
* `Linux` 内核有一个有趣的安全功能，叫做审计。它允许管理员在系统上监视安全事件，并将它们记录到 `audit.log` 中，该文件可以本地存储或远程存储在另一台机器上，以防止黑客试图掩盖他的踪迹。
* `/etc/shadow` 文件是一个经常要监控的安全文件，因为向其添加记录可能允许攻击者获得对系统的访问权限。管理员想知道是否有任何进程修改了该文件，你可以通过执行以下命令来执行此操作：
```
# auditctl -w /etc/shadow
```
* 在 `Docker` 情形中，`auid` 是未设置的`unset`（`4294967295`）；这意味着安全人员可能知道有进程修改了 `/etc/shadow` 文件但身份丢失了。
如果该攻击者随后删除了 `Docker` 容器，那么在系统上谁修改 `/etc/shadow` 文件将没有任何跟踪信息。`Podman` 则不存在这个问题，可以正常记录。
