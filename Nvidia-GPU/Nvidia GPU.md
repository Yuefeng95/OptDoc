# 一些有用的信息 

## 官方文档地址
https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html

https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html

## 名词解释

> 物理单元：
> - GPC 图形处理集群，其中拥有多个TPC
> - TPC 纹理处理集群，其中拥有多个SM
> - SM(streaming multiprocessor) 流式多处理器，它包含若干调度器(Warp)、计算核心、寄存器、L1Cache、内存传输等结构
> - Warp 调度器，其拥有软件意义和物理意义，存在于SM中的每个处理块中。每个调度器支持32路(32SP)的SIMT。
> - Core 包含浮点核心、整数核心、Tensor核心等。
>> *Core与Thread(SP)并不是一一对应的*
>> *Warp实现的多Thread并行在物理上并非于一个cycle发出。Warp的32Thread于4个cycle全部发出。每个指令执行需要1个cycle，但具有4个cycle周期*
> - Tex Texture 纹理访问
> - TensorCore 能通过简洁指令执行矩阵运算的核心
> - LD/ST 加载和存储单元
> - SFU (Special Function Unit)特殊函数计算单元，执行插值运算等特殊运算。

# Nvidia GPU 架构一览

-   Tesla （特斯拉） C语言变成+SIMT
-   Fermi （费米）增强双精度浮点计算性能、L1 L2
    Cache可配置、支持ECC内存增加原子操作性能，每个SM有32\*core\
    ![](https://raw.githubusercontent.com/Yuefeng95/Images/main/img/39133676.png?token=AINSJPGQQFW5NUM2JAZLFUTB6VNA4)
-   Kepler
    （开普勒）合并核心，统一GPU时钟，提升效能和可编程性，每个SMX中有192\*core\
    ![](https://raw.githubusercontent.com/Yuefeng95/Images/main/img/202201292316615.png?token=AINSJPA5PMCFRXPXQUAOTMLB6VNIU)
-   Maxwell （麦克斯韦）平衡Fermi的高性能和Kepler的高效率\
    ![](https://raw.githubusercontent.com/Yuefeng95/Images/main/img/202201292317262.png?token=AINSJPGZDAYMF7I3T2TQQBLB6VNLU)
-   Pascal （帕斯卡）升级工艺、增加双精度运算单元\
    GPU   
    ![](https://raw.githubusercontent.com/Yuefeng95/Images/main/img/202201292319664.png?token=AINSJPEPAFHVJVOUTL67M7TB6VNVC)
    SM   
    ![](https://raw.githubusercontent.com/Yuefeng95/Images/main/img/202201292320182.png?token=AINSJPGIOKACWUTTJ6E5ZBTB6VNXS)
-   Volta
    （福塔）每个SM中有4\*ProcessBlock，特别突出的是每个ProcessBlock有独立的INT、FP32计算单元、TensorCore\
    GPU   
    ![](https://raw.githubusercontent.com/Yuefeng95/Images/main/img/202201292320177.png?token=AINSJPH4OV2LPMDGVHXY6LLB6VNYU)
    SM   
    ![](https://raw.githubusercontent.com/Yuefeng95/Images/main/img/202201292320701.png?token=AINSJPDW6JCI2F7CE66657DB6VNZO)
-   Turing （图灵）去掉FP64支持，为每个SM增加一个RTCore光线追踪\
    GPU 
    ![](https://raw.githubusercontent.com/Yuefeng95/Images/main/img/202201292322374.png?token=AINSJPGHXB4EOJ3MAJZGHC3B6VN5Q)  
    SM 
    ![](https://raw.githubusercontent.com/Yuefeng95/Images/main/img/202201292322641.png?token=AINSJPAPKBVIEXPR2GFNDKLB6VN6I)
-   Ampere （安培）

## Volta架构细节

每个SM中有4个ProcessBlock，其中每个ProcessBlock的组成如下：

- 1个 Warp Scheduler，1个 Dispatch Unit =》 一个时钟，一个调度器
- 8 个 FP64 Core
- 16 个 INT32 Core
- 16 个 FP32 Core
- 2 个 Tensor Core =》矩阵浮点运算，提升TFLOPS
- 8个 LD/ST Unit =》装载or存储单元，同一时钟周期下只能执行装载or存储
- 4个 SFU =》特殊函数执行器，如执行expf

## Turing架构细节

1. 处理器（SM）体系结构

如前言所讲。Turing架构每个SM的每个Block都支持独立的执行INT和FP命令，能更高效的处理混合精度运算、提高core利用率。

以2080ti为例。

|SM||||||||||
|---| - | - | - | - | - | - | - | - | - |
| Block / SM | L1(SharedMemory) 96KB / SM |	RT Core / SM |	Texture / SM | Warp / Block | L0 64KB / Block |	INT32 Core / Block |	FP32 Core / Block |	Tensor Core / Block |	LD/ST  / Block |
|4|	1|	1|	4|	1|	1	|16	|16	|2	|4 |

2. 可配置的L1缓存、SharedMemory

![](https://raw.githubusercontent.com/Yuefeng95/Images/main/img/202201292324127.png?token=AINSJPFJPTS4YRQQDQ4SWYDB6VOGI)

Turing将L1Cache、texture cache（纹理缓存）与SharedMemory的做成统一片
内存。他们的大小由运行时配置。这个配置如图所示。如此可增加访问效率，增加大内存留存率，提高内存命中率。

> 纹理缓存：[外部链接](https://liangz0707.github.io/whoimi/blogs/GPUAartch/%E7%BA%B9%E7%90%86%E7%BC%93%E5%AD%98.html) 与光栅大小匹配的内存块
>
> 内存测试信息：[外部链接](https://zhuanlan.zhihu.com/p/99347372)