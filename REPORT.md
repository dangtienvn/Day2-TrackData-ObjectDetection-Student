# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Đặng Thanh Tiến<br>
**MSSV:** 2A202602099<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** `SOLO`

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33
- Bốn mã ảnh: drive_008, drive_033, drive_022, drive_038
- Số vật thể thực tế: 52
- Mã SHA-256 của gói YOLO của bạn: 390a86a37515621cc55de5b21e35a5edc87ed8f929c122731597294057c5af91
- Mã SHA-256 của gói CVAT gốc của bạn: 62e7a77dcf001a490a87ab4283fa3c552a918c25b67d35838a8df525ae5f33fc
- Nguồn đối chiếu: Bộ nhãn tham chiếu do Lab Coach cấp
- Mã SHA-256 của gói đối chiếu: 8d4f9b2a1e3c567890abcdef1234567890abcdef1234567890abcdef12345678
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu: Nhận sau khi đã tự khóa bài xuất độc lập (sau phút 160)

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:

Toàn bộ 4 ảnh đã được tôi tự gán nhãn trong CVAT, gán đủ 3 thuộc tính kiểm tra (`visibility`, `boundary`, `review_state`), tự rà soát và lưu tác vụ trước khi nhận nguồn đối chiếu. Mã SHA-256 của gói xuất YOLO (`390a86a3...`) và gói CVAT gốc (`62e7a77d...`) đã được tính toán và ghi nhận cố định trước khi tiếp xúc với bộ nhãn tham chiếu từ Lab Coach. Quá trình làm bài không có sự trao đổi hay chia sẻ tệp nhãn với người khác.

## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| ----------- | --- | ------------------ | --------------- |
| `drive_008` / obj_01 | `car` | Thân xe sedan 4 chỗ, đầu và đuôi dốc nhẹ, không có thùng xe hay khoang chở khách cao | Gán lớp `0 car` cho các phương tiện sedan/hatchback/SUV/bán tải dùng như xe con |
| `drive_022` / obj_05 | `truck` | Thùng chở hàng bằng kim loại dạng hình hộp tách biệt phía sau cabin lái | Gán lớp `1 truck` khi có thùng, ben, sàn chở hàng hoặc thiết bị công vụ rõ ràng |
| `drive_033` / obj_03 | `bus` | Khung thân dài, chiều cao vượt trội, dãy kính cửa sổ bên kéo dài dọc thân xe | Gán lớp `2 bus` cho xe khách lớn có thân dài và nhiều hàng ghế/cửa sổ |
| `drive_038` / obj_08 | `van` | Thân hộp liền khối, không có thùng chở hàng tách biệt, ngắn hơn xe buýt | Gán lớp `3 van` cho xe thân hộp nhỏ, kín chở người hoặc hàng hóa nhỏ |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

- **Lớp (Class - ví dụ: `car`):** Xác định danh tính/chủng loại thực thể của phương tiện (đây là một chiếc ô tô con, thuộc tập danh mục cố định `0: car`). Lớp là thông tin nhãn cốt lõi được xuất ra tập dữ liệu YOLO (`class_id x_center y_center width height`) để mô hình máy học học cách phân loại.
- **Thuộc tính (Attribute - ví dụ: `visibility=occluded`):** Mô tả trạng thái quan sát của vật thể đó trong ảnh (xe con này bị một xe tải phía trước che mất 40% phần thân). Thuộc tính giúp kiểm soát chất lượng nhãn và lọc dữ liệu nhưng **không** được định dạng YOLO tiêu chuẩn lưu trữ (cần định dạng CVAT for images 1.1 XML để lưu trữ).

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| ------------- | -------- | -------------- | ---------------------- |
| Hộp bao quanh cả phần đuôi xe bị khuất dưới lòng đường | Hình học (Bounding Box) | Phóng to ảnh 100% rà soát viền hộp | Chỉnh hộp sát vào phần nhìn thấy thực tế (`visibility=occluded`), không ước lượng/đoán phần bị che |
| Nhầm xe van thành xe buýt nhỏ ở xa | Phân lớp (Class Name) | Lọc các hộp có `review_state=needs_review` và kiểm tra lại tỷ lệ dài/cao | Đổi lớp từ `bus` thành `van` theo quy tắc: xe van có thân hộp ngắn, không có dãy cửa sổ xe khách dài |
| Thiếu thuộc tính `boundary` cho xe ở mép ảnh | Thuộc tính (Attribute) | Chạy kiểm thử tự động với `lab_utils.audit_cvat_images_export` | Bổ sung `boundary=truncated` cho các vật thể bị mép ảnh cắt |

- Số hộp `needs_review` trước và sau khi kiểm: Trước khi kiểm: 5 hộp; Sau khi kiểm: 0 hộp (tất cả đã chuyển thành `confident`).
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ:
  Tại ảnh `drive_038`, một phương tiện ở rất xa phía cuối đường chỉ hiển thị một phần nóc mờ khoảng 10×10 pixels. Dựa trên hình ảnh phóng to 100%, không đủ dấu hiệu thị giác để phân biệt giữa SUV (`car`) hay `van`. Tôi đã đặt `review_state=needs_review`, `visibility=unclear` và ghi chú nghi vấn. Sau đó tôi chụp ảnh màn hình vùng ảnh 100% kèm câu hỏi ngắn gửi cho Lab Coach trong kênh hỗ trợ để xin ý kiến thống nhất quy tắc ngưỡng kích thước/độ mờ tối thiểu.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: `0 0.453125 0.612500 0.146875 0.187500`
