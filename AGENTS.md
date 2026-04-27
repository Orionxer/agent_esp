# ESP-IDF Agent Rules

## 核心约束 (Always Active)
- **Tmux 管理**：优先复用现有 `tmux` 会话及已激活环境；仅在必要时新建。会话名默认取当前目录名（如 `agent_esp`）。
- **环境激活**：需执行 IDF 命令前，先确保 `source ~/.espressif/tools/activate_idf_v5.5.3.sh` 已执行。
- **IDF命令执行上下文强约束**：所有 `idf.py` 命令必须在 tmux 会话内执行；禁止在普通 shell 执行。
- **禁止替代调用**：禁止使用 `python .../idf.py`、`$IDF_PYTHON_ENV_PATH/bin/python .../idf.py` 等方式替代 `idf.py`。
- **失败处理优先级**：若 `idf.py` 在 tmux 内不可用，必须先修复环境（重新激活/检查 alias 与 PATH），未修复前不得继续构建；必须明确向用户报告“流程未合规，未执行构建”。
- **串口互斥**：若 `idf.py monitor` 正在运行，先发送 `Ctrl+]` 退出再执行其他命令。
- **禁止命令**：**严禁**执行 `idf.py fullclean`。
- **LSP 禁用**：**不要**运行 LSP 诊断/检查、语言服务器验证或基于 IDE 的代码分析，除非用户明确请求。

## 目标设备检查 (首次激活后执行一次)
1. 执行 `grep -rn TARGET sdkconfig` 获取当前 Target。
2. 若结果非 `esp32c5`，执行 `idf.py set-target esp32c5`。

## 通用硬件错误排查流程 (适用于 Flash / Monitor 失败)
当烧录或监控串口命令失败时，**必须**执行以下排查并反馈用户：
1. **问题诊断**：检查是[未挂载]、[无权限] 还是 [端口占用]。
2. **修复指引**（提示用户根据实际情况替换 `BUSID` 和 `PORT`）：
   - 未挂载：`usbipd.exe attach --wsl --busid <BUSID>`
   - 无权限：`sudo chmod 666 <PORT>`
   - 端口占用：`lsof <PORT>` 查看进程
3. **注意**：优先尝试自动获取端口号（通过 `ls /dev/tty*`），若失败再提示用户手动指定。

## 任务指令集

### 1. 编译项目 (Build)
- **前置**：进入tmux，激活环境，执行 `clear`。
- **动作**：`idf.py build`
- **注意**：严禁使用 `&&` 串联激活和编译命令。

### 2. 烧录固件 (Flash)
- **前置**：进入tmux，确保环境已激活。
- **动作**：`idf.py -b 6000000 flash`
- **错误处理**：若失败，按 **【通用硬件错误排查流程】** 执行。

### 3. 分析固件大小 (Size)
- **前置**：进入tmux，确保环境已激活。
- **动作**：`idf.py size`
- **输出要求**：仅汇报表格，格式如下（单位自适应 KB/MB）：
  | 项目 | 总大小 | 当前大小 | 使用率 | 剩余大小 |
  | :--- | :--- | :--- | :--- | :--- |
  | 固件分区 | - | - | - | - |
  | HP SRAM | - | - | - | - |

### 4. 监控串口 (Monitor)
- **前置**：进入tmux，确保环境已激活。
- **动作**：`idf.py monitor`
- **错误处理**：若失败，按 **【通用硬件错误排查流程】** 执行。

### 5. 一键构建 (Build + Flash + Monitor)
- **前置**：进入tmux，确保环境已激活，执行 `clear`。
- **动作**：`idf.py build && idf.py -b 6000000 flash && idf.py monitor`
- **错误处理**：若烧录或监控阶段失败，**中断流程**，按 **【通用硬件错误排查流程】** 执行。
