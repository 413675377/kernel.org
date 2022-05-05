# Linux kernel
Date：2022-05-03 start

本仓库主要内容是简化Linux Kernel[官方文档](https://www.kernel.org/doc/html/latest/)，降低Linux Kernel学习难度，仅此而已！

$ tree kernel.org/  
kernel.org  
├── [ALSA.md](ALSA.md)  
├── [DeviceModel.md ](DeviceModel.md)  
├── [Devicetree.md](Devicetree.md)  
├── [Kbuild.md](Kbuild.md)  
├── api  
│   ├── [API_ALSA.md ](api/API_ALSA.md)  
│   ├── [API_Devicetree.md ](api/API_Devicetree.md)  
│   ├── [API_Driver_basic.md ](api/API_Driver_basic.md)   
│   └── [LIB_Driver.md ](api/LIB_Driver.md)  

#TODO  

- [ ] Linux Input子系统
- [ ] Zephyr RTOS代码分析
- [ ] 结合案例分析
- [ ] 会使用Linux Driver API

#### Target
- 根据完整的笔记本电路原理图，理清代码逻辑
- 能够bringup，develop，maintain a board