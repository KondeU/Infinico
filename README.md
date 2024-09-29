# Infinico

Infinico是一个简易的SDF渲染框架，可以通过编写fragment/pixel shader来实现SDF的渲染呈现。

Infinico通过json配置文件的形式来配置主题(theme)，使用方法如下。

## 全局配置

全局配置可以配置主题名称（对应文件夹）、应用是否自启动与自启动配置过程是否静默。

```json
# config.json
{
    "theme": "qiaomuyu",
    "autorun": false,
    "silence": true
}
```

- theme字段对应主题名称：表示将使用应用目录内themes文件夹下的qiaomuyu子文件夹内的主题。
- autorun对应应用自启动：表示应用是否加入自启动，若为true，则尝试将该应用加入开机自启动注册表，该功能需要管理员权限。
- silence对应自启动静默：表示是否在自启动设置失败时，弹出警告对话框，若为true则静默不弹出告警，如果不使用管理员权限运行该程序，建议置为true来静默自启动配置失败告警。

## 主题配置

themes目录下用来存放主题，默认提供了一个敲木鱼的主题，主题名称为qiaomuyu，该主题的目录名即为qiaomuyu。

具体的主题文件目录下：至少需要有一个theme.json文件来组织主题。可选一个theme.ico文件作为主题图标文件，如果不提供该文件，将使用默认程序图标作为主题图标文件。

```json
# theme.json
{
    "version": 1,
    "width": 128,
    "height": 128,
    "menus": [
        {
            "name": "Open CMD...",
            "shell": "cmd.exe"
        },
        {
            "name": "Screenshot...",
            "shell": "SnippingTool.exe"
        }
    ],
    "shader": "qiaomuyu.hlsl",
    "images": [
        {
            "name": "muyu",
            "file": "muyu.png"
        },
        {
            "name": "gongde",
            "file": "gongde.png"
        }
    ],
    "audios": [
        {
            "trigger": "left_click_down",
            "file": "qiaomuyusheng.wav"
        }
    ]
}
```

- version字段：主题配置文件版本号，目前仅支持初代版本，必须为1
- width&height字段：窗口（渲染画布）的大小，必须为大于0的整型数字，不能超过屏幕尺寸
- shader字段：SDF的shader文件，支持HLSL语言
- images字段：在shader中传递的图像，name为在shader中的贴图名称，对应于Texture2D<uint4>，file为图像在磁盘中的相对文件名，支持png格式图像
- audios字段：管理音频播放，trigger目前支持left_click_down、right_click_down，分别对应鼠标左键和右键点击时播放音频，file为音频在磁盘中的相对文件名，支持wav格式音频
- menus字段：自定义右键功能菜单，name为菜单项名称，shell为要执行的shell脚本，支持bat语法

注意：配置文件的字段顺序不可更改，必须按如上示例的顺序组织。
即：version-width-height-menus(name-shell)-shader-images(name-file)-audios(trigger-file)的顺序有严格要求，不可交换次序。

## shader

SDF的shader，支持HLSL语言，可配置传送图像作为shader中的贴图，同时附加传送默认信息在constant buffer中可供使用，以qiaomuyu的主题简单举例。

```hlsl
// Input information:
// float2 canvasSize;          @ width, height
// float2 mouseCursor;         @ position: x, y
// float2 clickFlytime;        @ mouse: left, right
// uint2  clickCount;          @ mouse: left, right
// float2 cpuAndMemoryState;   @ percentage: cpu, memory
// SamplerState simpleSampler; @ linear sampler with wrap address mode.

float4 ImagePixelProcess(int2 canvasPosition)
{
    float4 output = float4(muyu.Load(int3(canvasPosition, 0))) / 255.0;
    uint gongdeWidth, gongdeHeight;
    gongde.GetDimensions(gongdeWidth, gongdeHeight);
    if (canvasPosition.y < int(gongdeHeight)) {
        uint4 gongdePixel = gongde.Load(int3(canvasPosition, 0));
        if ((gongdePixel.x + gongdePixel.y + gongdePixel.z) < (255 * 3)) {
            uint4 flytimePixel = gongdePixel + uint(clickFlytime.x);
            output = saturate(float4(flytimePixel) / 255.0);
        }
    }
    return output;
}
```

默认信息为：
- float2 canvasSize：(x,y)为图像的(宽,高)，像素单位
- float2 mouseCursor：当前的鼠标坐标位置(x,y)，像素单位，若鼠标不在窗口内，则为(-50000000, -50000000)
- float2 clickFlytime：鼠标单击后的飞行时间，若在窗口内单击，则flytime会归0，(x,y)分别对应左键和右键，单位是毫秒
- uint2  clickCount：鼠标单击的累计次数
- float2 cpuAndMemoryState：(x,y)分别是当前的CPU使用率与内存使用率，0~100的百分比

float4 ImagePixelProcess(int2 canvasPosition)是可供定制实现的SDF入口函数，canvasPosition是自定义算法当前处理的画布的位置值。
返回值为当前处理的画布的位置的归一化输出颜色值，对应RGBA，取值范围0~1。

**Shader的调试：**Infinico会将shader的编译错误信息输出到DebugMessage中，您可以使用[DebugView](https://learn.microsoft.com/zh-cn/sysinternals/downloads/debugview)进行查看。

## 窗口功能

程序启动执行后，会在鼠标当前的位置创建浮动窗口，并同时创建应用的系统托盘图标，单击图标可以隐藏浮动窗口，再次单击则恢复显示。

## 更多主题

我们开放主题频道，大家可以自定义自己的主题并进行分享交流，充分碰撞想法。

您可以前往idea讨论中心：https://github.com/KondeU/Infinico/discussions/categories/ideas

## 更新日志

**2024/9/29 1.2024.9.29**

加入新功能，并修复一些bug，发布正式的版本。

框架功能更新：
1. 修复下发至shader的UBO数据中的memory usage单位规格错误的问题，调整为\[0.0, 100.0\]的百分比，与cpu usage的单位规格一致；
2. 添加鼠标位置数据、单击统计数据下发至shader的UBO中的支持；
3. 添加最小化至系统托盘的支持，单击系统托盘图标，可以将浮动窗口最小化到托盘中，再次点击则恢复显示，方便小组件的隐藏，系统托盘图标可用户自定义；
4. 添加音频混叠的支持，快速播放多次音频时声音不会中断而是混叠；

默认主题更新：
1. 升级内置的敲木鱼主题的敲击木鱼声，高清影棚录制，敲击声深沉悠扬；
2. 添加敲木鱼计数功能，功德数实时呈现。

**2024/9/1 0.1.0.1**

发布Infinico的初始版本！
