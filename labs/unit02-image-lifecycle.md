# 单元2 · 镜像与容器生命周期（Unit 2: Image & Container Lifecycle）

> 证据文件（evidence），对应 README 里已勾的 `[x] Unit 2`。记录「分层」到底是什么、以及删容器为什么不动镜像。

## 两个概念（L1 自证）
1. **镜像（image）＝只读的模板，分层的**：像图书馆里的一本书，谁都能借来读，但没人能在书上写字。Dockerfile 里一条指令＝一层，层叠起来就是一个镜像。
2. **容器（container）＝镜像跑起来的实例，带一个可写层**：像读者借书后自己的笔记本——真正写东西写在本子上（可写层），书本身（镜像层）一点不动。同一个镜像跑两个容器＝两份各自独立的笔记本。
3. **关键差**：容器删掉，笔记本（可写层）没了，但书（镜像）还在架上。这就是「镜像层只读、容器层可写」的分层独立性。

## 预测闸（L2 原料）
- 问：删掉由某镜像启动的容器后，`docker images` 里那个镜像还在吗？
- 答：**在**。容器＝镜像 + 可写层；删容器只删可写层，镜像层不动。
- 为什么这题值钱：能把「删容器」的后果预测对，说明你分得清「模板」和「实例」——不是背命令，是懂结构。

## 实物验证（proof，非流水）
### A. 看分层：`docker history hello-world`
- 底部 `COPY hello /` 12.3kB ＝ 真文件层（把 hello 程序拷进镜像）
- 顶部 `CMD ["/hello"]` 0B ＝ 真配置层（规定跑起来执行什么）
- 结论：**指令一条＝一层**，层数在 `history` 里看得见。

### B. 生命周期闭环：删容器不动镜像
- 删前：`docker ps -a` 有两行 `Exited (0)`（unit01 跑 hello-world 留下的容器尸体）
- 删：`docker rm a987 4006`——按容器 ID 逐个删；docker 对不存在的 ID 只报 `No such container`，**不中断**、其余 ID 照删（本次 4006 照删成功）
- 删后：`docker ps -a` → **空表**，容器已全部清理
- 镜像：`docker images` → `hello-world` **仍在**，ID `11a3eb1597a8` → 镜像未被连带删除
- 结论：删了容器、镜像纹丝不动 ⇒ **分层独立**（容器可写层与镜像只读层分离）

> ✅ 已补（2026-10-22 单元3）：动态分层证据见 `unit03-dockerfile.md` §D —— `docker history myhello:v2` 三条层（`ADD alpine-minirootfs…tar.gz` 8.11MB / `CMD ["/bin/sh"]` 0B / `CMD ["echo" "hi"]` 0B），证实「改 CMD 不是改掉旧层，而是在它上面再压一层」。

## 关键结论（Key line）
Deleting a container removes only its writable layer — the image (read-only template) stays. One image, many containers; delete a container, the image is untouched. (删容器只删可写层；镜像是只读模板，一个镜像可跑多个容器，删容器不动镜像。)
