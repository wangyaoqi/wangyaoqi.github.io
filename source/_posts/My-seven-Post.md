---
title: omu简易的企业音频播放模拟校园铃声
date: 2024-07-11 17:07:59
tags:
---

## 干嘛的
音频播放定时  加载音频播放列表
## 结构
musicproject
    - environment.yml
    - omu/
    │
    ├── app.py
    ├── requirements.txt
    ├── Dockerfile
    ├── templates/
    │   └── index.html
    └── audios/
      - app.py  audios  logs  __pycache__  requirements.txt  static  tasks.json  templates  test.py
      - root@wangyaoqi:/home/wangyaoqi/container/orangemusic/omu# ls
        app.py  audios  logs  __pycache__  requirements.txt  static  tasks.json  templates  test.py
        root@wangyaoqi:/home/wangyaoqi/container/orangemusic/omu# cd audios/
        root@wangyaoqi:/home/wangyaoqi/container/orangemusic/omu/audios# ls
        base.mp3  云烟成雨.mp3
        root@wangyaoqi:/home/wangyaoqi/container/orangemusic/omu/audios# cd ../logs/
        root@wangyaoqi:/home/wangyaoqi/container/orangemusic/omu/logs# ls
        flask.log
        root@wangyaoqi:/home/wangyaoqi/container/orangemusic/omu/logs# cd ../static/
        root@wangyaoqi:/home/wangyaoqi/container/orangemusic/omu/static# ls
        style.css
        root@wangyaoqi:/home/wangyaoqi/container/orangemusic/omu/static# cat ../tasks.json
        []root@wangyaoqi:/home/wangyaoqi/container/orangemusic/omu/static# cd ../templates
        root@wangyaoqi:/home/wangyaoqi/container/orangemusic/omu/templates# ls
        index.html
        root@wangyaoqi:/home/wangyaoqi/container/orangemusic/omu/templates# cat index.html 

### environment.yml
``` bash
name: omu-app-env  # 虚拟环境的名称，可以根据您的项目进行调整
channels:
  - defaults
dependencies:
  - python=3.8  # 指定 Python 的版本
  - flask  # Flask web 框架
  - pygame  # Pygame 音频处理库
  - gunicorn  # 生产环境的 WSGI HTTP 服务器
  - schedule  # 定时任务调度库
```

## 天气插件放入位置
![alt text](My-Five-Post/1.png)

## 项目文件内容
![alt text](My-Five-Post/2.png)

## 代码文件

