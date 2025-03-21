---
title: electron-tool-study
date: 2024-12-05 15:55:50
tags:
---
#### Electron 应用 Windows 与 macOS 协议唤起区别及单实例处理文档
1. 协议唤起机制的区别
1.1 Windows
注册协议： Windows 系统中，注册自定义协议通常通过在安装程序中修改注册表实现。

plaintext
复制代码
[HKEY_CLASSES_ROOT\your-protocol]
@="URL:Your Protocol"
"URL Protocol"=""
[HKEY_CLASSES_ROOT\your-protocol\shell\open\command]
@="\"C:\\Path\\To\\YourApp.exe\" \"%1\""
%1 是传递的协议参数。
Windows 启动时会通过 process.argv 获取协议中的参数。
多实例问题： Windows 的 process.argv 中会包含协议参数。但默认情况下，每次协议唤起都会创建一个新实例。

解决办法： 使用 app.requestSingleInstanceLock()，监听 second-instance 事件处理。

javascript
复制代码
const gotTheLock = app.requestSingleInstanceLock();

if (!gotTheLock) {
  app.quit();
} else {
  app.on('second-instance', (event, commandLine, workingDirectory) => {
    if (mainWindow) {
      if (mainWindow.isMinimized()) mainWindow.restore();
      mainWindow.focus();
    }
    const protocolArg = commandLine.find(arg => arg.startsWith('your-protocol://'));
    if (protocolArg) handleProtocol(protocolArg);
  });
}
1.2 macOS
注册协议： macOS 使用 Info.plist 文件注册协议。

xml
复制代码
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLName</key>
    <string>your-protocol</string>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>your-protocol</string>
    </array>
  </dict>
</array>
协议唤起：

macOS 的协议通过 open-url 事件触发。
传递的 URL 作为参数，通过 event 和 url 获取。
代码示例：

javascript
复制代码
app.on('open-url', (event, url) => {
  event.preventDefault();
  handleProtocol(url);
});
多实例问题： macOS 默认不会启动多个实例，但仍需要用 requestSingleInstanceLock 确保跨平台一致性。

2. 单实例处理
2.1 核心代码
使用 app.requestSingleInstanceLock 确保应用是单实例。

javascript
复制代码
const gotTheLock = app.requestSingleInstanceLock();

if (!gotTheLock) {
  app.quit(); // 如果已存在实例，退出当前实例
} else {
  app.on('second-instance', (event, commandLine, workingDirectory) => {
    if (mainWindow) {
      if (mainWindow.isMinimized()) mainWindow.restore();
      mainWindow.focus();

      // 处理通过协议传递的参数
      const protocolArg = commandLine.find(arg => arg.startsWith('your-protocol://'));
      if (protocolArg) handleProtocol(protocolArg);
    }
  });
}
2.2 与协议唤起结合
Windows：
javascript
复制代码
app.on('ready', () => {
  const args = process.argv.slice(1);
  const protocolArg = args.find(arg => arg.startsWith('your-protocol://'));
  if (protocolArg) handleProtocol(protocolArg);
});
macOS：
javascript
复制代码
app.on('open-url', (event, url) => {
  event.preventDefault();
  handleProtocol(url);
});
3. Icon 生成的 Bash 脚本
3.1 macOS 的 icns 文件生成
macOS 使用 .icns 文件作为应用图标。

要求尺寸：

推荐使用 1024x1024 的 PNG 文件作为输入。
需要生成以下尺寸：16x16, 32x32, 128x128, 256x256, 512x512, 1024x1024。
脚本示例：

bash
复制代码
#!/bin/bash
# 将 1024x1024 的 PNG 文件转换为 ICNS 格式
ICONSET_NAME="AppIcon.iconset"
INPUT_IMAGE="icon.png"
OUTPUT_ICNS="AppIcon.icns"

mkdir $ICONSET_NAME
sips -z 16 16     $INPUT_IMAGE --out $ICONSET_NAME/icon_16x16.png
sips -z 32 32     $INPUT_IMAGE --out $ICONSET_NAME/icon_16x16@2x.png
sips -z 32 32     $INPUT_IMAGE --out $ICONSET_NAME/icon_32x32.png
sips -z 64 64     $INPUT_IMAGE --out $ICONSET_NAME/icon_32x32@2x.png
sips -z 128 128   $INPUT_IMAGE --out $ICONSET_NAME/icon_128x128.png
sips -z 256 256   $INPUT_IMAGE --out $ICONSET_NAME/icon_128x128@2x.png
sips -z 256 256   $INPUT_IMAGE --out $ICONSET_NAME/icon_256x256.png
sips -z 512 512   $INPUT_IMAGE --out $ICONSET_NAME/icon_256x256@2x.png
sips -z 512 512   $INPUT_IMAGE --out $ICONSET_NAME/icon_512x512.png
sips -z 1024 1024 $INPUT_IMAGE --out $ICONSET_NAME/icon_512x512@2x.png
iconutil -c icns $ICONSET_NAME -o $OUTPUT_ICNS
rm -r $ICONSET_NAME
3.2 Windows 的 ico 文件生成
Windows 使用 .ico 文件作为应用图标。

要求尺寸：

常见的尺寸为 16x16, 32x32, 48x48, 256x256。
脚本示例：

bash
复制代码
#!/bin/bash
# 将 PNG 文件转换为 ICO 格式
INPUT_IMAGE="icon.png"
OUTPUT_ICO="icon.ico"

convert $INPUT_IMAGE -define icon:auto-resize=16,32,48,256 $OUTPUT_ICO
需要确保安装了 ImageMagick：

bash
复制代码
brew install imagemagick # macOS
sudo apt install imagemagick # Linux
4. 总结与建议
协议唤起：

Windows 和 macOS 的注册和处理机制不同，但可以通过 app.requestSingleInstanceLock 统一处理多实例问题。
针对 Windows，需解析 process.argv。
针对 macOS，需监听 open-url。
单实例问题：

使用 app.requestSingleInstanceLock 确保跨平台一致性。
使用 second-instance 事件响应协议唤起的实例请求。
图标生成：

macOS 使用 .icns 文件，推荐使用 1024x1024 的 PNG 文件并按比例生成所需尺寸。
Windows 使用 .ico 文件，生成常见的 16x16, 32x32, 48x48, 256x256 尺寸。