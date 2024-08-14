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

q怎么读写内存？             
+ a：PID->进程句柄->读写内存

引入相关头文件
|||
|-|-|
|Windows.h||
|tlhelp32.h||


示例：
```C++
DWORD PID;
HWND hwnd = FindWindowA("MainWindow","植物大战僵尸中文版"); //如果获取进程PID为0，把第二个参数为空"NULL"
GetWindowThreadProcessId(hwnd,&PID);

cout << hex << PID << endl; //输出十六进制格式
```

获取进程句柄函数
```C++
//进程句柄函数
INT GetProcess(DWORD Pid)
{
    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, Pid);
    return reinterpret_cast<intptr_t>(hProcess);
}
```

## 两种获取句柄的方式

第一种获取进程句柄
```C++
    DWORD PID;
    HWND hwnd = FindWindowA("MainWindow",NULL);
    GetWindowThreadProcessId(hwnd,&PID);

    INT hp = GetProcess(PID);   //这是获取进程句柄函数

    cout << hex << "进程PID" << PID << endl;
    cout << hex << "进程句柄" << hp << endl;
```

第二种获取进程句柄
```C++
// process_snapshot 进程快照
// process_information 进程信息
// or_not 是否存在

INT GetPid(const WCHAR* name){
   HANDLE process_snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS,0);
   if(process_snapshot = INVALID_HANDLE_VALUE){return 0;}

   PROCESSENTRY32W process_information;

   process_information.dwSize = sizeof(PROCESSENTRY32);

   BOOL or_not = Process32FirstW(process_snapshot,&process_information);

   while (or_not != 0)
   {
        if(wcscmp(name,process_information.szExeFile) == 0){
            CloseHandle(process_snapshot);
            return process_information.th32ProcessID;
        }
        or_not = Process32NextW(process_snapshot,&process_information);
   }
   CloseHandle(process_snapshot);
   
}

INT hp = GetProcess(PID); //这是获取进程句柄函数

// 调用函数GetPid
DWORD PID = GetPid(L"MainWindow");

//输出内容
cout << hex << "进程PID" << PID << endl;
cout << hex << "进程句柄" << hp << endl;
```

句柄的获取方式是通过int强转获取，这里演示不强制转换
```C++
// 句柄函数
HANDLE GetProcess(DWORD Pid)
{
    return OpenProcess(PROCESS_ALL_ACCESS,0,Pid);
}

HANDLE hp = GetProcess(PID);

```

## 读取内存

|||
|-|-|
|ReadProcessMemory|读内存|
|WriteProcessMemory|写内存|

<a href="https://learn.microsoft.com/zh-cn/windows/win32/api/memoryapi/nf-memoryapi-readprocessmemory?redirectedfrom=MSDN">微软文档</a>

```C++
    u_int temp; //无符号类型；缓冲区；读写内存数值存放在这里
    ReadProcessMemory(hp,(LPCVOID)0x12E629D8,&temp,sizeof(u_int),NULL);
    
    cout << temp << endl;   //打印内存（这里打印的是十六进制）
```

写内存
```C++
    u_int temp;
    u_int sum = 1000;   //阳光数量修改
    WriteProcessMemory(hp,(LPVOID)0x12E629D8,&sum,sizeof(sum),0);
```