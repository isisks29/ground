# ground
课程
本节课，我为你们提供了2个保护系数较高的dylib授权靶场，两个靶场dylib几乎完全不一样，你们要任选其一，你们需要自行分析，尽量整合所有信息，并且可以借助AI
如果你能自己编译一个dylib来绕过靶场dylib的验证系统，并使该dylib的功能直接可用，我将替你申请升班资格。
每个人选完之后联系我，我会给予你这个靶场dylib的原ipa，其实这两个靶场都是带授权验证的 iOS 游戏 mod 菜单框架，且在自己的卡密验证逻辑上各有千秋，所以你们可不要掉以轻心啊，很难的！我都花了半天功夫！
建议你们多使用一些辅助工具。strongarm下载指令为：
git clone https://github.com/datatheorem/strongarm.git
cd strongarm
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip setuptools wheel 'pip-tools<7.0.0'
pip-sync requirements.txt requirements-dev.txt
ghidra的话，请自行用命令行下载适配版本。

x64DBG也很厉害的，但是你们也要自行下载适配版本。

给你们讲讲这两个靶场。签名安装带有ace或Zhuanz的游戏ipa后，进入游戏会有弹窗，要求输入卡密。卡密都是插件制作者及其代理员批量生成的，并且需要在特定网店购买，我相信你们不会傻到去购买它。如果成功激活了，若是ace，则需要点击左上角，然后就会显现功能面板；若是Zhuanz，则在成功激活后会显示一个悬浮球，悬浮球的样式也在仓库中。


目标：基础差的同学，可以进给我一份分析报告；有余力的同学，可以自行开发dylib，并达到这样的效果：将这个作业dylib注入到有ace或Zhuanz的ipa并签名安装后，原靶场文件的卡密系统被攻破，使得里面的功能正常可用。

截止时间：10月1日



有的同学身边可能没电脑，可以借助github的actions，yaml样本给你们


name: Build iOS dylib

on:
  push:
    branches: [ "main", "master" ]
  workflow_dispatch:
    inputs:
      source:
        description: "要编译的 ObjC 源文件(仓库内相对路径)"
        required: false
        default: "bypass.m"
      xcode:
        description: "Xcode 版本(留空用默认, 如 16.4)"
        required: false
        default: ""

jobs:
  # ---------- job 1: 编译 bypass.m -> bypass.dylib ----------
  build-bypass:
    name: Compile bypass.dylib (iphoneos arm64)
    runs-on: macos-latest
    timeout-minutes: 15
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Select Xcode
        if: ${{ inputs.xcode != '' }}
        run: sudo xcode-select -s /Applications/Xcode_${{ inputs.xcode }}.app

      - name: Verify iOS SDK / clang
        run: |
          xcrun --sdk iphoneos --show-sdk-path
          xcrun --sdk iphoneos clang --version | head -n1

      - name: Compile dynamic library (arm64, ObjC, ARC)
        run: |
          SRC="${{ inputs.source != '' && inputs.source || 'bypass.m' }}"
          test -f "$SRC" || { echo "source not found: $SRC"; exit 1; }
          case "$SRC" in
            *.m)  EXTRA="-fobjc-arc" ;;
            *.c)  EXTRA="" ;;
            *)    EXTRA="" ;;
          esac
          xcrun --sdk iphoneos clang \
            -arch arm64 \
            -dynamiclib \
            -O2 \
            -framework Foundation \
            -framework UIKit \
            -fobjc-arc \
            $EXTRA \
            -o bypass.dylib \
            "$SRC"
          ls -lh bypass.dylib
          file bypass.dylib

      - name: Ad-hoc sign
        run: codesign --force --sign - bypass.dylib

      - name: Upload bypass.dylib
        uses: actions/upload-artifact@v4
        with:
          name: bypass.dylib
          path: bypass.dylib
          if-no-files-found: error




失败了不要气馁！你们每个人都是大编程师！
