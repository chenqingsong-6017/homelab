# 单元1 · Docker 引擎上手（Unit 1: Docker Engine）

> 证据文件（evidence），对应 README 里已勾的 `[x] Unit 1`。本文件是"做过"的证明，不是声明。

## 学完能答的三件事（L1 自证）
1. **容器解决什么问题** = 程序换个环境就跑得不一样，因为每台机器环境不同 → 容器把"程序 + 它要的环境"一起打包带走，到哪都能跑一样。
2. **容器 vs 虚拟机** = 容器共享宿主机内核，省掉一整个操作系统 → 启动快、占用小；虚拟机是自带完整 OS 的"另一台电脑"。
3. **引擎为什么分三层** = dockerd（经理，接你命令）→ containerd（后厨总管，管容器的生老病死）→ runc（厨师，真正动手造容器）。分清"管"和"做"是这一格的地基。

## 预测闸（L2 原料）
- 问：containerd 挂了，`docker ps` / `docker run` 会怎样？
- 答：**全挂**。因为 dockerd 靠的是 containerd，不是直连 runc。runc 只是 containerd 手下的执行者。
- 为什么这题值钱：能说出"谁调用谁"，就到了 L2 门口——不是"会敲命令"，是"能预测故障范围"。

## 实物验证（proof，非流水）
- `docker version` Server 段直接打印 containerd v2.4.2 / runc 1.5.2 → 三层存在的实物证据
- 加 docker 组：`usermod -aG docker chen` + **重登** → chen 能跑通（不重登会 `Permission denied on docker.sock`）
- 绕墙：`docker run hello-world` 直连被墙 → 本地已有 `docker.m.daocloud.io/library/hello-world` → `docker tag` 加别名 `hello-world:latest` → 跑通
- 问题B 三诊断：`systemctl list-unit-files` 显示 docker.service **enabled**；`ps -ef` 两个进程 PPID 都是 1（systemd 起的）；`rpm -qa` 五个 docker 包齐全 → 结论：daemon 是 systemd 亲生的，重启自愈（非故障）
- 生命周期：`docker ps` 空表 + `docker ps -a` 两行 `Exited (0)` → 容器跑完即退，尸体保留可复活/清理

## 关键结论（Key line）
Image = recipe, Container = the cooked dish. One recipe, two runs = two dishes. (镜像＝菜谱，容器＝菜；一份菜谱跑两次＝两个独立容器。)
