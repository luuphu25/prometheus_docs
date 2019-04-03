Metrics
===========================================
**1. CPU**

- **CPU info** ( chi tiết về cpu time )-> Tính time các mode cpu time 

    ``avg by (mode)(irate(node_cpu_seconds_total{mode='iowait',instance=~"$node:$port",job=~"$job"}[5m])) * 100``

*Note: chi tiết các time trên cpu giúp ta hiểu rõ hơn vấn đề xuất phát từ CPU hay phần còn lại (disk). Xem thêm ở `Monitor method <Monitor_method.html>`_

- **CPU utilization**

    ``100 - (avg by(env, job) (irate(node_cpu_seconds_total{mode="idle"}[5m]))*100)``

*Note: Trong trường hợp này ta coi các mode còn lại của CPU đều là thể hiện 'busy' của CPU và tính theo %. các tính chi tiết: `CPU usage <https://www.robustperception.io/understanding-machine-cpu-usage>`_

- **CPU saturation** 

Riêng với CPU saturation như ở `Monitor method <Monitor_method.html>`_ ta định nghĩa bằng **Load_Avage** và **running process** thì đều là metrics được xuất ra sẵn bởi node_exporter (prometheus). 

**2. Memory**

* **Memory info**
Phần này hiển thị tình trạng của ram: cached, free, buffer được node_exporter xuất ra metric sẵn. 

- **Memory utilization**
Thông số này được tính bằng cách lấy: 

*node_memory_MemTotal_bytes - node_memory_MemFree_bytes - node_memory_Cached_bytes - node_memory_Buffers_bytes*
   
    ``100 - ((node_memory_MemFree_bytes + node_memory_Buffers_bytes + node_memory_Cached_bytes) /node_memory_MemTotal_bytes) * 100``

- **Memory saturation**
Thống số này sẽ được đánh giá qua metrics swap và pgscan/s. Đối với pgscan/s thì thông số này không được bật mặc định trên node_exporter ta phải mở ở collector.vmstat. 

**3. Disk**

- **Disk utilization**
Thông số này sẽ được tính bằng cách tính % time/s mà disk dùng cho i/o. Ex: trong 1s disk thực hiện i/o 0.25s  => 25% utilization `detail <https://unix.stackexchange.com/questions/318274/how-to-calculate-disk-io-load-percentage>`_

    ``irate(node_disk_io_time_seconds_total{device!~"^(md\\\\d+$|dm-)"}[5m])``

- **Disk saturation**
Thông số này được tính bằng số tiến trình đang đợi I/O. (sar -b) 
    ``rate(node_disk_io_time_weighted_seconds_total[5m])`` 

*Note: `chi tiết <https://www.robustperception.io/mapping-iostat-to-the-node-exporters-node_disk_-metrics>`_






