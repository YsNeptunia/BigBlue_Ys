# BigBlue_Ys
自动捡乒乓球小车，我的电赛校赛选拔的题目。项目时间2024.6。
三人小队，我担任队长，负责方案整理、车体结构设计、32端下位机控制代码的编写、协调各成员分工等工作。


## 原题目
### 一、任务
设计并制作一个自动捡球小车，根据要求完成要求的任务。
### 二、要求
#### 1、基本要求
（1）小车从场地中央位置出发，能自主行驶。
（2）场地内随机放置10个乒乓球，小车能完成乒乓球的拾取，每拾取一个乒乓球发出声光指示信息。
（3）90秒内能完成10个乒乓球的拾取，并且全部存放在小车上。
（4）小车有相关信息的显示。
#### 2、发挥部分
 (1) 场地内随机放置10个乒乓球，5个白色5个橙色，小车从场地中央位置出发，拾取5个黄色乒乓球。
 (2) 拾取5个黄色乒乓球后，再完成5个白色乒乓球的拾取。
 (3) 以上过程不超过90秒，超时后所有动作不得分。
 (4) 场地内随机放置10个乒乓球，5个白色5个橙色，小车从场地中央位置出发，交替拾取不同颜色乒乓球，直到乒乓球全部拾取完，该过程不超过90秒，超时后所有动作不得分。
### 三、说明
#### 1、场地地面为1.5*1.5m的普通瓷砖地面，四周有高于2cm的围挡防止乒乓球出场地。
#### 2、小车允许用玩具车改装，但不能由人工遥控，和外接拉线供电和控制，其外围尺寸（含车体上附加装置）的限制为：长度≤20cm，宽度≤15cm。


## 实现过程
  首先，本题的最大难点不是控制，而是：如何在白色瓷砖地板上识别出白色的乒乓球。

  老师在题目讲解时认为我们用OpenMV调一下阈值就可以，可问题是，场地瓷砖是反光的，当灯光或者阳光照到地板上被反光时，OpenMV摄像头根本分辨不出乒乓球轮廓和地板（事实上，这也是很多其他人的难点，很多人也想当然在最后一个周用OpenMV识别结果失败）。用传统的图像处理思路是行不通了，在多次尝试后我们确定并验证了方案：
  **使用部署在嵌入式上位机中的神经网络视觉识别出白背景中的白球，上位机通过串口传回球中心在摄像头画面中的坐标，STM32下位机解算坐标，控制智能小车实现抓取乒乓球。**
  ![image](https://github.com/user-attachments/assets/4efc2799-f705-495f-805b-f9b7af9495db)


  对于视觉部分，这部分的代码由有经验的队友1负责（队友2比较新手，我就让他负责上下位机通信协议部分了）。最初我们使用的嵌入式上位机是树莓派4b，但是其算力实在有限，精简后的YoloV5Lite算法只能跑到不到10帧，这对于有时间要求的题目而言显然是远远不足的。
  后来我们尝试了英伟达的JetsonNano开发板，由于可以调用GPU，同样的算法在这个平台上可以达到稳定30帧，最终决定了该方案。
  ![1738659613912](https://github.com/user-attachments/assets/6e3c4acb-9b86-41dd-ad57-7e16bf4897c2)

  
  对于结构部分，我自行测绘建模，3D打印了车体的底盘和机械臂等结构。使用减速电机控制小车运动，连接上位机的摄像头对场地内的球进行寻找，当摄像头识别到球在指定位置后用舵机控制机械臂下压，用矿泉水瓶和橡皮筋制作类似网兜的结构将地上的球框柱，运送到高处使球自己滚落到车体上的收集框中。同时由于有球数量计数要求，我在机械臂上安装了一个光电门进行计数，相比只计数下压次数而言，这种方案显然更合理。（只计算下压次数有可能错漏导致无法拾起所有球）
  ![1738659613925](https://github.com/user-attachments/assets/232799d2-5623-4b2c-9589-3232d9212ce7)
  ![1738659757531](https://github.com/user-attachments/assets/c197b96a-ed0a-43d4-8b90-2096838dd86f)



  对于控制部分，依然使用了我较为熟悉的PID算法，在接收到上位机传来的xy坐标后32下位机控制两轮电机，到达位置后实行抓取操作。题目有三种抓球顺序所以我也设计了三种抓球模式。曾经担心卡轮子等潜在问题，在小车前安装了一个测距传感器，后来发现其实不需要。
  

https://github.com/user-attachments/assets/106b4148-775f-4484-b51f-987c8159d40d



  从立项到最终完成时间大约一个月。最终可以实现稳定在90秒内抓取所有球。

  ## 说明
  ### Car压缩包内为32下位机的代码工程文件，使用STM32CubeMX的HAL库开发，环境为Keil5和VS Code等。
  ### onnxruntime1压缩包内为上位机的yolov5代码和串口通信代码。环境为Jetson Nano中的Linux系统，安装Pytorch等。
  ### STL文件夹为小车的建模工程文件。

  

