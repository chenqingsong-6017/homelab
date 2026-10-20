# homelab

我的 DevOps 动手实验室（My hands-on DevOps lab）。本仓库是我五格学习路线第 3 格「容器」（Grid 3: Containers），通往全球运维职业，代号 Plan B。

## Goal（目标）
通过动手学容器：引擎 → 镜像 → Dockerfile → compose → 一个陌生人一条命令（`docker compose up`）就能跑起来的 demo。
Learn containers by doing: engine → images → Dockerfile → compose → a demo anyone can run with one command (`docker compose up`).

## Environment（环境）
- 操作系统（OS）：CentOS Stream 9（VirtualBox 虚拟机 VM）
- Docker 引擎社区版（Docker Engine CE）29.9.1、containerd 2.4.2、runc 1.5.2

## Progress（进度）
- [x] 第 1 单元（Unit 1）—— Docker 引擎基础：装好 docker-ce、跑通 hello-world、搞懂镜像与容器区别。
- [ ] 第 2 单元（Unit 2）—— 镜像与容器生命周期（image & container lifecycle）
- [ ] 第 3 单元（Unit 3）—— 第一个 Dockerfile
- [ ] 第 4 单元（Unit 4）—— docker compose
- [ ] 第 5 单元（Unit 5）—— demo 项目（一条命令 docker compose up）

## Known Limitations（已知局限）
- Docker Hub 在我所在地区被墙；目前靠镜像源 daocloud 拉镜像；`daemon.json` 镜像加速还没配，在待办里。
- 本 README 还在写，学到更多设计取舍会再补。
