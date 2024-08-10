# vscode环境配置

## C/C++环境配置
### 配置文件
#### tasks.json
在tasks.json 文件是用来定义和配置任务（Tasks）
下面是一些示例
```json
"${fileDirname}\\imgui\\*.h",
"${fileDirname}\\imgui\\*.cpp",
"-ld3d11",
"-lgdi32",
"-ldwmapi",
"-ld3dcompiler",
// `${fileDirname}` 打开当前目录文件；通配符（*）
//`-ld3d11` 链接到d3d11.lib库
```