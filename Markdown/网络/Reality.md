## Reality

在常规的 TLS in TLS 代理中, 数据虽然被多层加密, 但每一层都会引入额外的包头。这种 “套娃加密” 带来的数据增量虽小, 却可能形成某些统计特征, 从而被防火墙识别。
为了解决这一问题, XTLS 曾尝试通过减少额外加密来改善特征。而在 Xray 1.8.0 中推出的 Vision 流控, 则让绝大多数数据包保持原始特征, 几乎无可区分, 从而显著提升了隐蔽性。
然而, 仅靠 TLS 并不能彻底避免端口封禁与指纹识别。为解决这一问题, Xray 在 2023 年 3 月推出了 REALITY 协议, 用来取代传统 TLS 服务。REALITY 能够有效消除服务端的 TLS 指纹, 保持前向保密性, 并且无需域名、证书或 443 端口, 即可伪装成任意目标网站。相比传统 TLS, REALITY 在安全性、便捷性和抗封锁能力上都更进一步。

Reality 不需要你自己购买域名和证书, 因为它可以偷别人的域名。但并非所有域名都能用, 目标网站的选择会直接影响速度、连通性和稳定性。


## Reality 条件
+ 目标网站必须支持 TLS1.3
+ 目标网站必须支持 X25519
+ 目标网站必须支持HTTP/2 (H2)
+ 目标域名必须和 SNI 匹配
+ 不使用 CDN
    + 如果 Reality 目标网站使用 CDN, 数据将转发到 CDN 节点, 使你的 Reality 节点成为别人的反向代理加速节点

### 如何寻找目标域名
[Reality协议目标网站检测工具](https://github.com/V2RaySSR/RealityChecker)

首先使用 `RealiTLScanner` 工具，扫描 VPS，导出结果 `file.csv` 文件，命令如下：
```bash
# MacOS、Linux 命令
# 若是提示文件损坏请 执行 sudo xattr -r -d com.apple.quarantine （app地址）
./RealiTLScanner -addr VPSIP -port 443 -thread 100 -timeout 5 -out file.csv
 
# Windows 命令(程序在桌面):
cd "%USERPROFILE%\Desktop"
RealiTLScanner-windows-64.exe -addr VPSIP -port 443 -thread 100 -timeout 5 -out file.csv


# RealiTLScanner 尽量在本地运行，不要在远端
# 多次运行RealiTLScanner时，请更改输出文件名，如：file1.csv、file2.csv、file3.csv 等
# 如果使用相同的文件名，可能会导致文件导出失败或覆盖之前的扫描结果

# 使用xray检查网站信息
xray.exe tls ping www.baidu.com

```

TLS1.3 X25518 检查
```bash
# 打开目标网站, 按 F12 打开开发者工具
# 切换到 隐私与安全 选项卡
# 在「网络连接」信息中, 应显示 TLS 1.3、X25519 和 AES_xxx_xxx
```

HTTP/2 检查
```bash
# 在开发者工具的控制台输入下面内容, 返回值应为 h2
window.chrome.loadTimes()?.npnNegotiatedProtocol
```

## REALITY 搭建
[REALITY](https://github.com/XTLS/REALITY)



服务端配置
```json
{
    "inbounds": [ // 服务端入站配置
        {
            "listen": "0.0.0.0",
            "port": 443,
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "", // 必填，执行 ./xray uuid 生成，或 1-30 字节的字符串
                        "flow": "xtls-rprx-vision" // 选填，若有，客户端必须启用 XTLS
                    }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "raw",
                "security": "reality",
                "realitySettings": {
                    "show": false, // 选填，若为 true，输出调试信息
                    "target": "example.com:443", // 必填，格式同 VLESS fallbacks 的 dest
                    "xver": 0, // 选填，格式同 VLESS fallbacks 的 xver
                    "serverNames": [ // 必填，客户端可用的 serverName 列表，暂不支持 * 通配符
                        "example.com",
                        "www.example.com"
                    ],
                    "privateKey": "", // 必填，执行 ./xray x25519 生成
                    "minClientVer": "", // 选填，客户端 Xray 最低版本，格式为 x.y.z
                    "maxClientVer": "", // 选填，客户端 Xray 最高版本，格式为 x.y.z
                    "maxTimeDiff": 0, // 选填，允许的最大时间差，单位为毫秒
                    "shortIds": [ // 必填，客户端可用的 shortId 列表，可用于区分不同的客户端
                        "", // 若有此项，客户端 shortId 可为空
                        "0123456789abcdef" // 0 到 f，长度为 2 的倍数，长度上限为 16
                    ],
                    "mldsa65Seed": "", // 选填，执行 ./xray mldsa65 生成，对证书进行抗量子的额外签名
                    // 下列两个 limit 为选填，可对未通过验证的回落连接限速，bytesPerSec 默认为 0 即不启用
                    // 回落限速是一种特征，不建议启用，如果您是面板/一键脚本开发者，务必让这些参数随机化
                    "limitFallbackUpload": {
                        "afterBytes": 0, // 传输指定字节后开始限速
                        "bytesPerSec": 0, // 基准速率（字节/秒）
                        "burstBytesPerSec": 0 // 突发速率（字节/秒），大于 bytesPerSec 时生效
                    },
                    "limitFallbackDownload": {
                        "afterBytes": 0, // 传输指定字节后开始限速
                        "bytesPerSec": 0, // 基准速率（字节/秒）
                        "burstBytesPerSec": 0 // 突发速率（字节/秒），大于 bytesPerSec 时生效
                    }
                }
            }
        }
    ]
}
```

客户端
```json
{
    "outbounds": [ // 客户端出站配置
        {
            "protocol": "vless",
            "settings": {
                "vnext": [
                    {
                        "address": "", // 服务端的域名或 IP
                        "port": 443,
                        "users": [
                            {
                                "id": "", // 与服务端一致
                                "flow": "xtls-rprx-vision", // 与服务端一致
                                "encryption": "none"
                            }
                        ]
                    }
                ]
            },
            "streamSettings": {
                "network": "raw",
                "security": "reality",
                "realitySettings": {
                    "show": false, // 选填，若为 true，输出调试信息
                    "fingerprint": "chrome", // 选填，使用 uTLS 库模拟客户端 TLS 指纹，默认 chrome
                    "serverName": "", // 服务端 serverNames 之一
                    "password": "", // 服务端私钥生成的公钥，对客户端来说就是密码
                    "shortId": "", // 服务端 shortIds 之一
                    "mldsa65Verify": "", // 选填，服务端 mldsa65Seed 生成的公钥，对证书进行抗量子的额外验证
                    "spiderX": "" // 爬虫初始路径与参数，建议每个客户端不同
                }
            }
        }
    ]
}
```