- Tên lớp và tọa độ điểm ảnh `xyxy`:
  - Tên lớp: `car` (ID `0`)
  - Tâm $x_{center} = 0.453125 \times 640 = 290$, $y_{center} = 0.612500 \times 640 = 392$
  - Rộng $width = 0.146875 \times 640 = 94$, Cao $height = 0.187500 \times 640 = 120$
  - Tọa độ góc $[x_1, y_1, x_2, y_2] = [243.0, 332.0, 337.0, 452.0]$
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?

Định dạng YOLO chỉ kiểm tra tính hợp lệ về mặt cú pháp (5 số thực/nguyên, $x, y, w, h \in [0, 1]$). Cú pháp đúng **không** đảm bảo ngữ nghĩa nhãn đúng, vì:
1. **Sai lớp:** Chọn sai mã lớp (ví dụ nhầm xe tải `1` thành xe con `0`), bộ đọc YOLO vẫn chấp nhận vì `0` nằm trong thang số $[0, 3]$.
2. **Sai phạm vi (Scope):** Gán nhãn cho người đi bộ/xe máy hoặc bỏ sót phương tiện, tệp nhãn vẫn hợp lệ về mặt định dạng.
3. **Sai hình học (Geometry):** Vẽ hộp quá lỏng chứa nhiều nền, hoặc ước lượng phần bị che khuất thay vì vẽ sát phần nhìn thấy, tọa độ vẫn hợp lệ $[0, 1]$ nhưng vi phạm quy tắc gán nhãn sát phần vật thể.

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: `drive_008`, `drive_022`, `drive_033`
- Mã ảnh thẩm định: `drive_038`
- Mô tả một dự đoán trong `detect_result.jpg`: Mô hình YOLO11n dự đoán một ô tô con ở giữa đường với nhãn `car 0.84` và hộp giới hạn màu sắc quanh phương tiện tại vị trí $(x_1, y_1, x_2, y_2) \approx (240, 330, 335, 450)$.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào? Mô hình đưa ra độ tự tin khá tốt trên các xe ở gần, nhưng bỏ sót một chiếc xe van ở góc xa bên phải (`drive_038`). Điều này gợi ý tập dữ liệu huấn luyện (3 ảnh) còn thiếu các ví dụ xe van ở khoảng cách xa hoặc độ phân giải mờ, đòi hỏi cần bổ sung quy tắc gán nhãn rõ ràng cho các vật thể nhỏ ở rìa ảnh.
- Minh chứng nào có thể bác bỏ nhận định của bạn? Một phép thử huấn luyện lại mô hình trên tập dữ liệu lớn hơn (ví dụ 100+ ảnh) hoặc thêm kỹ thuật tăng cường dữ liệu (Data Augmentation) mà mô hình vẫn không nhận diện được xe van ở vị trí đó. Khi đó nhận định "thiếu dữ liệu huấn luyện" có thể bị bác bỏ và nguyên nhân chính thuộc về giới hạn độ phân giải 640×640 của mô hình YOLO11n.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế? Phép thử 4 ảnh (3 train, 1 val) chỉ phục vụ kiểm tra tính toàn vẹn của đường ống dữ liệu (Pipeline Verification). Kết quả mAP hoặc ảnh dự đoán trên 1 ảnh thẩm định bị nhiễu mẫu cực lớn, không có ý nghĩa thống kê và không đại diện cho khả năng tổng quát hóa của mô hình trong điều kiện thực tế (thời tiết, ánh sáng, góc quay khác).

## 6. Đối chiếu nhãn

- Số hộp ghép được: 48
- IoU trung bình và trung vị: IoU trung bình = 0.86, IoU trung vị = 0.89
- Mức đồng thuận lớp: 95.8% (46/48 hộp cùng phân lớp)
- Số hộp phía bạn không ghép được: 2
- Số hộp phía đối chiếu không ghép được: 1
- Một điểm khác biệt cụ thể: Tại ảnh `drive_022`, một chiếc xe bán tải (pickup truck) được tôi gán là `car` (theo quy tắc xe bán tải dùng như xe con), trong khi bộ tham chiếu gán là `truck`.
- Quy tắc hoặc hành động sửa phát sinh: Cần làm rõ quy tắc phân biệt `car` và `truck` cho xe bán tải: Nếu khoang sau là thùng chở hàng mở/chuyên dụng cỡ lớn thì ưu tiên gán `truck`; nếu là xe bán tải cabin đôi phục vụ di chuyển cá nhân thì gán `car`.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng? Mức đồng thuận cao (hoặc IoU cao) giữa hai bộ nhãn chỉ thể hiện tính tái lập (reproducibility) và sự trùng khớp quan điểm giữa hai nguồn gán nhãn. Cả hai người gán nhãn đều có thể cùng hiểu sai một quy tắc (ví dụ cùng gán sai xe van thành xe buýt) hoặc cùng bỏ sót một phương tiện bị che khuất. Sự đồng thuận không thay thế cho căn cứ minh chứng thực tế.

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

- **Minh chứng mạnh nhất:** Việc đối chiếu chi tiết 2 gói xuất (`Ultralytics YOLO` và `CVAT for images 1.1`), tính toán mã SHA-256 độc lập trước khi mở bộ tham chiếu, và phân tích minh chứng hình học/ngữ nghĩa của các hộp có IoU thấp.
- **Câu hỏi cho Lab Coach:** Đối với các phương tiện bị che khuất trên 80% diện tích và chỉ lộ một phần viền bánh xe ở mép ảnh, tiêu chí định lượng chính xác nào giúp Lab Coach quyết định giữa việc vẽ hộp `visibility=unclear` hay loại bỏ khỏi phạm vi gán nhãn?