### app.py 
``` bash

from flask import Flask, request, jsonify, render_template, send_from_directory
from threading import Thread
import os
import json
import logging
import schedule
import time
import pygame  # 导入pygame库

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = 'audios'
app.config['TASKS_FILE'] = 'tasks.json'

# 设置日志记录
if not os.path.exists('logs'):
    os.makedirs('logs')

logging.basicConfig(level=logging.DEBUG,
                    format='%(asctime)s %(levelname)s: %(message)s [in %(pathname)s:%(lineno)d]',
                    handlers=[
                        logging.FileHandler('logs/flask.log'),
                        logging.StreamHandler()
                    ])

# 初始化Pygame混音器
pygame.mixer.init()

# 确保上传文件的目录存在
if not os.path.exists(app.config['UPLOAD_FOLDER']):
    os.makedirs(app.config['UPLOAD_FOLDER'])

# 确保任务文件存在且包含有效的 JSON 数据
if not os.path.exists(app.config['TASKS_FILE']):
    with open(app.config['TASKS_FILE'], 'w') as f:
        json.dump([], f)

# 加载任务
try:
    with open(app.config['TASKS_FILE'], 'r') as f:
        tasks = json.load(f)
except json.JSONDecodeError:
    tasks = []
    with open(app.config['TASKS_FILE'], 'w') as f:
        json.dump(tasks, f)


def play_audio(filename):
   # print("laileme")
    file_path = os.path.join(app.config['UPLOAD_FOLDER'], filename)
    try:
        logging.debug(f"Attempting to play audio file: {file_path}")
        sound = pygame.mixer.Sound(file_path)
        sound.play()
        while pygame.mixer.Channel(0).get_busy():
            pygame.time.Clock().tick(10)
        logging.info(f"Finished playing {filename}")
    except pygame.error as e:
        logging.error(f"Error playing audio: {str(e)}")

def run_schedule():
    while True:
      #  logging.debug("Running scheduled tasks...")
        schedule.run_pending()
       # logging.debug(f"Current scheduled jobs: {schedule.get_jobs()}")
       # logging.debug("Schedule check completed.")
        time.sleep(1)


def save_tasks():
    with open(app.config['TASKS_FILE'], 'w') as f:
        json.dump(tasks, f)


@app.route('/')
def index():
    audio_files = os.listdir(app.config['UPLOAD_FOLDER'])
    return render_template('index.html', audio_files=audio_files, tasks=tasks)


@app.route('/upload', methods=['POST'])
def upload_file():
    try:
        if 'audio_file' not in request.files:
            logging.error("No file part")
            return jsonify(error="No file part"), 400

        file = request.files['audio_file']
        if file.filename == '':
            logging.error("No selected file")
            return jsonify(error="No selected file"), 400

        if file:
            filename = file.filename
            file_path = os.path.join(app.config['UPLOAD_FOLDER'], filename)
            file.save(file_path)
            logging.info(f"Uploaded {filename}")
            return jsonify(message=f"File {filename} uploaded successfully")
    except Exception as e:
        logging.error(f"Error: {str(e)}")
        return jsonify(error=str(e)), 500


@app.route('/schedule', methods=['POST'])
def schedule_task():
    try:
        data = request.json
        filename = data['filename']
        play_time = data['play_time']
        days = data['days']
        logging.debug(f"Scheduling task: {filename} at {play_time} on {days}")

        for day in days:
            if day == "Monday":
                schedule.every().monday.at(play_time).do(play_audio, filename)
            elif day == "Tuesday":
                schedule.every().tuesday.at(play_time).do(play_audio, filename)
            elif day == "Wednesday":
                schedule.every().wednesday.at(play_time).do(play_audio, filename)
            elif day == "Thursday":
                schedule.every().thursday.at(play_time).do(play_audio, filename)
            elif day == "Friday":
                schedule.every().friday.at(play_time).do(play_audio, filename)
            elif day == "Saturday":
                schedule.every().saturday.at(play_time).do(play_audio, filename)
            elif day == "Sunday":
                schedule.every().sunday.at(play_time).do(play_audio, filename)

        tasks.append({"filename": filename, "play_time": play_time, "days": days})
        save_tasks()
        logging.info(f"Scheduled {filename} to play at {play_time} on {', '.join(days)}")
        logging.debug(f"Current scheduled jobs: {schedule.get_jobs()}")
        return jsonify(message=f"Task scheduled successfully")
    except Exception as e:
        logging.error(f"Error: {str(e)}")
        return jsonify(error=str(e)), 500


@app.route('/delete_task', methods=['POST'])
def delete_task():
    try:
        task_id = request.json['task_id']
        if 0 <= task_id < len(tasks):
            tasks.pop(task_id)
            save_tasks()
            return jsonify(message="Task deleted successfully")
        else:
            return jsonify(error="Invalid task ID"), 400
    except Exception as e:
        logging.error(f"Error: {str(e)}")
        return jsonify(error=str(e)), 500


@app.route('/audios/<filename>')
def get_audio(filename):
    return send_from_directory(app.config['UPLOAD_FOLDER'], filename)






if __name__ == '__main__':
    # 启动定时任务线程

    # 启动 Flask 应用
    schedule_thread = Thread(target=run_schedule, daemon=True)
    schedule_thread.start()
    app.run(host='0.0.0.0', port=5000, debug=False)
#

# play_audio("base.mp3")
```
### index.html
``` bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>OMU: 音频播放管理系统</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
    <link rel="stylesheet" href="/static/style.css">
</head>
<body>
<div class="container mt-5">
    <h1 class="text-center mb-4">OMU: 音频播放管理系统</h1>
    <div class="card mb-4">
        <div class="card-header">
            上传音频文件
        </div>
        <div class="card-body">
            <form id="uploadForm" enctype="multipart/form-data">
                <input type="file" name="audio_file" accept=".mp3" required>
                <button type="submit">上传</button>
            </form>
        </div>
    </div>
    <div class="row">
        <div class="col-half">
            <div class="card mb-4">
                <div class="card-header">
                    设置任务
                </div>
                <div class="card-body">
                    <form id="scheduleForm">
                        <div class="form-group">
                            <label for="filename">选择音频文件：</label>
                            <select name="filename" id="filename" class="form-control" required>
                                {% for audio in audio_files %}
                                    <option value="{{ audio }}">{{ audio }}</option>
                                {% endfor %}
                            </select>
                        </div>
                        <div class="form-group">
                            <label for="play_time">播放时间：</label>
                            <input type="time" name="play_time" id="play_time" class="form-control" required>
                        </div>
                        <div class="form-group">
                            <label for="days">选择播放日期：</label>
                            <select name="days" id="days" class="form-control" multiple required>
                                <option value="Monday">星期一</option>
                                <option value="Tuesday">星期二</option>
                                <option value="Wednesday">星期三</option>
                                <option value="Thursday">星期四</option>
                                <option value="Friday">星期五</option>
                                <option value="Saturday">星期六</option>
                                <option value="Sunday">星期日</option>
                            </select>
                        </div>
                        <button type="submit" class="btn btn-primary">设置任务</button>
                    </form>
                </div>
            </div>
        </div>
        <div class="col-half">
            <div class="card mb-4">
                <div class="card-header">
                    当前任务
                </div>
                <div class="card-body">
                    <ul id="taskList" class="list-group">
                        {% for task in tasks %}
                            <li class="list-group-item">
                                {{ task.filename }} - {{ task.play_time }} - {{ task.days | join(", ") }}
                                <button class="btn btn-danger btn-sm" onclick="deleteTask({{ loop.index0 }})">删除
                                </button>
                            </li>
                        {% endfor %}
                    </ul>
                </div>
            </div>
        </div>
    </div>
    <div class="card mb-4">
        <div class="card-header">
            当前音频文件
        </div>
        <div class="card-body">
            <ul id="audioList" class="list-group">
                {% for audio in audio_files %}
                    <li class="list-group-item">
                        <a href="/audios/{{ audio }}" target="_blank">{{ audio }}</a>
                    </li>
                {% endfor %}
            </ul>
        </div>
    </div>
</div>
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
<script>
    $(document).ready(function () {
        $('#uploadForm').on('submit', function (e) {
            e.preventDefault();
            var formData = new FormData(this);
            $.ajax({
                url: '/upload',
                type: 'POST',
                data: formData,
                processData: false,
                contentType: false,
                success: function (response) {
                    alert(response.message);
                    location.reload();
                },
                error: function (xhr) {
                    alert(xhr.responseJSON.error);
                }
            });
        });

        $('#scheduleForm').on('submit', function (e) {
            e.preventDefault();
            var data = {
                filename: $('#filename').val(),
                play_time: $('#play_time').val(),
                days: $('#days').val()
            };
            $.ajax({
                url: '/schedule',
                type: 'POST',
                contentType: 'application/json',
                data: JSON.stringify(data),
                success: function (response) {
                    alert(response.message);
                    location.reload();
                },
                error: function (xhr) {
                    alert(xhr.responseJSON.error);
                }
            });
        });
    });

    function deleteTask(taskId) {
        $.ajax({
            url: '/delete_task',
            type: 'POST',
            contentType: 'application/json',
            data: JSON.stringify({task_id: taskId}),
            success: function (response) {
                alert(response.message);
                location.reload();
            },
            error: function (xhr) {
                alert(xhr.responseJSON.error);
            }
        });
    }
</script>
</body>
</html>

```
### requirements.txt
``` bash
Flask
schedule
```
### test.py 
``` bash
    root@wangyaoqi:/home/wangyaoqi/container/orangemusic/omu# cat test.py 
    import pygame  
    import sys  
    
    # 初始化pygame的mixer模块  
    pygame.mixer.init()  
    
    # 加载音频文件  
    # 注意：你需要将'your_audio_file.mp3'替换为你的音频文件的实际路径  
    sound = pygame.mixer.Sound('base.mp3')  
    
    # 播放音频  
    # 可选参数：loops=0 表示播放一次，loops=-1 表示无限循环播放  
    sound.play()  
    
    # 等待音频播放完成  
    # 注意：pygame.mixer.Channel.get_busy() 方法用于检查当前通道是否正在播放  
    while pygame.mixer.Channel(0).get_busy():  
        pygame.time.Clock().tick(10)  # 简单的延迟来避免阻塞  
    
    # 退出pygame  
    pygame.quit()  
    sys.exit()
```

