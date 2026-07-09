# Agent for ESP IDF

> 以 ESP-IDF 最简单的 hello world 工程为例，演示 AI Agent 通过 `CLAUDE.md` / `AGENTS.md` 规则文件自动对ESP工程的编译、烧录、监视，分析固件大小等功能。


## 环境准备

- 系统：WSL Linux / macOS
- 软件：tmux
- ESP-IDF 环境：v5.5.3
- AI Agent: Codex (GPT5.4/GPT5.5)、Claude Code(deepseek)
- 开发板: ESP32-C5 等，支持 esp32 / esp32p4 / esp32s3 / esp32c5

## 运行项目

在运行本项目之前，最好确保当前环境已经能够正常编译/烧录/监控 ESP-IDF 项目，尽可能排除不必要的环境问题干扰。尤其注意需要挂载好设备并提升读写权限。

### 下载项目
```sh
git clone https://github.com/Orionxer/agent_esp
```

### 启动AI
- Codex / Claude Code 等能够识别 `CLAUDE.md` 或 `AGENTS.md` 的 AI 均可以（`AGENTS.md` 为指向 `CLAUDE.md` 的软链接，两者内容一致）
- 聊天框输入：**编译项目**，可以尝试进入 tmux 观察 AI 的执行结果

## 编译对上下文窗口的占用
> 以下数据基于 Codex GPT5.4 测试（Claude Code 等不同 AI 的上下文机制和计费方式有所差异，仅供参考）。

Codex GPT5.4，上下文窗口大小 *258k*：完整编译一次大概花费 **20K**，占用上下文窗口 **8%** ，后续只要不是全量编译，编译一次占用会降低，大概在 **1%**

- [x] 使用子智能体分担编译任务，减少主智能体的上下文窗口占用 ==>> 实测上下文窗口压缩的效果不明显，且**编译时间明显增加**，耗时接近2分钟，时间主要花在调度智能体上。

## ⚡ 直接获取规则文件

`CLAUDE.md` 为主规则文件，`AGENTS.md` 为指向 `CLAUDE.md` 的软链接。在项目根目录下执行以下命令获取，并根据实际情况进行适配性修改：
```sh
curl -O https://raw.githubusercontent.com/Orionxer/agent_esp/master/CLAUDE.md
```
`AGENTS.md` 为软链接，clone 项目即可自动处理，或手动创建:
```sh
ln -s CLAUDE.md AGENTS.md
```

## tmux监控
由于AI Agent通过tmux执行命令并读取输出，因此在使用过程中，用户也可以直接在tmux中查看命令执行的过程和结果，甚至可以直接在tmux中执行一些命令来辅助排查问题。

### 列出当前所有tmux session
```sh
tmux ls
```
### 进入指定的tmux session
```sh
tmux attach -t agent_esp
```

## 规则文件适配性修改

> `CLAUDE.md`（及软链接 `AGENTS.md`）需要根据实际的硬件以及软件情况进行适配性修改。

### IDF版本适配
当前使用的 ESP-IDF 版本是 **v5.5.3**，修改 `activate_idf_v5.5.3.sh` 中的版本号以匹配实际环境。

### 目标设备
规则已支持**自动检测**：优先从 `sdkconfig` 读取 `CONFIG_IDF_TARGET`，无需手动修改目标设备。若 `sdkconfig` 不存在或读取失败，AI 会通过弹窗交互让用户选择芯片型号（esp32 / esp32p4 / esp32s3 / esp32c5 等）。

### 系统兼容性
规则已兼容 WSL Linux 和 macOS。设备端口部分：
- **WSL Linux**：采用 `usbipd` 挂载并提升读写权限
- **macOS**：直接使用 `/dev/tty*` 端口，通常无需额外挂载

- [x] Mac 版本 `CLAUDE.md` ✅ 已支持

### 烧录波特率
当前使用的烧录波特率是 `6000000`，实际波特率受限于USB转串口芯片速率上限、USB线材质量等环境因素，实际可能需要调低。修改 `idf.py -b 6000000 flash` 中的波特率数值即可。



## 获取.gitignore

直接在项目根目录下执行以下命令获取 `.gitignore`，并根据实际情况进行适配性修改：

```sh
curl -O https://raw.githubusercontent.com/Orionxer/agent_esp/master/.gitignore
```
