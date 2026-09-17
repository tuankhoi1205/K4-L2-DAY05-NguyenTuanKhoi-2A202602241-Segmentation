# Báo cáo Day 5 — Segmentation

- Mã học viên theo lớp: 2A202602241
- Ngày / CVAT local: 17/09/2026 · CVAT local `http://localhost:8080`
- Công cụ đã dùng: Brush, Polygon, AI detector EoMT-DINOv3 panoptic (COCO), Edit/Add poly/Remove poly trong CVAT

## 1. Bài đã nộp

Ghi tên ZIP đúng như file trong `submissions/` và số ảnh đã vẽ, Save. Chưa làm hoặc export lỗi thì ghi `chưa có`, không tạo ZIP rỗng. Cột điểm là điểm tối đa của task, **không phải điểm tự chấm**.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | `cp1_holes.zip` | 1 / 1 | 3 |
| cp2_slice | `cp2_slice.zip` | 1 / 1 | 3 |
| cp5_occlusion | `cp5_occlusion.zip` | 1 / 1 | 3 |
| cp3_thin | `cp3_thin.zip` | 1 / 1 | 3 |
| cp4_curb | `cp4_curb.zip` | 1 / 1 | 3 |
| cp6_coverage | `cp6_coverage.zip` | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Đã chạy `python scripts\inspect_submissions.py --dir submissions`: cả chín ZIP đều báo `OK`. Dòng cảnh báo của `hard_panoptic` chỉ nhắc kiểm trực quan chồng lấn và độ phủ; tôi đã xem lại trong CVAT trước khi export.

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: ảnh `000000181542.jpg`, người phụ nữ đứng ở giữa ảnh, phía trước các xe máy.
- Class và quy tắc tôi dùng để chọn biên: class `person`; tôi bám theo phần cơ thể và quần áo nhìn thấy, dừng mask tại đường viền thật của người, không lấy phần xe máy hoặc nền xung quanh và không ước lượng qua vùng bị che.
- Nếu dùng gợi ý sau đó: sau object đầu tiên tôi mới dùng EoMT-DINOv3 cho các object còn lại. Tôi giữ các vùng gợi ý bám đúng vật thể; vùng tràn nền, biên lệch hoặc object bị gộp/tách sai được chỉnh lại bằng Brush, Add poly và Remove poly.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `medium_instance`, ảnh `000000181542.jpg`, một instance bị che ở nửa phải ảnh có hai phần nhìn thấy rời nhau.
- Lỗi thuộc loại: gộp-tách — hai phần của cùng một vật ban đầu được lưu thành hai polygon và hai object ID.
- Bằng chứng tôi nhìn thấy: hai vùng có cùng class và thuộc cùng một vật lý, nhưng bảng Objects của CVAT hiển thị thành hai object riêng.
- Quy tắc và hành động sửa: áp dụng quy tắc “một vật vật lý = một mask”; tôi tạo lại một mask, thêm cả hai vùng nhìn thấy trong cùng phiên Brush/Add poly, không vẽ xuyên vùng bị che, rồi xóa polygon/object thừa.
- Sau sửa đã Save và export lại chưa? Đã Save và export lại `medium_instance.zip` ở định dạng COCO 1.0; bản ZIP mới có 65 annotations (`polygon`: 27, `RLE`: 38).

Kết quả tự đánh giá liên quan lỗi vừa sửa: chưa có điểm/reference; kiểm cấu trúc local đã `OK`. Không đưa ground truth vào fork.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `cp1_holes` · `000000144300.jpg` · kính chắn gió và các khe trên xe máy | Khoét các vùng kính/khe khỏi mask, hoặc giữ chúng trong mask xe | Quy tắc riêng của checkpoint ghi kính/khe nằm trong mask vật và không tự khoét tùy tiện | Tôi giữ kính/khe bên trong mask `motorcycle`. |
| `cp2_slice` · `000000017627.jpg` · các ô tô cùng lớp đứng sát nhau | Gộp các xe thành một mask, hoặc tách từng xe thành instance riêng | Mỗi vật thể vật lý là một instance; vẫn nhìn thấy biên/khe phân cách giữa các xe | Tôi tạo mask riêng cho từng xe, không gộp chỉ vì chúng đứng sát nhau. |
| `cp5_occlusion` · `000000336232.jpg` · xe bị vật khác che trong cảnh giao thông | Tách hai phần nhìn thấy thành hai object, hoặc giữ cùng một instance | Một vật bị che có thể có các vùng nhìn thấy rời nhau; chỉ gán phần nhìn thấy và không đoán xuyên vật che | Tôi giữ các phần nhìn thấy trong cùng một mask/instance và để trống vùng bị che. |
