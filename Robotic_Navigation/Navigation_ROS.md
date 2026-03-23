## 參考自[[# 带你理清：ROS机器人导航功能实现、解析、以及参数说明]](https://blog.csdn.net/qq_42406643/article/details/118754093)
# 機器人導航先決條件
**第一件事是确保机器人本身准备好了，具有导航能力**。这包括三个组件检查：**距离传感器**、**里程计**、**定位**。
1. **距离传感器** : 可以尝试在rviz中查看传感器信息，是否可以通过话题Topic获取传感器信息，并以预期的速度获取。
2. **里程计** : 往往机器人无法正确定位，问题根源不在Amcl算法调参，而是里程计不可靠。
	1. **第一个测试检查角速度是否合理**
	2. **第二个测试检查线速度是否合理**
3. **定位**

---
# 導航
在ROS中，導航三套件
1. move_base : 路径规划
2. amcl：根据已经有的地图进行定位。
3. gmapping：根据激光数据（或者深度数据模拟的激光数据）建立地图。
## move_base
### **架構圖 :**
 ![[Pasted image 20251117230945.png]]
### **輸入到長方形(move_base)的資料 :**
- **/tf**：提要提供的tf包括map_frame、odom_frame、base_frame以及机器人各关节之间的完成的一棵tf树。tf树用于跟踪和管理多个坐标系之间的关系。
- **/odom**：里程计信息。odem更多資訊可看[[ ROS导航中关于odom frame的理解]]
- **/scan 或 /pointcloud**：传感器的输入信息，最常用的是激光雷达(sensor_msgs/LaserScan类型)，也有用点云数据(sensor_msgs/PointCloud)的。
- **/map**：地图，可以由SLAM程序来提供，也可以由map_server来指定已知地图（或自定义的室内地图）
- **move_base_simple/goal**：目标点位置（也就是导航的目标点）
前四个Topic是必须持续提供给导航系统的(robot本身的資料、走過那些地方、現在的環境)，最後一個是可随时发布的[[topic]]。
### **長方形(move_base)的插件 :**
- **base_local_planner**插件
- **base_global_planner**插件
- **recovery_behavior**插件
	- clear_costmap_recovery: 实现了清除代价地图的恢复行为
	- rotate_recovery: 实现了旋转的恢复行为
	- move_slow_and_clear: 实现了缓慢移动的恢复行为

### **長方形(move_base)的資料 :**
在ROS中使用costmap_2d这个软件包来实现的，该软件包在原始地图上生成了两张新的地图。
- global_costmap : 全局路径规划准备的。
- local_costmap : 局部路径规划准备的。
這兩地圖都可配置以下圖層 : 
- Static Map Layer：静态地图层，基本上不变的地图层，一般是指SLAM建立完成的静态地图。
- Obstacle Map Layer：障碍地图层，由激光雷达等障碍扫描传感器提供实时数据来构建。
- Inflation Layer：膨胀层，在以上两层地图上进行膨胀（向外扩张），以避免机器人的撞上障碍物。
- 或者有voxel_layer体素层，利用三维空间中的体素。
- Other Layers：你还可以通过插件的形式自己实现costmap，目前已有Social Costmap Layer、Range Sensor Layer等开源插件。

![[Pasted image 20251118005432.png]]

![[Pasted image 20251118005456.png]]

# 調參方法

參考這篇[代价地图、局部规划器调参说明](https://blog.csdn.net/qq_42406643/article/details/119007076)



