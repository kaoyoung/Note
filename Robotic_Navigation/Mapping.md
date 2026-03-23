參考自[ROS2-Mapping & Navigation Courses](https://docs.hiwonder.com/projects/JetAutoPi/en/latest/docs/5.mapping_and_navigation.html)
URDF Model
---
The Unified Robot Description Format (URDF) is an [[XML]] file format widely used in ROS (Robot Operating System) to comprehensively describe all components of a robot.

這邊的Model想要定義機器人的動作，而機器人的動作可由link、joint來表達，所以xml主要紀錄機器人的
+ link
+ joint

### XML format
#### Basic Syntax
+ **Elements** : 
```
<element>

</element>
```
+ **Properties** : define characteristics and parameters
```
<element

property_1="property value1"

property_2="property value2">

</element>
+ Elements: 
```
- **Comments :** have no impact on the definition of other properties and elements.
```
<!– comment content –>
```
#### Link
The Link element describes the visual and physical properties of the robot’s rigid component.

---







### Principles of SLAM Mapping
### **Introduction to SLAM**
SLAM stands for Simultaneous **Localization** and **Mapping**.
	1. **Localization** : determining the pose of a robot in a coordinate system.
	2. **Mapping** : involves creating a map of the surrounding environment perceived by the robot.

---
### **SLAM Mapping Principle**
#### Step 1: (Preprocessing)
Optimizing the raw data from the radar point cloud, filtering out problematic data or performing filtering.
![[Pasted image 20251111222449.png]]
> Point cloud : In simple terms, the surrounding environment information obtained by lidar is called a point cloud.
#### Step 2: (Matching)
Matching the point cloud data of the current local environment with the established map to find the corresponding position.
#### Step 4: (Map Fusion)
Integrating new round data from the lidar into the original map, ultimately completing the map update.
#### Notes
1. Begin the mapping process by positioning the robot in front of a straight wall or within an enclosed box.(給參考物)
2.  Initiate a 360-degree scan of the environment using the Lidar to ensure a comprehensive survey of the surroundings.(給大綱)
3. For larger areas, it’s recommended to complete a full mapping loop before focusing on scanning smaller environmental details. (給大綱)
---
### slam_toolbox Mapping Algorithm

