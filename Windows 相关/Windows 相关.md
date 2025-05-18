
# doc

转UTF-8:
```sh
CHCP 65001
```

递归查找当前文件夹下所有文件中的`MyStr`字段(中文报错的话就转UTF-8):
```sh
findstr /s /i /n "MyStr" *.*
```

`cd`的`/d`选项表示路径可以跨盘(否则无法跨盘cd).

`%cd%`获取当前工作目录.

查看文件md5:
```sh
certutil -hashfile file_name MD5
```




# bat编写

`%~dp0`表示bat所在目录的路径, 在bat文件开头写这个表示此bat与调用它的位置无关.

简单的调用python：
```bash
@echo off
cd /d %~dp0

set PYTHON_EXEC=.\python-3.8.10-embed-amd64\python.exe
set SCRIPT_PATH=.\173Pyqt\main.py

%PYTHON_EXEC% %SCRIPT_PATH%

pause
```

把所有`.m`后缀的文件内容全部展开放到一个txt里面，文件之间使用空行分隔，但是本来就有空行的就不会另加空行了。
```shell
@echo off
setlocal enabledelayedexpansion
set "first=1"

for /f "delims=" %%f in ('dir /b *.m') do (
    if !first! equ 1 (
        type "%%f" > combined.txt
        set "first=0"
    ) else (
        echo. >> combined.txt
        echo. >> combined.txt
        type "%%f" >> combined.txt
    )
)
endlocal
```



