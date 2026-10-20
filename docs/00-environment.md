# 环境（Environment）

本文件记录 homelab 的机器环境与安装 Docker 引擎的全过程。**只记事实与为什么，不记操作步骤流水。**

## 机器（Machine）
- 操作系统（OS）：CentOS Stream 9（运行在 VirtualBox 虚拟机 VM 里）
- 宿主机：Ryzen 7 5700X / 32GB
- Docker 引擎社区版（Docker Engine CE）29.9.1
- containerd 2.4.2（容器运行时，dockerd 之下的"后厨总管"）
- runc 1.5.2（底层运行时，真正动手造容器的"厨师"）

> 镜像源选择（Why）：Docker Hub 在该地区受限（直连拉镜像报 `connection refused`），所以安装与拉镜像都走 **daocloud 镜像源**，不是直连官方。这是 2026-10-15 实测定下的，不是偏好。

## 装引擎过程（Install）
1. 加 Docker 官方仓库（经 daocloud 加速地址）
2. `dnf install -y docker-ce` → 装好 29.9.1
3. `systemctl enable --now docker` → 开机自启并启动
4. 验证：`docker version` 的 **Server 段**能出（含 containerd / runc 版本号，这是三层结构的实物证据，比 `ps` 更直接）；`docker run hello-world` 出 `Hello from Docker!`

## 已知坑（Known pitfalls）
- Docker Hub 被墙 → 拉镜像必须走镜像源或本地已有镜像（本次 hello-world 是 10-15 经 daocloud 拉进来的，名字挂着 `docker.m.daocloud.io/...` 长名）
- 加 docker 组后必须**重登**才生效（组员身份在登录那刻定死，详见 troubleshooting.md #3）

Key line (English): A container runtime has layers — the manager (dockerd) hands off to the supervisor (containerd), which calls the executor (runc). Knowing who calls whom is the difference between L1 and L2.
