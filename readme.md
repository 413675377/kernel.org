# Linux kernel
Date：2022-05-03 start

本仓库主要内容是简化Linux Kernel[官方文档](https://www.kernel.org/doc/html/latest/)，降低Linux Kernel学习难度，仅此而已！

$ tree kernel.org/  
kernel.org  
├── [API.md](API.md) 
├── [ALSA.md](ALSA.md)   
├── [DeviceModel.md ](DeviceModel.md)  
├── [Devicetree.md](Devicetree.md)  
├── [Kbuild.md](Kbuild.md)   

==TODO==  

- [ ] Coachz audio 实现逻辑 [on-going]
- [ ] Device class
- [ ] Linux Input子系统
- [ ] Zephyr RTOS代码分析
- [ ] 会使用Linux Driver API

#### Code阅读方法
1. 细化问题
1. VSCode追踪
2. 查API
3. 画出整个流程
3. 结合案例

#### Target
- 根据完整的笔记本电路原理图，理清代码逻辑

- 能够bringup，develop，maintain a board

[Plan](ChromebookTextbook.html)