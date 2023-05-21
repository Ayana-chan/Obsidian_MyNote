# 安装配置

记得使用posix的mingw。换mingw后重启电脑，才能让cmake识别到。

可能有包（ffmpeg）无法下载，则去`...\opencv\mingw_build\CMakeDownloadLog.txt`里面找到下载链接，手动下载三个文件并放到报错目标位置。

[Windows上使用CLion配置OpenCV环境\_迷途小书童的Note的博客-CSDN博客](https://blog.csdn.net/djstavaV/article/details/125383444)

安装成功后修改cmakelist：

```cmake
#OpenCV  
set(OpenCV_DIR "D:\\DeveloperTools\\opencv-4.7.0-windows\\opencv\\mingw_build\\install")  
find_package(OpenCV REQUIRED)  
include_directories(${OpenCV_INCLUDE_DIRS})  

add_executable(test OpenCVTest.cpp)
#连接到项目  
target_link_libraries(test ${OpenCV_LIBS})
```

# 库与语法

## waitKey

阻塞直到收到键盘点击事件。

参数可以指定阻塞最长时间，超出则关闭窗口。


















