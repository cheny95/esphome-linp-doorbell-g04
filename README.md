# 领普自发电门铃G6L-WIFI版基于ESPHome的定制化组件(linp-doorbell-g04)

### 本仓库forked from [pauln/esphome-linp-doorbell-g04](https://github.com/pauln/esphome-linp-doorbell-g04)，加以翻译及内容修改。

## 背景
领普自发电门铃G6L（Linptech G6L-WIFI）是一款带有自发电按钮的wifi门铃。它是Mijia（小米智能家居）生态系统的一部分，但是，通过各种插件，无法在局域网内找到它的事件（例如按下按钮）。因此，该项目旨在提供替换固件，以通过[ESPHome](https://esphome.io/)实现完全本地控制，然后可以接入[Home Assistant](https://www.home-assistant.io/)，提供了一些简单实体。

<img width="1048" alt="image" src="https://user-images.githubusercontent.com/6293952/224540313-dc5a4e79-ddae-4f08-8e55-280283cfa5dd.png">

截至 2023 年初，最新版本是 linp-doorbell-g04，它不再包含辅助微控制器来处理基本的门铃功能;ESP32 现在直接连接到射频接收器和门铃提示芯片。如果您有带有 STM8S005K6 微控制器的早期 linp-doorbell-g03 版本，请参阅 [linp-doorbell-g03](https://github.com/pauln/esphome-linp-doorbell-g03/tree/feature/external_components) 仓库以获取适用于您设备的自定义组件。

 
## 硬件
门铃的接收器是设计的很紧实，没有螺钉或卡扣。从两半的接缝轻轻翘起，然后拉开。

**注意断电！**

打开后，您会看到一个 ESP32 模块（用于处理 wifi/网络/云控制，但不能处理基本的门铃功能），旁边有一排焊点，标记（从上到下）：
- RXD0
- TXD0
- IO0
- GND
- 3.3V

这些焊盘可用于通过 UART 串口与 ESP32 通信，以便将 ESPHome 刷写到 ESP32。

如果将主电路板从塑料外壳中取出，你会在电路板背面的靠中间的位置找到“3.3V”和“GND”焊点，它们可用于为整个装置供电而不是为电源供电。 
您也可以从 ESP32 旁边的 3.3V 和 GND 焊盘为其供电，但使用这些背焊盘可以更轻松地使用其他焊盘来为 ESP32 充电。 
我强烈建议您在处理门铃时通过这些垫子中的一组为其供电，因为使用隔离的 3.3V 直流电源更安全（我在插入 USB 的 USB 电缆末端使用 3.3V 稳压器 充电器或电池组），而不是在重新刷写 ESP32 时尝试从电源供电。 （你也可以电路板上拆焊了电源输入线，拔下扬声器，这样电路板就可以完全从塑料外壳中取出）。

## ESP32 的奇怪问题
该设备中使用的 ESP32 模块是单核变体。 此外，出厂时写入 EFUSE 的 MAC 地址 CRC 似乎不正确，因此需要修改核心库以禁用 MAC CRC 检查失败时的重置。 
最新版本的 ESPHome 提供了配置选项来处理使用 ESP-IDF 框架时的这两个问题； 有关适当的配置，请参阅“doorbell.yaml”。

## 刷机要求
- 安装ESPHome
- 串口转 USB 转接器

## 配置
- 在 YAML 文件的 `wifi:` 部分设置您自己的 WiFi 凭据，以便它知道如何连接到您的网络。 示例配置使用 !secret 指令； [有关信息，请参阅 ESPHome 文档](https://esphome.io/guides/faq.html) 如果您对此不熟悉。
- 如果您还没有使用串行记录器从库存固件中获取按钮 ID，请确保启用了转储程序（`remote_receiver` 配置中的`dump: linptech_g6l`）。

## 刷写 ESP32
为了将 ESP32 启动到刷机模式（以便您可以向其写入 ESPHome），您需要将最靠近 ESP32 模块右下角的两个引脚接地； 我通过将细线焊接到底部边缘最右边的引脚、ESP32旁边的“IO0”和“GND”测试点来做到这一点，但是如果你有更简单的方法将两个边角焊盘暂时可靠地短路到地，也可以。
![image](https://user-images.githubusercontent.com/6293952/224540976-e64cf18d-0446-47ff-baaf-ff145d3db8ef.png)


1. 将 GND、TXD0、RXD0 连接到串口转 USB 适配器（确保将 ESP 的 RXD0 连接到适配器的 TX（或 TXD）引脚，将 ESP 的 TXD0 连接到适配器的 RX（或 RXD）引脚）
2. 将您的串口转 USB 适配器连接到您的计算机，记下它显示的端口（取决于您的操作系统，这可能类似于 COM0 或 /dev/ttyUSB0）
3. 将右下角的2个焊盘（如上所述）短接至 GND
3. 使用电路板背面的 3.3V 和 GND 焊盘为门铃供电
4. 运行 `esphome run doorbell.yaml`（将“doorbell.yaml”替换为您的 YAML 文件名称）
5. 编译完成后，系统会提示您选择如何执行更新； 选择您的串口转 USB 适配器
6. 刷机完成后，拔掉门铃的5V电源，去掉右下角那对引脚和GND的短接，重新上电即可正常开机
7. ESP32 应该启动并连接到您在 yaml 文件中配置的 WiFi 网络； 然后您可以将它添加到您的 Home Assistant 并开始将它集成到您的家庭自动化中！

## 配置发射器按钮
刷入 ESPHome（启用转储程序）后，使用“esphome logs doorbell.yaml”连接（通过串口或 wifi）到 ESP32 并查看日志输出。 按其中一个按钮应该会产生如下日志消息：

`[remote.linptech_g6l:068]: Received Linptech G6L: address=0x123456`

然后，您可以在配置文件里的remote_receiver区域 （参见 doorbell.yaml）中使用 address= 之后的部分（包括 0x 前缀）。

## Home Assistant
使用 ESPHome 刷写门铃并连接到 Home Assistant 后，您应该会看到以下服务出现（如果您将它们包含在您的配置中）：

| Service name  | Description | Parameter 1 | Parameter 2 | Parameter 3 |
| ------------- | ----------- | ----------- | ----------- | ----------- |
| `esphome.doorbell_play_tune` | Play a tune/chime | `乐曲` \[int, 1-40] | `音量` \[int, 1-8] | `模式` \[int, 1-4] |
| `esphome.doorbell_stop_playing` | Stop the tune/chime, if one is currently playing |  |  |

有关可用“曲调”的列表，请参阅 [SZY8039B 数据表](https://github.com/cheny95/esphome-linp-doorbell-g04/blob/main/SZY8039B.pdf) 中的表格。 


`mode` 参数不是特别有用，因为在 G6L-WIFI 中，LED 连接到 ESP32 而不是 SZY8039B。 因此，模式 1、2 和 4 播放音乐而模式 3 不播放（这没什么用）。

请注意，所有服务名称上的 `doorbell` 前缀是您的 ESPHome 节点的名称，如您的 yaml 文件的 `esphome:` 块中所定义。

如果这些服务没有出现在 Home Assistant 中，请尝试重新启动门铃，以便它重新连接到 Home Assistant。
