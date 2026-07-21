# Macast 开发指南

在 Windows 和 Linux 上，我们使用 **pystray** 实现菜单栏图标支持，使用 **pyinstaller** 打包应用。
macOS 则使用 **rumps** 和 **py2app**，性能更好且打包体积更小。

> **Apple Silicon (M 系列) 用户请注意**：macOS 构建已默认支持 Apple Silicon 原生 arm64 架构，
> 构建脚本会自动在 CI runner 上安装原生 arm64 的 mpv。

---

## macOS 开发指南

### 1. 下载 mpv

```shell
brew install mpv
mkdir -p bin/MacOS && cp "$(brew --prefix mpv)/bin/mpv" bin/MacOS/mpv
```

### 2. 调试

```shell
pip install -r requirements/darwin.txt
python Macast.py
```

### 3. 打包

```shell
pip install py2app
pip install setuptools==44.0.0 # 如果 Macast.app 无法运行可尝试此版本
python setup_py2app.py py2app
cp -R bin dist/Macast.app/Contents/Resources/
open dist
```

---

## Windows 开发指南

### 1. 下载 mpv

```powershell
$client = new-object System.Net.WebClient
$client.DownloadFile('https://nchc.dl.sourceforge.net/project/mpv-player-windows/stable/mpv-0.33.0-x86_64.7z','mpv.7z')
7z x -obin mpv.7z *.exe
```

### 2. 调试

```powershell
pip install -r requirements/common.txt
python Macast.py
```

### 3. 打包

```powershell
pip install pyinstaller
pyinstaller --noconfirm -F -w --additional-hooks-dir=. --add-data=".version;." --add-data="macast/xml/*;macast/xml"  --add-data="i18n/zh_CN/LC_MESSAGES/*.mo;i18n/zh_CN/LC_MESSAGES" --add-data="assets/*;assets" --add-binary="bin/mpv.exe;bin" --icon=assets/icon.ico Macast.py
```

---

## Linux 开发指南（以 Ubuntu 为例）

### 1. 安装 mpv

```shell
sudo apt install mpv
```

### 2. 调试

```shell
pip install -r requirements/common.txt
python Macast.py
# 如果有问题，尝试：
export PYSTRAY_BACKEND=gtk && python3 Macast.py
```

提示：确保能使用 **gi** 库：

```
$ python3
Python 3.7.10 (default, Jun  3 2021, 17:51:26)
Type "help", "copyright", "credits" or "license" for more information.
>>> import gi
>>>
```

如果有问题，尝试：**sudo apt-get install python3-gi**

如果使用 conda，请参考：https://stackoverflow.com/a/40303128

GUI 后端支持的详细信息，请参考：https://pystray.readthedocs.io/en/latest/usage.html#selecting-a-backend

### 3. 打包

```shell
# 构建二进制
pip install pyinstaller
pyinstaller --noconfirm -F -w --additional-hooks-dir=. --add-data=".version:." --add-data="macast/xml/*:macast/xml"  --add-data="i18n/zh_CN/LC_MESSAGES/*.mo:i18n/zh_CN/LC_MESSAGES" --add-data="assets/*:assets" Macast.py
# 构建 deb 包
export VERSION=`cat .version`
mkdir -p dist/DEBIAN
mkdir -p dist/usr/bin
mkdir -p dist/usr/share/applications
mkdir -p dist/usr/share/icons
echo -e "Package: Macast\nVersion: ${VERSION}\nArchitecture: amd64\nMaintainer: xfangfang\nDescription: DLNA Media Renderer\nDepends: mpv" > dist/DEBIAN/control
echo -e "[Desktop Entry]\nName=Macast\nComment=DLNA Media Renderer\nExec=/usr/bin/macast\nIcon=/usr/share/icons/Macast.png\nTerminal=false\nType=Application\nCategories=Video" > dist/usr/share/applications/macast.desktop
mv dist/Macast dist/usr/bin/macast
cp assets/icon.png dist/usr/share/icons/Macast.png
dpkg -b dist Macast-v${VERSION}.deb
```

### 4. 使用 Docker 构建（感谢 **cdrx/docker-pyinstaller**）

不确定是否能正常运行，用于为旧版 Linux 添加支持。

```shell
cp requirements/common.txt requirements.txt
docker run \
  --env PYPI_INDEX_URL="https://pypi.tuna.tsinghua.edu.cn/simple" \
  --env PYPI_URL="https://pypi.tuna.tsinghua.edu.cn" \
  --rm -v "$(pwd):/src/" xfangfang/pyinstaller-linux:python3 \
    'pip install --upgrade pip &&\
    pip install --no-use-pep517 --upgrade pyinstaller &&\
    pyinstaller --noconfirm -F -w \
      --additional-hooks-dir=. \
      --add-data=".version:." \
      --add-data="macast/xml/*:macast/xml" \
      --add-data="i18n/zh_CN/LC_MESSAGES/*.mo:i18n/zh_CN/LC_MESSAGES" \
      --add-data="assets/*:assets" \
    Macast.py'
```
