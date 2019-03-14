## 0x00 实现原理

> 参考： https://arxiv.org/abs/1609.01388

一、 帧过滤

1. 低质量帧
    - 黑色
    - 单色
    - 模糊（锐度）
    - 一致性（标准化强度直方图）
2. 淡入淡出（镜头转换） 
3. 溶解
4. 擦除

二、关键帧提取

1. 特征提取（过滤重复）
    - HSV直方图
    - 边缘直方图
2. 关键帧提取
    - 子镜头识别（复活意外被丢弃的高质量帧）
    - 运动能量低（静止）
三、缩略图选择
1. 相关性
    - 聚类
2. 画面吸引力
    - 无监督图像美学（ML）
         - 色彩
         - 纹理（光滑、均匀度）
         - 质量（对比度、曝光...）
         - 构图
   - 监督图像美学（ML）
        - 随机森林回归模型

## 0x01 安装方法（CentOS）

> 目前已在 `in-prod-songbook-silence-1` 配置好环境，可直接使用

### 安装ffmpeg

ffmpeg 安装流程较为复杂，现已整理为shell，直接运行即可。
每种依赖提供了编译安装和yum安装两种方式，目前默认yum，可自行修改。

shell 如下：
```
# 参考： https://trac.ffmpeg.org/wiki/CompilationGuide/Centos

yum -y install autoconf automake bzip2 cmake freetype-devel gcc gcc-c++ git libtool make mercurial pkgconfig zlib-devel bzip2-devel
mkdir ~/ffmpeg_sources


#安装nasm
yum -y install nasm
#cd ~/ffmpeg_sources
#curl -O -L http://www.nasm.us/pub/nasm/releasebuilds/2.13.02/nasm-2.13.02.tar.bz2
#tar xjvf nasm-2.13.02.tar.bz2
#cd nasm-2.13.02
#./autogen.sh
#./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin"
#make
#make install

#安装Yasm 用yum的方式
yum -y install yasm
#cd ~/ffmpeg_sources
#curl -O -L http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz
#tar xzvf yasm-1.3.0.tar.gz
#cd yasm-1.3.0
#./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin"
#make
#make install

#libx 264
cd ~/ffmpeg_sources
git clone --depth 1 http://git.videolan.org/git/x264
cd x264
PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" --enable-static --disable-asm
make
make install

#libx265
cd ~/ffmpeg_sources
hg clone https://bitbucket.org/multicoreware/x265
cd ~/ffmpeg_sources/x265/build/linux
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="$HOME/ffmpeg_build" -DENABLE_SHARED:bool=off ../../source
make
make install



#libfdk_aac
cd ~/ffmpeg_sources
git clone --depth 1 https://github.com/mstorsjo/fdk-aac
cd fdk-aac
autoreconf -fiv
./configure --prefix="$HOME/ffmpeg_build" --disable-shared
make
make install


#libmp3lame
cd ~/ffmpeg_sources
curl -O -L http://downloads.sourceforge.net/project/lame/lame/3.100/lame-3.100.tar.gz
tar xzvf lame-3.100.tar.gz
cd lame-3.100
./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" --disable-shared --enable-nasm
make
make install

#libopus
cd ~/ffmpeg_sources
curl -O -L https://archive.mozilla.org/pub/opus/opus-1.2.1.tar.gz
tar xzvf opus-1.2.1.tar.gz
cd opus-1.2.1
./configure --prefix="$HOME/ffmpeg_build" --disable-shared
make
make install

#libogg
cd ~/ffmpeg_sources
curl -O -L http://downloads.xiph.org/releases/ogg/libogg-1.3.3.tar.gz
tar xzvf libogg-1.3.3.tar.gz
cd libogg-1.3.3
./configure --prefix="$HOME/ffmpeg_build" --disable-shared
make
make install

#libvorbis
cd ~/ffmpeg_sources
curl -O -L http://downloads.xiph.org/releases/vorbis/libvorbis-1.3.5.tar.gz
tar xzvf libvorbis-1.3.5.tar.gz
cd libvorbis-1.3.5
./configure --prefix="$HOME/ffmpeg_build" --with-ogg="$HOME/ffmpeg_build" --disable-shared
make
make install

#libvpx------------
cd ~/ffmpeg_sources
git clone --depth 1 https://chromium.googlesource.com/webm/libvpx.git
cd libvpx
./configure --prefix="$HOME/ffmpeg_build" --disable-examples --disable-unit-tests --enable-vp9-highbitdepth --as=yasm
make
make install

#FFmpeg
cd ~/ffmpeg_sources
curl -O -L https://ffmpeg.org/releases/ffmpeg-4.1.tar.bz2
tar xjvf ffmpeg.tar.bz2
cd ffmpeg
PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure \
--prefix="$HOME/ffmpeg_build" \
--pkg-config-flags="--static" \
--extra-cflags="-I$HOME/ffmpeg_build/include" \
--extra-ldflags="-L$HOME/ffmpeg_build/lib" \
--extra-libs=-lpthread \
--extra-libs=-lm \
--bindir="$HOME/bin" \
--enable-gpl \
--enable-libfdk_aac \
--enable-libfreetype \
--enable-libmp3lame \
--enable-libopus \
--enable-libvorbis \
--enable-libvpx \
--enable-libx264 \
--enable-libx265 \
--enable-nonfree
make
make install
hash -r


# 覆盖之前安装的版本
\cp -f ffmpeg /bin/ffmpeg
\cp -f ffmpeg /usr/bin/ffmpeg
\cp -f ffprobe /bin/ffprobe
\cp -f ffprobe /usr/bin/ffprobe

```

### 安装opencv3

> 参考：https://www.vultr.com/docs/how-to-install-opencv-on-centos-7

安装依赖
```
yum groupinstall "Development Tools" -y
yum install cmake gcc gtk2-devel numpy pkconfig -y

```
下载&解压
```
cd
wget https://github.com/opencv/opencv/archive/3.3.0.zip
unzip 3.3.0.zip
```
编译&安装
```
cd opencv-3.3.0
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=DEBUG -D CMAKE_INSTALL_PREFIX=/usr/local ..
make
make install

export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig/
echo '/usr/local/lib/' >> /etc/ld.so.conf.d/opencv.conf
ldconfig

```

测试效果
```
cd
git clone https://github.com/opencv/opencv_extra.git
export OPENCV_TEST_DATA_PATH=/root/opencv_extra/testdata

cd /root/opencv-3.3.0/build/bin
ls
./opencv_test_photo
```


### hecate 安装 & 使用

> 参考：https://github.com/yahoo/hecate

安装

```
$ git clone https://github.com/yahoo/hecate.git
$ cd hecate
$ vim Makefile.config
 - Set INCLUDE_DIRS and LIBRARY_DIRS to where your 
   opencv library is installed. Usually under /usr/local.
 - If your OpenCV version is 2.4.x, comment out the line 
   OPENCV_VERSION := 3
 - Save and exit
$ make all
$ make distribute
```
使用
```
$ ./distribute/bin/hecate -i examples/video.mp4 --generate_jpg --njpg 1
```

