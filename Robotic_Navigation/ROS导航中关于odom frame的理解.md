## 參考自[ROS导航中关于odom frame的理解](https://blog.csdn.net/allenhsu6/article/details/112427489)
# 目標

想要推算出機器人的位置。odometry是通过航迹推算算出来的机器人位姿。

---
# 解法

使用航迹(靠輪子轉了幾圈、IMU 慣性測量單元)來推算現在的位置，特點是**連續平滑，但有累積誤差**。再由推算的位置`/odom_frame`和`/bae_frame`之间的tf变换
![[Pasted image 20251118001529.png]]

---
# 和map_frame結合

**TF 樹的「單親原則」：「每個 frame 都只能有一個 father**」。
- 機器人想知道自己的位置，理論上可以透過 `/odom` 算出來（`odom` -> `base`）。    
- 機器人也可以透過定位算法（AMCL）在地圖上算出來（`map` -> `base`）。
![[Pasted image 20251118001754.png]]
- **綠圈** : `/map_frame`和`/odom_frame`之间的tf关系由AMCL来维护。只需要AMCL算法用求出来的真实的pose变换
- **紅圈** : 綠圈減去/odom_frame`和`/bae_frame的變換。

---
# 為何不直接由map_frame控制

**機器人控制（Navigation / Control）的實務上，這樣做會導致災難性的後果，主要有兩個致命傷：「瞬移（Teleportation）」和「更新頻率（Frequency）」**。
### 瞬移問題（Teleportation）
AMCL（或 GPS、SLAM）的定位是**「修正型」**的。它不是連續的，而是當它發現「啊！原來我剛剛偏左了一點」，它會**瞬間**把機器人的座標修正過去。
### 更新頻率（Frequency）
**AMCL / SLAM(定位算法)計算量大，很吃 CPU，通常**每秒只能更新 1~5 次 (1-5 Hz)，但**機器人馬達控制**需要非常即時的回饋，通常**每秒需要 20~100 次 (20-100 Hz)**。

### 主要矛盾
精準的東西要比較久，但導航沒法等那麼久只好先走odom將就一下，過段時間再來修正。

### 總結
|**變換層級**|**服務對象**|**特性需求**|**誰在看這個數據？**|
|---|---|---|---|
|**`/odom` -> `/base_link`**|**馬達控制、避障**|**必須平滑、高頻率** (不能跳變)|**Local Planner (局部規劃器)**<br><br>  <br><br>(如 DWB, TEB)|
|**`/map` -> `/odom`**|**全域導航、到達目的地**|**必須準確** (可以跳變、低頻率)|**Global Planner (全域規劃器)**<br><br>  <br><br>(如 NavFn, Global Planner)|