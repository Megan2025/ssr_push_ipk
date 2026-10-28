name: build ssr+ arm_a53+x86

on:
  workflow_dispatch:

jobs:
  build:
    name: Test ${{ matrix.arch }}
    runs-on: ubuntu-22.04
    strategy:
      fail-fast: false
      matrix:
        include:
          - arch: aarch64_cortex-a53
          - arch: x86_64

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup custom feed for SSR+
        run: |
          mkdir -p custom_feed
          # 拉取包含 luci-app-ssr-plus 的完整源码
          git clone --depth=1 https://github.com/fw876/helloworld.git custom_feed/helloworld

      - name: Build
        uses: immortalwrt/gh-action-sdk@v4.1
        env:
          ARCH: ${{ matrix.arch }}-openwrt-23.05
          FEEDNAME: custom_feed
          PACKAGES: luci-app-ssr-plus
          V: s

      - name: Move created packages to project dir
        run: |
          # 严格筛选：排除 -src 源码包，只复制 SSR+ 及其配套的核心插件与协议组件
          find bin/packages/ -type f -name "*.ipk" ! -name "*-src_*.ipk" | grep -E "(luci-app-ssr-plus|luci-i18n-ssr-plus|chinadns-ng|dns2socks|dns2tcp|shadowsocks-libev|shadowsocks-rust|simple-obfs|tcping|xray-core|v2ray|trojan|lua-neturl|microsocks)" | xargs -I {} cp -f {} . || true

      - name: Collect metadata
        run: |
          COMMIT_ID=$(git rev-parse --short HEAD)
          echo "ARCHIVE_NAME=${{ matrix.arch }}-$COMMIT_ID" >> $GITHUB_ENV

      - name: Generate metadata
        run: |
          cat << _EOF_ > PKG-INFO
          Name: ${{ env.ARCHIVE_NAME }}
          Platform: ${{ matrix.arch }}
          Package: luci-app-ssr-plus
          Full file listing:
          _EOF_
          ls -al *.ipk >> PKG-INFO 2>/dev/null || true
          cat PKG-INFO

      - name: Store packages
        uses: actions/upload-artifact@v4
        with:
          name: ${{ env.ARCHIVE_NAME }}-packages
          path: |
            *.ipk
            PKG-INFO
