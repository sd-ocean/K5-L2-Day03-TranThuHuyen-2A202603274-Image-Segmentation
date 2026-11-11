# Báo cáo thực hành Day 3

- Mã học viên theo lớp: 2A202603274
- Ngày / CVAT local: 11/11-12/11/2026
- Công cụ đã dùng: CVAT, OpenCV, SAM3 (text detector và interactor qua nuclio) cho gợi ý mask, sau đó tự kiểm và sửa tay.

## 1. Bài đã nộp

Ghi tên ZIP đúng như file trong `submissions/` và số ảnh đã vẽ, Save. Chưa làm hoặc export lỗi thì ghi `chưa có`, không tạo ZIP rỗng. Cột điểm là điểm tối đa của task, **không phải điểm tự chấm**.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |


## 2. Một quyết định trước khi dùng gợi ý

Chọn object đầu tiên bạn tự vẽ ở `medium_instance`, trước khi xem bất kỳ đề xuất tự động nào cho object đó. Ghi ảnh/vị trí đủ để tìm lại; “quy tắc biên” là lý do bạn chọn hoặc dừng mask ở ranh đó.

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: người mặc áo dài ở giữa ảnh 000000181542.jpg.
- Class và quy tắc tôi dùng để chọn biên: class person, biên theo viền áo, không lấy bóng dưới chân.
- Nếu dùng gợi ý sau đó: dùng openCV tool để vẽ sky nhưng bị nhầm sang phần vegetation, đã căn chỉnh lại cho chính xác.
- Nếu không dùng gợi ý: có dùng như đã báo bên trên.

## 3. Một lỗi tôi tìm thấy và sửa

Chọn một lỗi **có thật** trong bài. Nếu công cụ lỗi khiến bạn chưa sửa được, ghi rõ đã thử gì và cần coach hỗ trợ gì; không ghi “đã sửa” khi chưa sửa.

- Task/ảnh/vùng: `hard_panoptic`, cả hai ảnh 000000350023 và 000000460147; toàn bộ vùng stuff.
- Lỗi thuộc loại: phủ vùng, thiếu toàn bộ 5 class stuff: road, sidewalk, building, vegetation, sky.
- Bằng chứng nhìn thấy: bản export COCO đầu tiên có 71 mask nhưng tất cả là thing (car, person, bus, truck, motorcycle, traffic light); `scripts/inspect_submissions.py` cảnh báo "chưa thấy stuff class: building, road, sidewalk, sky, vegetation".
- Quy tắc và hành động sửa: panoptic phải có cả stuff lẫn từng thing. Đã bổ sung road, sidewalk, building, vegetation, sky cho cả hai ảnh, sử dụng gợi ý SAM3 rồi chỉnh tay, sau đó export lần nữa để bổ sung vegetation còn thiếu ở ảnh 000000460147.
- Sau sửa đã Save và export lại chưa? Đã Save và export lại COCO 1.0; `inspect_submissions.py` báo OK cho cả 9 task.

Theo kết quả chấm được sau khi ground truth ba tier được phát (bản đã sửa lỗi trên, chưa sửa thêm theo đáp án): easy_semantic mIoU 0.872 → 20.0/20; medium_instance matched IoU 0.805, R@0.5 0.94 → 25.6/32; hard_panoptic PQ 0.355 (SQ 0.566, RQ 0.471) → 10.4/30; tổng ba tier khoảng 56/82. Phản hồi cho thấy lỗi còn lại lớn nhất là thiếu vegetation ở ảnh 000000350023 (IoU 0.00) và xe ở xa bị tính FP. Không có điểm trước khi sửa lỗi stuff vì lúc đó chưa có ground truth.


## 4. Ba ca chưa chắc hoặc đã cân nhắc

Mỗi ca là một **vùng cụ thể** khiến bạn phải cân nhắc hai cách hiểu. Ghi dấu hiệu nhìn thấy hoặc quy tắc đã dùng, rồi nêu quyết định hoặc câu hỏi cho coach. Không cần ba lỗi; ca đã quyết định được cũng hợp lệ.

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `easy_semantic` 817bca71-00000000, trước các toà nhà | (a) `yard` vì là khu trống đằng trước building; (b) `sidewalk` như làn đường riêng bo theo đường xe chạy, nhưng cũng bao gồm khu vực có thể đi như yard | Ranh yard–sidewalk theo chức năng và ngăn cách bơi thảm cỏ; cũng có 1 số ô tô đang đỗ ở yard. | Chỉ gán nhãn vùng sidewalk thẳng bo theo đường xe chạy, không lấy khoảng không rộng phía trước building. |
| `hard_panoptic` 000000460147, xe chở ô tô ở giữa ảnh | (a) một `truck` gồm cả các ô tô đang chở; (b) `truck` cho xe chở và mỗi ô tô trên đó là một `car` riêng | Các ô tô trên xe nhìn thấy được rõ, đếm được; nhưng chúng không tham gia giao thông, mà được vận chuyển | Hỏi coach xem nên gán thành `truck` chở vật thể như thường, hay tách `car` riêng. |
| `hard_panoptic` 000000460147, xe van tối màu ở mép trái và các xe rất nhỏ ở xa | (a) van là `car`, xe xa vẽ từng chiếc; (b) van là `truck`, xe quá nhỏ (<15 px) không tách riêng | Van chở người, kích thước như ô tô con; xe ở xa vẫn nhìn thấy nhưng biên không rõ | Hỏi coach xem ngưỡng kích thước tối thiểu cho để gán nhãn cho từng thing. |
