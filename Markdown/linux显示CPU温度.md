# linux显示CPU温度
## lm-sensors
```shell
#安装lm-sensors
apt-get install lm-sensors

#传感器授权操作
sensors-detect

#查看传感器温度
sensors
```
### 在htop中显示
```
F2Setup
Display options
勾选Also show cpu frequency(也显示cpu频率)及Also show CPU temperature(requires libsensors)(也显示cpu温)
```

## 其他选择
```
s-tui
```