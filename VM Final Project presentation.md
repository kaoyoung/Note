# 題目: Coordinated Prefetch I/O Scheduling for Concurrent Firecracker MicroVM Snapshot Restoration

# 問題本身
在 host 端用 userspace daemon 協調 N 個並發 Firecracker microVM 還原時的 prefetch I/O,比較三種 scheduling policy(none / FIFO / RR)對 P50/P95/P99 first-response latency 的影響。
# 基本概念




清除快取
echo 3 > /proc/sys/vm/drop_caches

# 檔案
## io_coordinator.py




## worker.py



## run_experiment.sh


## record_ws.py





## bench_sparse.sh



## summary_sparse.py


## setup_network.sh


# 限速模擬




# 結果分析







### Question 1: prefetch 跟 page fault  區分
### Linux kernel 對於 I/O contention 的優化

cat /sys/block/sda/queue/scheduler
### Question 2: 為何不用 FaaSnap 的 repo
我們模擬了 FaaSnap 的 concurrent paging 機制