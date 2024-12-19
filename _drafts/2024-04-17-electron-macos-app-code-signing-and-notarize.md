---
title: Electron macOS 应用程序的代码签名和公证
title_en: 2024-04-17-electron-macos-app-code-signing-and-notarize
date: 2024-04-17 18:52:30
tags:
---

## 背景

> 签名 ✅ 公证 ✅

从 App Store 之外下载 macOS 应用程序时，经常会遇到应用已损坏和不可信的提示，这是由于应用程序未进行代码签名和公证导致的。

基于 [electron-builder](https://github.com/electron-userland/electron-builder) 的 Electron macOS 应用程序，想消除相关提示，让应用能顺畅的安装，需要进行代码签名和公证。

## 本地环境

- 安装证书
  - 从团队内具备 Apple 开发者计划会员资格的成员那获取 Developer ID 证书的 .p12 文件
    - 将 Developer ID 证书添加到钥匙串访问
    - 将 Developer ID G2 CA 中间证书添加到钥匙串访问
  - 或直接通过 XCode 下载 Developer ID 证书，需要具备 Apple 开发者计划会员资格
- 配置签名相关环境变量
  - CSC_LINK => 证书路径
  - CSC_KEY_PASSWORD => 证书密码
- 配置公证相关环境变量
  - APPLE_API_KEY
  - APPLE_API_KEY_ID
  - APPLE_API_ISSUER
- 检查项目中相关插件的配置
  - @electron/osx-sign
  - @electron/notarize
- 构建应用程序（插件根据配置进行代码签名和公证）
- 安装公证工具（手动公证、查看公证记录）

## 服务器环境

- 配置签名和公证相关环境变量（机密级）
  - BUILD_CERTIFICATE_BASE64 => 证书 base64 格式
  - P12_PASSWORD => 证书密码
  - KEYCHAIN_PASSWORD => 新建的钥匙串的密码
  - API_KEY_BASE64
  - APPLE_API_KEY_ID
  - APPLE_API_ISSUER
- 构建应用程序

## 参考链接

https://www.electron.build/code-signing
https://www.electronforge.io/guides/code-signing/code-signing-macos
https://appstoreconnect.apple.com/access/integrations/api
https://developer.apple.com/account/resources/certificates/list

## 常见问题

### 1. skipped macOS application code signing reason=cannot find valid "Developer ID Application" identity or custom non-Apple code signing certificate, it could cause some undefined behaviour, e.g. macOS localized description not visible, see https://electron.build/code-signing allIdentities= 1) D3977C3110DDE144616708C77183532CA53C9AF4 "Developer ID Application: Zhuo Xiao (72GE3Z7C6A)" (CSSMERR_TP_NOT_TRUSTED)

> 类似问题：Warning: unable to build chain to self-signed root for signer "Developer ID Application

#### 原因

1. 证书不正确
2. 缺少 Apple 中间证书

通过命令行查询本地安装的证书：

1. `security find-identity -vp codesigning`

```
➜ *********-******-*******-*** git:(test) certtool y | grep Developer\ I
   Common Name : Developer ID Certification Authority
   Common Name : Developer ID Application: **\* \*** (3W**\*\***KN)
   Common Name : Developer ID Certification Authority
   Common Name : Developer ID Certification Authority
   Common Name : Developer ID Application: **\* \*** (72**\*\***6A)

```

2. `certtool y | grep Developer\ I`

```
➜ *********-******-*******-*** git:(test) security find-identity -vp codesigning
1) 505D78...E31676 "Developer ID Application: **\* \*** (3W**\*\***KN)"
2) D3977C...3C9AF4 "Developer ID Application: **\* \*** (72**\*\***6A)"
   2 valid identities found
```

#### 解决方案

- 安装 Apple 中间证书
- 或删除证书后，通过 XCode 安装证书

确保本地安装的是 Developer ID Application 证书，以及对应安装了中间证书。
如果是通过 XCode 下载的证书，会自动安装中间证书；通过 .p12 文件安装的话，需要安装中间证书。

#### 参考链接

https://github.com/electron-userland/electron-builder/issues/2513
https://forums.developer.apple.com/forums/thread/86161
https://www.apple.com/certificateauthority/

### 2. Failed to display codesign info on your application with code: 1, **\***.app: code object is not signed at all

检查是否设置代码签名相关环境变量

- CSC_LINK => 证书路径
- CSC_KEY_PASSWORD => 证书密码

### 3. A JavaScript error occurred in the main process. Uncaught Exception: Error: dlopen(/var/folders/9n/gmncy9wx7xb4fyc0pwrfsfh40000gn/T/...

#### 原因

第三方库没有进行代码签名

#### 解决方案

将 com.apple.security.cs.disable-library-validation 设置为 true

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>com.apple.security.cs.allow-jit</key>
    <true/>
    <key>com.apple.security.cs.disable-library-validation</key>
    <true/>
  </dict>
</plist>
```

#### 参考链接

https://forums.developer.apple.com/forums/thread/129977
https://developer.apple.com/forums/thread/126895

### 4. Unexpected token 'E', "Error: HTT"... is not valid JSON failedTask=build stackTrace=SyntaxError: Unexpected token 'E', "Error: HTT"... is not valid JSON

检查网络是否通畅，公证会将应用程序上传到 Apple 在 AWS S3 的存储桶中
