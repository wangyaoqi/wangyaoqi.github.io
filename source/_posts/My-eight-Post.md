---
title: 起一个Python接口，控制终端shell
date: 2024-09-10 20:07:00
tags:
---

## 选择
1.python  flask提供接口服务 记得做好跨域的osfp
###### pip3 install flask -i https://pypi.tuna.tsinghua.edu.cn/simple
2.编辑配置脚本，例如我的配置脚本是控制docker 开关   
###### 脚本权限
``` bash 
如果你的脚本存放在 /home/wangyaoqi/container/haikang 目录下，你可以按照以下步骤来配置 sudo 权限：

1. 打开 sudoers 文件
使用 visudo 命令来安全地编辑 sudoers 文件：

bash
复制代码
sudo visudo
2. 在 sudoers 文件中添加条目
找到文件的底部，添加以下两行以允许 wangyaoqi 用户在不输入密码的情况下运行这些脚本：


wangyaoqi ALL=(ALL) NOPASSWD: /bin/bash /home/wangyaoqi/container/haikang/start_docker.sh
wangyaoqi ALL=(ALL) NOPASSWD: /bin/bash /home/wangyaoqi/container/haikang/stop_docker.sh
3. 保存并退出 vim
按 Esc 键进入命令模式。
输入 :wq 保存并退出。
4. 确保脚本具有执行权限
确保 start_docker.sh 和 stop_docker.sh 脚本具有执行权限：


chmod +x /home/wangyaoqi/container/haikang/start_docker.sh
chmod +x /home/wangyaoqi/container/haikang/stop_docker.sh
5. 测试权限配置
你可以测试是否可以在不需要密码的情况下运行这些脚本：



sudo /bin/bash /home/wangyaoqi/container/haikang/start_docker.sh
sudo /bin/bash /home/wangyaoqi/container/haikang/stop_docker.sh
```

### 相关代码
``` bash 
    from flask import Flask, request, jsonify
from flask_cors import CORS  # 导入 CORS

app = Flask(__name__)
CORS(app)  # 允许所有域名的跨域请求

@app.route('/control', methods=['POST'])
def control():
    data = request.get_json()
    action = data.get('action')
    if action == 'start':
        # 处理 start 操作
        return jsonify({'status': 'success', 'message': 'Started successfully'})
    elif action == 'stop':
        # 处理 stop 操作
        return jsonify({'status': 'success', 'message': 'Stopped successfully'})
    else:
        return jsonify({'status': 'error', 'message': 'Invalid action'})

if __name__ == '__main__':
    app.run()
```
CORS(app) 允许所有域名访问你的 Flask 应用。如果你只希望允许特定域名，可以使用以下方式：
``` bash 
    python

    CORS(app, resources={r"/*": {"origins": "http://your-frontend-domain.com"}})

```
#### shell脚本
stop_docker.sh 和 start_docker.sh 应该像这样：
``` bash 
            bash
            复制代码
            #!/bin/bash
            sudo docker stop ad0eefcd5b13
            bash
            复制代码
            #!/bin/bash
            sudo docker start ad0eefcd5b13
```
#### 脚本权限
``` bash 
sudo chmod +x /home/wangyaoqi/container/haikang/stop_docker.sh
sudo chmod +x /home/wangyaoqi/container/haikang/start_docker.sh
```
### 脚本运行
#### Supervisor 配置

确保 supervisor 的配置文件（通常在 /etc/supervisor/conf.d/ 目录下）正确无误。例如，flask_app.conf 文件可能需要类似以下内容：ini\
``` bash 

'''例如 /etc/supervisor/conf.d/flask_app.conf '''
``` 

``` bash 
[program:flask_app]
command=/home/wangyaoqi/miniconda3/bin/python3 /home/wangyaoqi/container/haikang/app.py   #我的python目录  与运行文件目录 可以自行查
autostart=true
autorestart=true
stderr_logfile=/var/log/flask_app.err.log
stdout_logfile=/var/log/flask_app.out.log
``` 
然后，重新加载 supervisor 配置并重启应用：
``` bash

复制代码
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl restart flask_app
``` 
#### Suppervisor 日志查询方式
``` bash
sudo tail -f /var/log/flask_app.out.log
sudo tail -f /var/log/flask_app.err.log
``` 
