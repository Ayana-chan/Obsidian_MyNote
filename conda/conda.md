

# 概念

conda允许用户创建独立的环境，然后在环境里面安装全局软件、设置全局变量，这样只要切到这个环境就能使用这些东西了。

# 安装相关

anaconda包含了较全面的库，而miniconda则只有基本的库。

安装后，默认会把环境和包下载到C盘里面，这时候就需要修改用户文件下的`.condarc`文件来修改配置（默认不存在，要进行配置修改才会被创建）（默认是空文件，在里面写上要设置的属性后，会自动修改conda属性）。

```
envs_dirs:
  - D:\miniconda3\envs
pkgs_dirs:
  - D:\miniconda3\pkgs
```

修改env和pkg位置后，可能出现无法读取的问题，比如`D:\miniconda3`可能是管理员文件，对普通用户只读，那么就需要手动给改文件夹权限。

# 基本使用

创建环境：

```shell
conda create -n myenv python=3.8 -y
```

- -n myenv 指定了新环境的名称为 myenv。
- python=3.8 指定了要安装的 Python 版本。
- -y 表示自动确认所有安装操作。

激活环境：

```shell
conda activate myenv
```

退出环境：

```shell
conda deactivate
```

激活环境后，由于环境里面就有Python，因此此时的pip操作都是在环境里面进行操作。










