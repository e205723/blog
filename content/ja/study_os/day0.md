---
title: "OS自作入門0日目"
description:
publishdate: 2021-04-25T00:17:57+09:00
---

## MacでOSを自作する

「[30日でできる! OS自作入門](https://www.amazon.co.jp/30%E6%97%A5%E3%81%A7%E3%81%A7%E3%81%8D%E3%82%8B-OS%E8%87%AA%E4%BD%9C%E5%85%A5%E9%96%80-%E5%B7%9D%E5%90%88-%E7%A7%80%E5%AE%9F-ebook/dp/B00IR1HYI0/ref=sr_1_1?adgrpid=103879434706&dchild=1&gclid=CjwKCAjwg4-EBhBwEiwAzYAlsjAlULfFE38PlApuNzYealG_zn3f-wXh9x9xsqYVI0tfga1SS-0gBxoCC5oQAvD_BwE&hvadid=448588871691&hvdev=c&hvlocphy=9053273&hvnetw=g&hvqmt=e&hvrand=12615786351100528979&hvtargid=kwd-525591903592&hydadcr=27267_11561158&jp-ad-ap=0&keywords=30%E6%97%A5%E3%81%A7%E3%81%A7%E3%81%8D%E3%82%8B+os%E8%87%AA%E4%BD%9C%E5%85%A5%E9%96%80&qid=1619277691&sr=8-1)」という本を使って自作OSの勉強をしようかと思います。本来この本はWindows環境向けなのですが、macOS用の環境を作ってしかも公開してくれている方がいるので、その方の[レポジトリ](https://github.com/tatsumack/30nichideosjisaku)を使ってこの本を読むにあたっての環境を構築したいと思います。

### 自分のMacBookの環境
```
$sw_vers

ProductName:	macOS
ProductVersion:	11.2.2
BuildVersion:	20D80
```

---

### 手順に従って環境を構築します

レポジトリのクローン
```
git clone https://github.com/tatsumack/30nichideosjisaku
```

[qemu](https://www.weblio.jp/content/qemu)のinstall
```
brew install qemu
```

qemuのバージョン確認
```
$qemu-system-i386 -version

QEMU emulator version 5.2.0
Copyright (c) 2003-2020 Fabrice Bellard and the QEMU Project developers
```

```
$ cd 30nichideosjisaku/01_day/helloos0/
$ make run
```

動作確認
```
$cd 30nichideosjisaku/01_day/helloos0/
$make run
```

このようなWindowが出れば動作確認成功です

<img src="/images/day0/console.jpg">

