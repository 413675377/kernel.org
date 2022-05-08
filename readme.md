# Linux kernel
Date：2022-05-03 start

本仓库主要内容是简化Linux Kernel[官方文档](https://www.kernel.org/doc/html/latest/)，降低Linux Kernel学习难度，仅此而已！

### Recommand
API search:  [https://www.kernel.org/doc/html/latest/](https://www.kernel.org/doc/html/latest/)   
Code browser: [https://elixir.bootlin.com/linux/v5.18-rc5/source](https://elixir.bootlin.com/linux/v5.18-rc5/source)    
CS Lesson：[https://linux-kernel-labs.github.io/refs/heads/master/so2/index.html](https://linux-kernel-labs.github.io/refs/heads/master/so2/index.html)    

$ tree kernel.org/     
├── [Kbuild.md](Kbuild.md)   
├── [Devicetree.md](Devicetree.md)   
├── [DriverModel.md ](DriverModel.md)   
├── [DriverPorting.md](DriverPorting.md)   
├── [ALSA.md](ALSA.md)   
├── [ALSA: CoachZ audio driver](Driver_Audio_CoachZ.md)  [on-going] 

==TODO==  
- [ ] [CoachZ Sound Driver](CoachZ Sound Driver.html) [on-going]
- [ ] Device class
- [ ] Linux Input子系统
- [ ] Zephyr RTOS代码分析
- [ ] 会使用Linux Driver API
- [ ] [AI Plan](ChromebookTextbook.html)

### Code阅读方法
1. 解决的问题
2. Code追踪，（查API）
3. 画流程
4. 实验

### Target
- 能根据原理图，理清代码逻辑
- 能独立bringup，develop，maintain a board
- 能移植kernel

