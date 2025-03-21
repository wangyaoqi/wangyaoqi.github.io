---
title: Electron 打包及应用签名指南
date: 2024-11-25 18:44:21
tags:
---
#### electron Electron 打包及应用签名指南

##### 本文档介绍了如何使用 Electron 打包应用并为应用进行签名，以确保应用的安全性和可信度。

1. 准备工作
在开始之前，请确保你已安装以下软件和工具：

Node.js（推荐使用 LTS 版本）
Electron
Electron Builder（用于打包应用）
代码签名证书
2. 创建 Electron 应用
首先，创建一个新的 Electron 项目，或者在现有项目中进行打包。

初始化项目
在项目根目录下执行以下命令来初始化一个新的 Node.js 项目：

bash
复制代码
npm init -y
安装 Electron 和 Electron Builder
使用以下命令安装 Electron 和 Electron Builder：

bash
复制代码
npm install electron --save-dev
npm install electron-builder --save-dev
创建基础文件结构
确保你的项目中有以下结构：

scss
复制代码
your-project/
├── package.json
├── main.js  (Electron 主进程文件)
├── index.html (应用入口页面)
├── src/
│   └── app.js
└── build/
    └── config.json
3. 配置打包设置
在 package.json 中配置打包相关设置。你需要在 build 部分添加一些参数来指定如何打包应用。

json
复制代码
{
  "name": "your-app-name",
  "version": "1.0.0",
  "description": "A simple Electron app",
  "main": "main.js",
  "scripts": {
    "start": "electron .",
    "pack": "electron-builder --dir",
    "dist": "electron-builder"
  },
  "build": {
    "appId": "com.yourcompany.yourapp",
    "mac": {
      "category": "public.app-category.productivity",
      "icon": "assets/icons/mac/icon.icns"
    },
    "win": {
      "target": "nsis"
    }
  },
  "devDependencies": {
    "electron": "^xx.xx.xx",
    "electron-builder": "^xx.xx.xx"
  }
}
appId：设置应用的唯一标识符，通常使用反向域名命名。
mac.icon：指定 macOS 应用图标路径。
win.target：设置 Windows 打包目标格式（例如，NSIS 安装程序）。
4. 打包应用
执行以下命令来打包应用：

bash
复制代码
npm run dist
此命令会根据你的配置生成打包好的安装包。它会生成适合你目标操作系统的文件（例如 .dmg 或 .exe）。

5. 应用签名
在许多操作系统中，尤其是 macOS 和 Windows，应用签名是非常重要的，它能确保用户能够安全地安装和使用你的应用。以下是如何进行应用签名的步骤。

5.1 macOS 应用签名
在 macOS 上，你需要使用 Apple Developer Program 提供的签名证书来对应用进行签名。具体步骤如下：

申请开发者证书
前往 Apple Developer 网站，加入开发者计划，并申请适用于 macOS 应用的签名证书。

配置签名证书
在 Xcode 中，选择你的证书并配置到你的应用项目中。在 package.json 中的 build.mac.identity 部分，配置你的证书身份：

json
复制代码
"mac": {
  "identity": "Developer ID Application: Your Name (Team ID)",
  "category": "public.app-category.productivity",
  "icon": "assets/icons/mac/icon.icns"
}
执行签名操作
执行以下命令对 macOS 应用进行签名：

bash
复制代码
electron-builder --mac --x64 --sign
5.2 Windows 应用签名
在 Windows 上，你需要申请一个代码签名证书。可以从 Digicert, Comodo, 或其他可信的证书机构申请证书。

申请签名证书
从受信任的证书机构申请代码签名证书。

配置签名证书
将证书文件安装到你的计算机上，并配置 Electron Builder 使用它进行签名。可以在 package.json 中的 win 部分配置签名证书：

json
复制代码
"win": {
  "target": "nsis",
  "sign": "./path/to/certificate.pfx"
}
执行签名操作
在 Windows 上执行签名操作，使用以下命令进行签名：

bash
复制代码
electron-builder --win --x64 --sign
6. 测试应用
在应用打包和签名完成后，建议进行以下操作：

在不同操作系统上测试应用的安装和运行。
确认应用是否被操作系统识别为已签名且可信。
检查安装过程中的任何警告或错误。
7. 发布应用
当你完成了打包和签名工作后，你可以将应用上传到各种应用商店（如 macOS 的 Mac App Store）或通过你自己的渠道分发（如官方网站）。

8. 常见问题
8.1 打包时遇到 "Error: Invalid signature" 错误
如果你在打包时遇到签名错误，确保你使用的是有效的签名证书，并且配置文件中的证书路径正确。

8.2 macOS 安装时出现 "未认证的开发者" 错误
确保你使用的是有效的开发者签名证书。如果签名未正确配置，macOS 会显示此警告。你可以在 Xcode 中重新检查你的证书和配置。

9. 结论
通过使用 Electron 和 Electron Builder，你可以轻松打包和签名 Electron 应用，确保你的应用在用户设备上安全运行并符合操作系统的安全要求。