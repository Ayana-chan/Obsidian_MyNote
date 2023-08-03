

`make`是最常用的**构建系统**之一。

执行`make`时，它会去参考当前目录下名为`Makefile`的文件。所有构建目标、相关依赖和规则都需要在该文件中定义，它看上去是这样的：

```makefile
paper.pdf: paper.tex plot-data.png
	pdflatex paper.tex

plot-%.png: %.dat plot.py
	./plot.py -i $*.dat -o $@
```

冒号左侧的是**构建目标**，冒号右侧的是构建它所需的**依赖**。缩进的部分是从依赖构建目标时需要用到的一段**程序**。

若`make`不带参数，则默认构建第一条。以目标名为参数就可以构建该目标：`make plot-data.png`。

规则中的 `%` 是一种模式，它会匹配其左右两侧相同的字符串。例如，如果目标是 `plot-foo.png`， `make` 会去寻找 `foo.dat` 和 `plot.py` 作为依赖。

make会以目标为单位检查之前的构建是否因其依赖改变而需要被更新。如果一个目标的依赖都没变，则不会对该目标做任何事。


















