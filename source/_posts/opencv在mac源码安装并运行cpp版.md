---
title: "opencv在mac源码安装并运行cpp版"
date: 2019-03-14 20:00:00
tags:
- opencv
- cpp
---



### mac 安装 cmake



+ 下载安装 CMake。

  https://cmake.org/download/   Mac OS X 10.7 or later

+ 安装完成之后，使用以下指令创建/usr/local/bin下 CMake 的软链接。

```shell
sudo "/Applications/CMake.app/Contents/bin/cmake-gui" --install
```



### 源码安装 opencv

目前opencv已经出到4.0+版本了, 网上大部分教程都是2.0,3.0版本的.

不过我们选择最新的版本, 直接从github上拉取

```shell
git clone https://github.com/opencv/opencv.git
mkdir build
cd build
cmake ..
make 
sudo make install
```

<!-- more -->

### xcode 配置opencv

如果不想用xcode来开发, 编译的时候直接指定下面的选项

+ Header Search Paths

```
/usr/local/include/opencv4  (不一定是这个, 要看你的make install 安装到哪个目录了)
```

+ Library Search Paths

```
/usr/local/lib (不一定是这个, 要看你的make install 安装到哪个目录了)
```

+ Other Linker Flags

```
-lopencv_calib3d -lopencv_core -lopencv_dnn -lopencv_features2d -lopencv_flann -lopencv_gapi -lopencv_highgui -lopencv_imgcodecs -lopencv_imgproc -lopencv_ml -lopencv_objdetect -lopencv_photo -lopencv_stitching -lopencv_video -lopencv_videoio 
```

不一定是这些, 要去`/usr/local/lib` 文件夹(`make install`安装目录)下看 opencv开头的库




### opencv rotate code

```cpp
#include <stdio.h>
#include <opencv2/opencv.hpp>

using namespace cv;

int main(int argc, char* argv[])
{
    if (argc != 3)
    {
        printf("usage: ./images path angle\n");
        return -1;
    }
    
    Mat source = imread(argv[1], 1);
    if (!source.data)
    {
        printf("No image data \n");
        return -1;
    }
    
    double angle = atof(argv[2]);
    Point2f src_center(source.cols/2.0F, source.rows/2.0F);
    Mat rot_mat = getRotationMatrix2D(src_center, angle, 1.0);
    Mat dst;
    warpAffine(source, dst, rot_mat, source.size());
    

    imshow("Source", source);
    imshow("Rotate", dst);
    waitKey(0);
    
    return 0;
}
```

