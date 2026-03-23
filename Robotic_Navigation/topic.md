## 參考自[# 中国大学MOOC---《机器人操作系统入门》课程讲义](https://sychaichangkun.gitbooks.io/ros-tutorial-icourse163/content/)第三章

# 簡介 
- 实时性、周期性的消息，使用topic来传输是最佳的选择。
- topic是一种点对点的单向通信方式，这里的“点”指的是node，也就是说node之间可以通过topic方式来传递信息。
- topic通信属于一种**异步**的通信方式。
![[Pasted image 20251117235021.png]]
publisher节点和subscriber节点都要到节点管理器进行注册，然後從publisher單向射到subscriber。
- Subscriber接收消息会进行处理，一般这个过程叫做**回调(Callback)**。所谓回调就是提前定义好了一个处理函数（写在代码中），当有消息来就会触发这个处理函数，函数会对消息进行处理。
----
# 通信示例
![[Pasted image 20251117235643.png]]**总结三点**：
1. topic通信方式是异步的，发送时调用publish()方法，发送完成立即返回，不用等待反馈。
2. subscriber通过回调函数的方式来处理消息。
3. topic可以同时有多个subscribers(扩展性好、软件复用率高)，也可以同时有多个publishers。ROS中这样的例子有：/rosout、/tf等等。

---
# 操作命令
![[Pasted image 20251117235821.png]]

# 测试实例
1. 首先打开`ROS-Academy-for-Beginners`的模拟场景，输入`roslaunch robot_sim_demo robot_spawn_launch`,看到我们仿真的模拟环境。该`launch`文件启动了模拟场景、机器人。
2. 查看当前模拟器中存在的topic，输入命令`rostopic list`。可以看到许多topic，它们可以视为模拟器与外界交互的接口。
3. 查询topic`/camera/rgb/image_raw`的相关信息：`rostopic info /camera/rgb/image_raw`。则会显示类型信息type，发布者和订阅者的信息。
4. 上步我们在演示中可以得知，并没有订阅者订阅该主题，我们指定`image_view`来接收这个消息，运行命令`rosrun image_view image_view image:=<image topic> [transport]`。我们可以看到message，即是上一步中的type。
5. 同理我们可以查询摄像头的深度信息depth图像。
6. 在用键盘控制仿真机器人运动的时候，我们可以查看速度指令topic的内容`rostopic echo /cmd_vel` ，可以看到窗口显示的各种坐标参数在不断的变化。