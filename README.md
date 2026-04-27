# ESP-IDF 新建组件

> 组件可以通过**命令行**或者**VSCode**插件两种方式**创建**，下面分别介绍两种方式的步骤
## 环境准备

- 系统：WSL Ubuntu
- SDK: ESP-IDF **v5.5.3**
- 设备：esp32c5
## 直接验证

### 下载工程
```sh
git clone https://github.com/Orionxer/hello_components
```

### 验证步骤
- cd 进入工程
- 激活 ESP-IDF 环境
- 设置目标芯片类型
- 编译烧录监控

## 命令行新建组件

### 进入工作区
```sh
cd ~/esp/esp32c5
```
### 激活ESP-IDF环境
```sh
source ~/.espressif/tools/activate_idf_v5.5.3.sh
```
### 创建空工程
```sh
idf.py create-project hello_components
```
### 进入工程
```sh
cd hello_components
```
### 项目根目录下创建并进入组件目录
```sh
mkdir -p components && cd components
```
### idf.py创建组件
```sh
idf.py create-component my_component
```
### 回到项目根目录
```sh
cd ..
```
### 查看根目录结构
```sh
tree
```
### 根目录结构如下
```sh
.
├── CMakeLists.txt
├── components
│   └── my_component
│       ├── CMakeLists.txt
│       ├── include
│       │   └── my_component.h
│       └── my_component.c
└── main
    ├── CMakeLists.txt
    └── hello_components.c
```

## VSCode插件新建组件

- **Ctrl + Shift + P** 打开命令面板
- 输入 **ESP-IDF: Create New ESP-IDF Component**
- 输入组件名称，例如 **hello_component**

 生成的结构目录与命令行新建组件相同

## 组件加入编译

### 组件实现任意打印功能
`./components/my_component/my_component.c` 参考内容如下
```c
#include <stdio.h>
#include "my_component.h"
#include "esp_log.h"
void func(void)
{
    ESP_LOGI("MyComponent", "Hello, this is a message from MyComponent!");
}
```
### 添加组件依赖
`main/CMakeLists.txt` 参考内容如下
```cmake
idf_component_register(SRCS "hello_components.c"
                    INCLUDE_DIRS "."
                    REQUIRES my_component) # 强制要求先编译该组件
```
### 调用组件函数
`main/hello_world_main.c` 参考内容如下
```c
#include "hello_component.h"

void app_main(void)
{
    func(); // 调用组件函数
}
```

## 编译烧录监控验证

```sh
idf.py -b 6000000 build flash && idf.py monitor
```
