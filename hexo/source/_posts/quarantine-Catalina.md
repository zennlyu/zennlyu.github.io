---
title: Catalina quarantine for corrupted apps
categories: [devtools]
tags: [devtools]

---

低于 `Big Sur` 系统版本的 `macOS` 或 `m1` 芯片机器在安装一些未适配软件时候可能会出现 `安装包损坏, 打不开“XXX”,因为它来自身份不明的开发者` 等报错，在此状态下可以尝试以下做法。

```shell
sudo spctl --master-disable
sudo xattr -r -d com.apple.quarantine /Applications/Sketch.app/
// 输入 sudo xattr -r -d com.apple.quarantine 后将应用程序拖入 teminal 中
```

（以及 Catalina 真的好耐看啊！）
