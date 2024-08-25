
ECS(Entity-Component-System)是一种编程范式，通常用在游戏开发里面。Entity就是抽象实体，而Component是类似于Trait的“功能”概念，System则进行功能执行。

比如Unity放一个物体就是Entity，物体的位置、大小、运动、物理、光影都是Component；而物理引擎就是一个需要位置、大小、运动、物理的System，这个System会找到所有拥有这些Component的Entity，进行相关操作。

[bevy_ecs](https://crates.io/crates/bevy_ecs)是Rust的ecs实现，使用起来比较方便。Entity被表示为类似于id的统一类型，Component是纯结构体（存储相关内容信息），而System则是纯函数（通过操作一个Entity拥有的Component里面包含的数据来操作此Entity）。它们都被放进World大容器里面进行管理。线程安全。还有Resource来表示互斥资源，Schedules来管理所有System的执行。
