.. _Page_Fault-and-Swapping:

Page Fault
===========================================

* **Paging:** Linux cấp phát bộ nhớ cho tiến trình bằng cách chia bộ nhớ vật lý vào các pages (thông thường có độ lớn 4K) -> mapping các page cho bộ nhớ ảo mà tiến trình cần. Công việc này được thực hiện bởi Memory Management Unit (MMU) ở CPU. Những pages trong Linux có các trạng thái khác nhau: free( chưa được sử dụng), đang chứa mã thực thi hay chứa dữ liệu của chương trình...Các giải thuật khác nhau được sử dụng trong việc quản lý các pages đó: cached, free hoặc loaded tùy theo từng trường hợp.

**#Page Fault**
Linux có thể phải thực thi số lượng lớn các chương trình mà mỗi chương trình có thể có mã thực thi lên tới hàng trăm MB, nhưng không phải toàn bộ mã đều cần chạy 1 lúc. Có nhiều đoạn mã chỉ được chương trình khi khởi tạo hoặc khi có điều kiện kích hoạt. Cho nên theo thời gian Linux có thể loại bỏ các page bộ nhớ mà nó nghĩ ko cần thiết hoặc ít sử dụng. Do đó không phải tất cả mã của chương trình đều nằm trong bộ nhớ (vẫn có trên disk) kể cả khi chương trình đó đang được thực thi. 
Một chương trình được thực thi theo các mã hướng dẫn (instruction) bởi CPU. Mỗi instruction được lưu trên bộ nhớ với 1 địa chỉ nhất định. MMU sẽ map các địa chỉ vật lý (RAM) sang các địa chỉ bộ nhớ ảo mà chương trình lưu instruction. Trong khi chương trình thực thi, mà CPU cần địa chỉ nhớ (chưa instruction) mà không có trên bộ nhớ. MMU biết rằng page chứa mã đó không tồn tại (Linux thông báo điều đó) khi đó CPU sẽ raise **page fault**

Page Fault không phải là lỗi, nó nói cho HĐH biết rằng cần truy cập vật lý vào nhiều mã hơn. LInux sẽ cấp nhiều page hơn cho tiến trình, điền mã mà tiến trình cần vào các page mới -> cấu hình lại MMU -> thông báo cho CPU thực thi tiếp. 

**#Minor page fault**:
Là tình trạng page fault mà mã hoặc dữ liệu CPU cần thực thi đã có trên RAM nhưng không được cấp phát cho tiến trình hiện tại. Ví dụ: nếu 1 user đang thực thi trình duyệt web, khi đó memory pages chứa mã thực thi trình duyệt có thể chia sẻ cho người dùng khác (binary chỉ đọc) nên khi có người dùng khác sử dụng trình duyệt, Linux sẽ ko lấy lại mã từ disk mà sẽ mapping memory page chia sẻ được từ tiến trình người dùng 1 cho tiến trình người dùng 2. 
Nói cách khác, trường hợp này HĐH chỉ cần config lại MMU trong việc thực hiện Page Fault.
**#Major page fault**: 
Là trình trạng page fault cần thực hiện load mã hoặc dữ liệu từ disk sang bộ nhớ.

**#Swapping**
Swaping xảy ra khi một số memory page được ghi ra đĩa để có bộ nhớ trống cho việc load các mã (dữ liêu) từ đĩa vào nên chắc chắn sẽ có tình trạng Major Page Fault ở tình huống này. 
Tình trạng swap sẽ ảnh hưởng lớn tới hiệu suất của server nếu Linux phải swap-out để có thể swap-in ( do Page Fault liên tục xảy ra). Tình trạng này là **thrashing**. Khi có tình trạng thrashing thì hệ thống phải tốn thời gian để xử lý page fault nhiều hơn là thực thi tiến trình do đó có thể dẫn đến trình trạng không phản hồi và tình trạng bận đĩa cao. 

