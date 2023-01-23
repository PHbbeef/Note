注：Linux没安装gdb需要安装
相关视频地址：[用VSCode调试C++其实很简单！配置文件的个人心得](https://www.bilibili.com/video/BV1nE411C7Va?share_source=copy_web)

launch.json
```
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name":"C++ Launch",
            "type":"cppdbg",
            "request": "launch",
            "program": "${workspaceRoot}/a.out",
            "stopAtEntry": false,
            "externalConsole": true,
            "cwd":"${workspacefolder}",
            "preLaunchTask": "task",
            "windows": {
                "MIMode": "gdb",
                "miDebuggerPath": "/usr/bin/gdb"
            }
        }
    ]
}
```


tasks.json
```
{
    "version": "2.0.0",
    "tasks": [
    {
        "label": "task",
        "type": "shell",
        "command":"g++",
        "args":[
            "-g",
            "${file}"
        ],
        "group":{
            "kind": "build",
            "isDefault": true
        }
    }
  ]
}
```