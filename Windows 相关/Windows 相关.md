
# doc

递归查找当前文件夹下所有文件中的`MyStr`字段:
```dos
findstr /s /i /n "MyStr" *.*
```

`cd`的`/d`选项表示路径可以跨盘(否则无法跨盘cd).

`%cd%`获取当前工作目录.
# bat编写

`%~dp0`表示bat所在目录的路径, 在bat文件开头写这个表示此bat与调用它的位置无关.

```bash
@echo off
cd /d %~dp0

set PYTHON_EXEC=.\python-3.8.10-embed-amd64\python.exe
set SCRIPT_PATH=.\173Pyqt\main.py

%PYTHON_EXEC% %SCRIPT_PATH%

pause
```





