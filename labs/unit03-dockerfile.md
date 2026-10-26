# 单元3 · 第一个 Dockerfile（Unit 3: My First Dockerfile）

> 证据文件（evidence）。记录「怎么自己写一块镜像」：Dockerfile 是什么、FROM/CMD 各管什么、CMD 两种写法差在哪、以及层是怎么一层层叠上去的。
> **★ 实物＝同目录 `Dockerfile`**（2026-10-23 落成真文件；本笔记是它的解读与验证记录，**笔记不顶产出**）。

工作目录：`~/dockerlab`（`mkdir dockerlab && cd dockerlab`），文件名必须是 **`Dockerfile`**（大写 D、无后缀、结尾是 f-i-l-e）。

## 三个概念（L1 自证）

1. **Dockerfile ＝ 做镜像的说明书（菜谱）**：它不是命令，是**写进文件里的字**。文件名固定叫 `Dockerfile`，内容一行一条指令。`docker build` 才是把这份说明书变成镜像的动作。
2. **`FROM` ＝ 打地基**：指定从哪块基础镜像起步。**必须大写、必须是第一行**——没有基础镜像就没有文件系统，后面的指令根本无处落地。本次用 `alpine:3.20`：它展开才 8.11MB，对「只想跑一句 echo」来说，没必要背 Ubuntu 那几百 MB；同时下载快、攻击面小。
3. **`CMD` ＝ 规定启动后干什么**：它**在 build 时不执行，只在 `docker run` 那一刻执行**。证据：build 全程没有打印 `hi`，run 才打印。

## 预测闸（L2 原料）

- 问：`CMD echo hi` 和 `CMD ["echo", "hi"]`，`docker run` 看到的输出一样吗？
- 答：**一样，都是 hi**。差别不在输出，在镜像内部记下来的配置——前者被 Docker 改写成 `[/bin/sh -c echo hi]`（多套一层 shell），后者原样是 `[echo hi]`。
- 为什么这题值钱：能预测「改写法不影响输出」，说明你知道 CMD 存的是配置、不是执行结果。

## 实物验证（proof，非流水）

### A. 手打建文件：`cat > Dockerfile`
```
FROM docker.1ms.run/library/alpine:3.20
CMD ["echo", "hi"]
```
- `cat >` = 把后面敲的字写进文件；`Ctrl+D` 存盘退出。
- **踩坑**：第二行敲完不按回车就 `Ctrl+D`，文件最后一行会缺换行符，某些版本 Docker 会报解析错误。核实方法：`cat -A Dockerfile`（`-A` = show all，每行末尾会显示一个 `$`，没有 `$` 就是缺换行）。

### B. 构建与运行
- `docker build -t myhello .` → 首次成功，但底部挂一条警告：
  `JSONArgsRecommended: JSON arguments recommended for CMD to prevent unintended behavior related to OS signals (line 2)`
- `docker run myhello` → 打印 `hi` 后回到 `$`。**build 无 hi、run 出 hi ⇒ CMD 只在 run 时执行，成立。**

### C. 两种 CMD 写法的真身：`docker inspect`
```
docker inspect --format '{{.Config.Cmd}}' myhello:latest myhello:v2
```
输出两行：
```
[/bin/sh -c echo hi]     ← shell 形式：Docker 替你套了一层 /bin/sh -c
[echo hi]                ← exec 形式：原样保留，不插东西
```
**为什么 exec 形式更好**：shell 形式下，容器里的 1 号进程（PID 1）是 `sh`，`echo` 只是它的子进程；`sh` 不转发外来的停止信号，所以 `docker stop` 时信号卡在 `sh` 手上，要等 10 秒超时才被强杀。exec 形式下 `echo` 自己就是 PID 1，信号直达。
- 预测验证：把 CMD 换成 JSON 形式后重新 build，**那条 `JSONArgsRecommended` 警告消失了**（构建日志从带警告变成全蓝无警告）。

### D. 分层动态证据：`docker history myhello:v2`
```
IMAGE          CREATED      CREATED BY                                SIZE    COMMENT
96e6f399fa5f   6 months ago CMD ["echo" "hi"]                          0B      buildkit.dockerfile.v0
<missing>      6 months ago CMD ["/bin/sh"]                            0B      buildkit.dockerfile.v0
<missing>      6 months ago ADD alpine-minirootfs-3.20.10-x86_64.tar.gz 8.11MB buildkit.dockerfile.v0
```
三条结论：
1. **你的 CMD 并没有「改掉」alpine 自带的那条 `CMD ["/bin/sh"]`，而是在它上面又压了一层。** 旧的还在（history 里看得见），只是被盖住不生效——这就是「镜像只读、只往上叠加」的实物证据。
2. 两条 CMD 都是 **0B**：CMD 是配置、不是文件，不占空间。8.11MB 全在 `ADD`（alpine 根文件系统）那一层。
3. `<missing>` = 这一层没有独立镜像 ID（中间层）。只有最上面那行有 ID `96e6f399fa5f`，那才是 `myhello:v2` 自己的 ID。
- 注：`CREATED` 列全写 `6 months ago`，是从基础镜像继承的时间戳，不是本次构建时间，不影响结论。

