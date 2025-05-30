# lego-gogogo

## 这是什么？

借助 Flask 用于远程控制 Lego EV3 的简单实现（按钮等功能在远程执行程序时可能无法使用）。  
此外，仓库内自带了由 AI 生成的前端页面 `web/index.html`。

## 怎么使用？

1 首先，确保在 EV3 安装了 `ev3dev`。这是一个基于 Debian 的系统，更多信息可以在 https://www.ev3dev.org/ 找到。  
**请注意！** 我们测试使用的系统并不是官方版本，而是基于 [此处讨论](https://github.com/orgs/ev3dev/discussions/1634#discussion-6831416) 提到的 Docker 镜像修改的。

2 在 EV3 上为 Python 安装软件包 `Flask flask-cors evdev`。由于 EV3 本身性能较低，推荐直接修改系统，安装软件包后，再重新打包并烧录新系统。具体方法可参考上方“讨论”链接（注：原作者所给出的示例命令有误，实际使用请看下方示例）。  
下面针对 `Ubuntu 22.04` 系统（已安装 Docker），按我们的实际操作给出示例。

- 首先，安装所需软件包并拉取 Docker 镜像。

```bash
sudo add-apt-repository ppa:ev3dev/tools
sudo apt-get update && sudo apt-get install -y qemu-user-static brickstrap
sudo docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
sudo docker pull growflavor/ev3images:ev3dev10imgv02b
sudo docker run -it growflavor/ev3images:ev3dev10imgv02b su -l robot
```

- 接下来，你将看到 `robot@ ` 开头的提示符。执行下面命令为自带的 Python 3.11 虚拟环境安装软件包：

```bash
./venv311/bin/pip install Flask flask-cors evdev
```

- 安装完成后，登出。

```bash
exit
```

- 然后，将刚刚修改的容器提交到镜像，打包为 img 文件。

```bash
sudo docker ps -a
sudo docker container commit 上一步查询到刚刚更改的容器ID.例如b09c76f149e1 ev3d10new:latest
sudo brickstrap create-tar ev3d10new:latest ev3dev10new.tar
sudo brickstrap create-image ev3dev10new.tar ev3dev10new.img
```

- 将上面得到的 img 文件用烧录工具烧录到 SD 卡即可。

3 为 EV3 配置好网络通信设置，可参考 https://www.ev3dev.org/docs/networking/ 。将 `python-src` 目录下的文件上传到 EV3，示例位置：`/home/robot/python-src`。不要忘记为 `app.py` 添加执行权限:`chmod +x /home/robot/python-src/app.py`。

4 运行 `app.py` 即可，等待启动。启动完成后，即可在前端控制台网页连接到 EV3。

## 怎么开机启动 / 后台运行？

可以利用 systemd 服务实现。请通过 SSH 连接到 EV3，参考下面的示例进行操作：

- 首先创建一个服务单元文件:

```bash
sudo vim /etc/systemd/system/lego-api.service
```

- 写入以下内容并保存：

```
[Unit]
Description=Lego EV3 API Service
After=network.target

[Service]
User=robot
WorkingDirectory=/home/robot
ExecStart=/home/robot/venv311/bin/python3 /home/robot/python-src/app.py
Restart=always
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=lego-api
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
```

- 重新加载 systemd 配置，并配置开机启动、立即启动：

```bash
sudo systemctl daemon-reload
sudo systemctl enable lego-api.service
sudo systemctl start lego-api.service
```

下面是一些常用控制命令：  
检查服务状态 `sudo systemctl status lego-api.service`  
停止服务 `sudo systemctl stop lego-api.service`  
查看服务日志 `sudo journalctl -u lego-api.service`  
## 💡Tips!  
**CSDN = Copy, Steal, pay-to-Download Network**.