## 如何构建
``` bash
    root@wangyaoqi:/home/wangyaoqi/container/nodockerconda# supervisorctl status
    omu                              RUNNING   pid 20454, uptime 47 days, 20:31:13
    
    root@wangyaoqi:/etc/supervisor# ls
conf.d  supervisord.conf
root@wangyaoqi:/etc/supervisor# cat supervisord.conf 
; supervisor config file

[unix_http_server]
file=/var/run/supervisor.sock   ; (the path to the socket file)
chmod=0700                       ; sockef file mode (default 0700)

[supervisord]
logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
childlogdir=/var/log/supervisor            ; ('AUTO' child log dir, default $TEMP)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///var/run/supervisor.sock ; use a unix:// URL  for a unix socket

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

[include]
files = /etc/supervisor/conf.d/*.conf
root@wangyaoqi:/etc/supervisor# cd /etc/supervisor/conf.d/
root@wangyaoqi:/etc/supervisor/conf.d# ls
omu.conf
root@wangyaoqi:/etc/supervisor/conf.d# cat omu.conf 
[program:omu]
command=/home/wangyaoqi/miniconda3/envs/omu/bin/python /home/wangyaoqi/container/orangemusic/omu/app.py
directory=/home/wangyaoqi/container/orangemusic/omu
autostart=true
autorestart=true
stderr_logfile=/var/log/omu.err.log
stdout_logfile=/var/log/omu.out.log
user=wangyaoqi
environment=HOME="/home/wangyaoqi",USER="wangyaoqi"


```