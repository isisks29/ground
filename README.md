# ground
课程
本节课，我为你们提供了2个保护系数较高的dylib授权靶场，两个靶场dylib几乎完全不一样，你们要任选其一，你们需要自行分析，尽量整合所有信息，并且可以借助AI。
每个人下发的任务都不一样，但是属于同系列。比如说，我给A的靶场dylib是ace19、Zhuanz29，那么给B的靶场就可能是ace2、Zhuanz19。所以你们抄袭不了，但是可以向同学借鉴，这也算提升自己了。
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
我知道你们都是linux系统，所以如果要用到frida，那么它就在仓库里。
x64DBG也很厉害的，但是你们也要自行下载适配版本。

给你们讲讲这两个靶场。签名安装带有ace或Zhuanz的游戏ipa后，进入游戏会有弹窗，要求输入卡密。卡密都是插件制作者及其代理员批量生成的，并且需要在特定网店购买，我相信你们不会傻到去购买它。如果成功激活了，若是ace，则需要点击左上角，然后就会显现功能面板；若是Zhuanz，则在成功激活后会显示一个悬浮球，悬浮球的样式也在仓库中。我可以给你们一些提示：
zhuanz 密钥与算法

核心密钥 — __ckmask section（0x1176d0，64字节）：

• AES-256 密钥（前32字节）：58fd32abb393075c606a24b1c9c10071492b42c793e8161fabe516e0cae1cfe7

• HMAC-SHA256 密钥（后32字节）：cbab2edb13a8eb70386155033fa01398f91feead58d650783650d703e2fcbade

加密算法：AES-CBC、AES-GCM、HMAC-SHA256、DES（旧版兼容）、ECDSA 签名验证（secp256r1）

验证流程：LoadActivation → DeriveKeyFromParts → HmacSha256 → AesCbcDec → SecKeyVerifyOnce（ECDSA验签）→ ActivateSeal → PersistActivation

反 hook 机制：AuthCore +isFeatureEnabled 开头有代码完整性校验（bss字节 XOR __cqcici2 段字节），另有调用次数限制计数器（0x125a8a8）。

嵌入资源：10+ 个 UnityFS 加密资源包 + 一个 ELF 可执行文件（__fghong 段）。项目名 bsphp。


ace 密钥与算法

核心密钥 — +d2 函数硬编码 XOR 解密：

• 加密字节：39 0c 95 f8 df 22

• XOR 密钥：5a 7f a4 c9 ee 13

• 解密结果：cs1111（6字节，传入 objc_msgSend，length=6, type=4）

加密工具类 _0xA6C1F894：

• +d0 → 设备指纹生成（UIDevice + UIScreen bounds/scale → 格式化字符串）

• +d1 → 调用 d0 获取指纹，从字典查询/解密

• +d2 → 用 cs1111 密钥做校验

当然由于版本不同，我和你们的信息可能不一样，请谨慎甄别。

目标：基础差的同学，可以进给我一份分析报告；有余力的同学，可以自行开发dylib，并达到这样的效果：将这个作业dylib注入到有ace或Zhuanz的ipa并签名安装后，原靶场文件的卡密系统被攻破，是的里面的功能正常可用。

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
