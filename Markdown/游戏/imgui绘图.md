# 安装
imgui项目地址：<a href="https://github.com/ocornut/imgui">GitHub</a>

基本代码（删除无用的代码只保留基础），在百度网盘`代码`文件夹（imgui_back.7z）


# 基本结构
有头有尾
```C++
ImGui::Begin();
// xxxx
ImGui::End();
```
|||
|--|--|
|ImGui::Begin()|这个方法有两个参数，窗口标题，true或false（这是窗口是否显示有可关闭的按钮|
|ImGui::End()||

其它相关代码说明
```C++
//加载字库
ImFont* font = io.Fonts->AddFontFromFileTTF("c:\\Windows\\Fonts\\msyh.ttc", 19.0f, nullptr, io.Fonts->GetGlyphRangesChineseFull());

//中文乱码如何解决
ImGui::Begin(u8"你好",&windows_open);

//更改风格样式；样式是有三种可以选择
ImGui::StyleColorsClassic();
```


# 控件
```C++
//在同一行
ImGui::SameLine();
xxx
ImGui::SameLine();

//颜色
ImGui::TextColored(ImColor(0.255,255,255),u8"PHb制作");
```

|||
|-|-|
|Button|按钮；按下时为true|
|BulletText|符点缩进|
|RadioButton|按钮复选框|
|DragFloat|滑动条快|
|SliderFloat|浮动条块|


# 绘制
|||
|-|-|
|AddLine|直线|
|AddRect|矩形|
|AddCircle|圆形|
|AddText|文本|

代码示例
```C++
ImGui::GetForegroundDrawList()->AddText(ImVec2(10,10),ImColor(0.255,255,255),u8"你好世界");
```

## 窗口
### 窗口透明
<a href="https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/nf-winuser-setlayeredwindowattributes">微软文档地址</a>
+ 窗口透明颜色和字体颜色如果设置255（255是黑色），颜色不要和主题颜色类似。字体显示会出现问题

```C++
SetLayeredWindowAttributes  //句柄，颜色，透明度，样式

// 示例
SetLayeredWindowAttributes(hwnd,ImColor(0,0,0),NULL,LWA_COLORKEY);
```

### 去除标题栏

关键参数详细`微软文档`
|||
|-|-|
|CreateWindowExW|窗口层叠样式|
|WS_POPUP|去除窗口标题|

```C++
HWND hwnd = ::CreateWindowExW(WS_EX_TOPMOST|WS_EX_LAYERED,wc.lpszClassName,NULL,WS_POPUP,100, 100, 1280, 800, nullptr, nullptr, wc.hInstance, nullptr);
```


### 跟随游戏窗口移动
+ 什么是客户区？

```C++
HWND GameHwnd = FindWindowA("MainWindow",NULL);  //获取游戏窗口句柄
RECT GameRect;
GetClientRect(GameHwnd,&GameRect);    //获取客户区坐标
POINT ClientD2D = {GameRect.left,GameRect.top};
ClientToScreen(GameHwnd,&ClientD2D);
MoveWindow(hwnd,ClientD2D.x,ClientD2D.y,GameRect.right,GameRect.bottom,true);
```


# 内存读写
读取内存首先获取进程的句柄，进程句柄获取有两种
+ 通过窗口句柄->api->获取PID

引入相关头文件
|||
|-|-|
|Windows.h||
|tlhelp32.h||

示例：
```C++
DWORD PID;
HWND hwnd = FindWindowA("MainWindow","植物大战僵尸中文版");
GetWindowThreadProcessId(hwnd,&PID);
```

|||
|-|-|
|ReadProcessMemory|读内存|
|WriteProcessMemory|写内存|