> 这一条同时补上了 unit02 留的「改一行看层数变化」的动态证据（unit02 只有 hello-world 的静态层）。

### E. RUN 与 CMD 的执行时机（证人实验，10-23）

Dockerfile 加第三行 `RUN echo building now`，让它在 build 日志里当**证人**：

```
Step 1/3 : FROM alpine:3.20          ---> 49e053e87e55      （本地命中，无拉取＝不联网）
Step 2/3 : RUN echo building now      ---> Running in 462657f2f95d
building now                                                 ← RUN 真执行了
Step 3/3 : CMD ["echo","hi"]          ---> Running in c47595b347aa
                                                             ← 一个输出都没有＝CMD 不执行
Successfully built 3ab7686067c9
Successfully tagged myhello:v3
```

再 `docker run myhello:v3`：屏幕只打印 `hi`，**`building now` 不再出现**。

三条结论：
1. **`RUN` 在 build 时执行，结果固化成一层**（本次层 `912d0c6ec2c0`）；`CMD` 在 build 时只登记，不执行。
2. 老构建器对每条 RUN/CMD 都起一个**临时容器**（`Running in …`），跑完 `Removed intermediate container …` 删掉——镜像就是这么一层层砌出来的。
3. `docker run` 时被执行的**只有 CMD**：`hi` 出现、`building now` 消失。

## 环境墙实战（troubleshooting，真实踩坑）

- 现象：把 CMD 改成 JSON 形式后重跑 `docker build -t myhello:v2 .`，报
  `403 Forbidden … this image is not in the allowlist`（附 GitHub issue 链接）。
- 根因：`docker.m.daocloud.io` 这个免费镜像站收紧了白名单，`library/alpine` 被移出。同一条 FROM 在 16:21 还能成、16:39 就被拒 ⇒ **免费镜像站不稳定，随时可能失效**。
- 排查：`docker image ls` 一看，**本机根本没有 alpine 那一行**——BuildKit 只把 base image 当构建缓存用，没在本地留下可引用的镜像，所以「用本地缓存绕开网络」这条路不通。
- 处置：换一家拉取 `docker pull docker.1ms.run/library/alpine:3.20` → `Status: Downloaded newer image for …`（Digest `sha256:d9e853e8…`），再把 Dockerfile 的 FROM 域名改成 `docker.1ms.run`，重新 build 即成（0.2s、`CACHED`、无警告）。
- **第二次（10-23）**：`docker.1ms.run/library/alpine:3.20: not found`（前一天刚拉成功过）。根因查清：**FROM 带源站域名时，BuildKit 和老构建器都强制联网去问「这块镜像的 metadata」——本地明明有也要问，源站一挂构建就死**。连 `DOCKER_BUILDKIT=0` 也救不了，因为问题不在构建器、在「带域名」这件事本身。
- **绕行三步（10-23 已完成，全程不联网）**：
  1. `docker tag docker.1ms.run/library/alpine:3.20 alpine:3.20` —— 给本地镜像挂短名（ID 仍是 `d9e853e87e55`，同一块镜像两个名字）
  2. Dockerfile 改 `FROM alpine:3.20` —— 纯短名，不带域名
  3. `DOCKER_BUILDKIT=0 docker build -t myhello:v3 .` —— `Step 1/3` 直接 `---> 49e053e87e55` 命中本地，零网络请求
- **结论（以后写 Dockerfile 的默认写法）**：FROM 一律用本地已有短名；需要新基础镜像时先 `pull` → `tag` 挂短名 → 再改 FROM。
- 待办：把稳定源写进 `daemon.json` 做全局镜像加速（挂账，未做）。

## 命令小记（本次新出现）

| 命令 | 作用 |
|---|---|
| `cat > Dockerfile` | 手打创建文件；`Ctrl+D` 存盘 |
| `cat -A Dockerfile` | 显示行尾（`$`），查换行是否齐全 |
| `docker build -t <名字>:<标签> .` | 按 Dockerfile 做镜像；`.` = 当前目录 |
| `docker pull <镜像源>/library/<名>:<版本>` | 手动拉取基础镜像 |
| `docker inspect --format '{{.Config.Cmd}}' <镜像>` | 只取镜像配置里的 CMD 字段 |
| `docker history <镜像>` | 列出每一层（倒序，最上最新） |

## 关键结论（Key line）

A Dockerfile is a recipe, not a command: `FROM` picks the base image (first line), `CMD` says what runs at `docker run` — never at build time. Prefer the exec form `CMD ["echo","hi"]`: it keeps your process as PID 1 so stop signals reach it, instead of hiding behind a shell.
（Dockerfile 是菜谱不是命令；FROM 选基础镜像且必须是第一行，CMD 规定 run 时跑什么、build 时不跑。优先用 exec 形式，让你的进程自己当 PID 1，停止信号才能直达，而不是躲在 shell 背后。）
