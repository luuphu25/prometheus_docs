Method monitoring
===========================================
Nhiều service có thể bị ảnh hưởng hiệu suất do nền tảng OS mà nó chạy phía trên (linux). Đặc biệt việc monitor tài nguyên hệ thống của Linux càng cần thiết nếu ứng dụng nhạy cảm với tài nguyên hệ thống ( như I/O ảnh hưởng tới DB service)

Đối với Linux, ta cần quan tâm nhất là CPU, Disk, một phần memory (cho saturation) và network. Thực chất Network hiện nay khá khó overload (tầng OS) nên phần lớn vẫn dựa vào monitor ở service (lantency, rate) … 

Tuy nhiên cũng có thể monitor rate của IN/OUT network để cung cấp khả năng phát hiện các hoạt động bất thường đối với hệ thống với việc phổi hợp cùng với các metrics khác.
 

**1. CPU**

Khía cạnh thực tế cần quan tâm là **Saturation** và **Utilization**.  
**#Saturation:**  Để dễ dàng người ta thường đánh giá thông số **Load_Average** (số lượng tb tiến trình đang sử dụng CPU hoặc đợi tài nguyên hệ thống < cả CPU và DISK>). Do trong Linux **Load_Avg** là gộp chung cả CPU và Disk I/O nên thông số này chưa nói lên được vấn đề ở Disk hay CPU.  Nếu service của hệ thông không hoạt động I/O nhiều thì chỉ số Load Avg có thể dùng đánh giá CPU saturation. Nhìn chung metric này vẫn cho ta nhận định được hệ thống có vấn đề không để từ đó xem xét các metrics cụ thể hơn của CPU và Disk khi monitor 

Ngoài ra, đối với các server hypervisor ta có thể quan tâm thêm chỉ số **%Steal time**. **Steal time** nếu cao cũng chỉ ra CPU đang bị đầy.  Nếu `%steam > 20` % thì cần nên xem xét hoạt động của server [1]_

Mặt khác Percona (1 hãng phần mềm) cũng đưa ra cách thức riêng để đánh giá các chỉ số liên quan tới **Saturation** [2]_  

`Sample <https://pmmdemo.percona.com/graph/d/qyzrQGHmk/system-overview?refresh=1m&panelId=33&fullscreen&orgId=1&var-interval=$__auto_interval&var-host=ps56&from=now-12h&to=now>`_ 

`Detail về CPU <https://www.slashroot.in/linux-cpu-performance-monitoring-tutorial>`_

*Lưu ý rằng, metrics CPU saturation sẽ không có nhiều ý nghĩ nếu hệ thống chạy service single-thread: Nodejs server, Nginx … khi đó cần đánh giá* **CPU Utilization**
 

**#Utilization**

Ta tính tổng % 

``sum CPU %: User + Nice + System + (probably) Steal``

Ngoài tính %CPU sử dụng cần lưu một số thông số cụ thể của CPU time:

* I/O wait nếu chỉ số I/O wait cao là dấu hiệu hệ thống disk có bottleneck và làm chậm hệ thống  
* Softirq cao có thể là dấu hiệu có vấn đề ở thiết bị mạng  
* Nếu chỉ số idle = 0 trong 1 khoảng thời gian => khả năng xử lý của CPU tới giới hạn  

https://singapore.my-netdata.io/default.html#menu_system_submenu_cpu;theme=slate;help=true 

**2. Memory**  

Đối vơi memory ta cũng chỉ quan tâm tới **Saturation** và **Utilization**

**#Saturation**
Bất ki việc swap nào đều thể hiện việc thiếu RAM.  Tuy nhiên trên các OS linux khi chỉ số swappiness được set !=1 , thì có thể có tình trạng swap trước khi Ram được sử dụng hết điều này là do kernel ko giảm các cache khi thực hiện swap nên bộ nhớ của bạn có thể cạn kiệt trước khi toàn bộ RAM đều đầy bởi hoạt động của các tiến trình.  


**#Utilization**
Tính toán % lượng Ram đã sử dụng
```used = total - (free + buffer + cache)```

  
**#page faults and swapping**
Để đánh giá chính xác hơn tình trạng hệ thống có xử dụng RAM, ta cần quan tâm nhất tới mức độ (tần suất) swap_in/out. Khi rate cao thì nghĩa là ram vật lý không đủ và hệ thông phải sử dụng disk thường xuyên - với hiệu suất thấp hơn RAM. `Page_fault <Page_fault.html>`_

Còn nếu rate của page_out thấp thì ko ảnh hưởng tới hiệu suất của hệ thống 

**Metrics quan tấm**

- Swap Activity (swap-ins and swap outs)
- Amount of swap space used

Cũng nên theo dõi **OOM error** (OUT of Memory) : khi RAM bị đầy nghiêm trọng, OS sẽ tự động kill process user chiếm nhiều RAM và đó có thể là service MySQL, server .. 

 

**3. Disk:** 

Cách mapping các metrics I/O càn thiết trong Node_exporter (prometheus) [3]_

**Request Rate :** chỉ số IOPS của disk system ( r/s và w/s) 

**Error Rate:** Không có lỗi cụ thể trên hệ thống đĩa ngoại trừ độ trễ quá cao. Hầu hết lỗi có thẻ xảy ra liên quan tới kiểu file hệ thống: ext4 hay nfs đều khó phát hiện và thường tìm thông qua kernel log.  

**Latency:** Cần theo dõi cả read và write time.và chỉ số trễ thường tăng cao nếu disk bị bão hòa.  
**Saturation:** Tiến hành monitor bằng I/O queue trên từng thiết bị. Có thể lấy bằng 
``iostat -aqu-sz``. I/O queue cho biết số yêu cầu I/O còn tồn đọng. thường chỉ số này ở múc 2-3 là tối ưu. Càng nhiều hơn càng có nguy cơ bottle neck

**Utilization:** nghĩa là đĩa đang thực hiện công việc nào đó. Thường thì 100% chỉ ra tình trạng đĩa có thể bị tắc nghẽn. Thông thường tính bằng sử dụng %util (Percentage of CPU time during which I/O requests were issued)
 

Disk system là thành phần trọng yếu trong việc monitor server do nó có thể ảnh hưởng lớn tới khả năng hoạt động của service. Đồng thời cũng là nơi thường gặp vấn đề nhất. Tuy nhiên cũng có những vấn đề bị che dấu gián tiếp bởi các thành phần khác. ( IBM red book)  như thiết RAM, CPU sử dụng cao bởi các tiến trình đang đợi I/O  

Disk chậm có thể là kết quả của:  

- Memory buffer bị đầy bởi dữ liệu cần ghi => Delay request vì không còn free mem để thực hiện write request hoặc response phải đợi dữ liệu được đọc từ disk 

- Thiếu Ram, không đủ buffer cho network request -> lỗi ghi đồng bộ  

Disk utilization, controller utilization 

- Disk chưa hoàn thành ( châm) -> được thể hiện bở response time cao hay network utilization thấp  

- Disk I/O thời gian dài -> hàng đợi full -> CPU high idle + low utilization do đợi requests tiếp theo.  
 


**4. Network:** 

Rate: READ/WRITE bandwidth / s  

Utilization:  Read/write bandwith / total bandwitdth  


.. [1] https://scoutapp.com/blog/understanding-cpu-steal-time-when-should-you-be-worried 
.. [2] https://www.percona.com/blog/2017/08/04/saturation-metrics-in-pmm-120/  
.. [3] https://www.robustperception.io/mapping-iostat-to-the-node-exporters-node_disk_-metrics
 

 
