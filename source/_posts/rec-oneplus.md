---
title: 一加系列手机刷机记录
date: 2025-11-16 08:43:23
categories: 折腾记录
---

记录下折腾过程中遇到的问题，供未来参考．

## 原厂 ROM 下载

稍微搜索一下就可以找到 [这个网站](https://yun.daxiaamu.com/OnePlus_Roms/)，不过如果下载方式符合直觉，就没有单开一节记录的必要了．

事实上你点进上述网站提供的链接后会报 403，然后再逛逛 xda 就可以找到一个叫做 `FirmwareDownloader_English_by_NeFeroN.rar` 的文件，尝试一下，发现将上述网站中的链接粘贴进去，是可以下载的．

这是怎么一回事呢？检查压缩包里的文件可以发现，批处理脚本中这样调用了 `bin.exe`：

```bat
bin.exe -x16 -s16 --continue "%url%"
```

命令行参数看起来有点眼熟！这不是我们 aria2 吗．于是直接使用 aria2 下载，发现可行．

这就有点诡异了，再使用 curl 下载，同样不会报 403．

于是怀疑网站对 UA 做了判断，不允许浏览器访问．删除浏览器 UA 的 `Mozilla` 前缀后发现不再报 403，有点搞．

写这一节的时候我还搜到了 [这个仓库](https://github.com/spike0en/oneplus_archive)，不过这里只上传了比较新的 ROM．

## ocdt 分区

据说自 ColorOS 14 以来，一加的 bootloader 会校验 ocdt 分区，此时刷入其他人提供的 ocdt 会导致无法进入 fastboot 模式．如果你像我一样在救砖过程中进行了这个操作，那么唯一让机器再次开机的方法是去售后．不过我这里的售后给我免费刷了，不赖．

不过，如果你可以正常进入 fastboot**d** 模式的话，可以刷入 ColorOS 13 的 abl 分区来回避此问题．我在 `ColorOS PHB110_15.0.0.850(CN01)` 版本下尝试刷入了 `ColorOS PHB110_13.1.0.196(CN01)` 的 abl 分区，能够正常工作，不过 ota 更新会稳定失败一次，且每次 ota 更新都会刷掉 abl 分区．

如何预防必须前往售后的惨剧发生？鉴于本人技术能力有限，未能成功使用网络上流传的 9008 工具进行全量备份，我能够建议的方案是拿到手机后立刻 root，通过 [这篇文章](https://droidwin.com/how-to-backup-ocdt-and-persist-partition-on-oneplus-12-12r/) 中提到的方式备份 ocdt 和其他重要分区．如果你的 ocdt 分区已丢失，**永远不要回锁**，并保持至少一个槽位中的 abl 分区是 ColorOS 13 的．

## oplusstanvbk 分区

注意，由于本人的一加 11 已出手，本节内容未经实践，仅作记录．

直接跟随 LineageOS wiki 上的 [安装流程](https://wiki.lineageos.org/devices/salami/install/)，最后一步重启时会卡 logo．经过一番搜索，[这篇帖子](https://xdaforums.com/t/rom-discontinued-lineageos-21-for-oneplus-11.4640224/post-89361281) 中提到刷入 ColorOS 对应版本的 oplusstanvbk 分区能够解决问题，不知为何内容没有进入 wiki．

## Ace 3 的诡异相机问题

我的一加 Ace 3 在 `ColorOS PJE110_15.0.0.860(CN01)` 版本下刷入了 LineageOS 23.0，发现相机的录像模式存在以下问题：

1. 选中 1080p 模式并切换到 .9x 相机，会导致相机应用闪退．
2. 不开启 HLG 时，所有相机画面都有显著畸变．
3. 60fps 下，画面有奇怪的抖动．

[这个帖子](https://xdaforums.com/t/rom-official-lineageos-23-weeklies-for-oneplus-12r-ace-3.4669911/post-89899918) 中提到了问题 1，适配者的回复是他固件版本有问题．尽管没有近一步的回复，我仍然尝试跟随 LineageOS wiki 上的 [指南](https://wiki.lineageos.org/devices/salami/fw_update/)，在所有槽位中重新刷入了 `ColorOS PJE110_15.0.0.860(CN01)` 的固件，问题没有被解决．

[这个帖子](https://xdaforums.com/t/rom-16-official-pixelos-for-oneplus-12r-ace3-aosp-22-11-25.4662225/post-90114331) 中提到了问题 3，没有得到任何回复．

此外我还看到零星的回复反馈相机存在问题，由于没有日志或者视频，此处略过．

目前的解决方案是将相机降到 720p 24fps，开启 HLG 使用，鉴于 .9x 是微距镜头，在视频摄制中几乎不会使用，故最严重的问题 1 可以规避．

此外我还注意到上次 ota 后，控制中心中的相机权限总开关消失，这让我继续怀疑两个槽位中的固件版本不一致，尽管我进行的操作应当能够规避此问题．~~本周日应当还有一次 ota，再更新一次就知道了．~~ ota 后问题未出现，推测错了．

后面可能会考虑刷入最新固件再次尝试，不过我对解决问题不抱希望．
