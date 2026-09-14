# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Đặng Thanh Tiến<br>
**MSSV:** 2A202602099<br>
**Hình thức:** Cá nhân<br>
**Mã cặp:** `SOLO`

## 1. Phạm vi

- Chỉ gán phương tiện thuộc bốn lớp bên dưới.
- Mỗi phương tiện là một hộp; không gộp nhiều xe.
- Không gán người, xe máy, xe đạp, biển báo hoặc phần phản chiếu.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; ghi lý do vào nhật ký quyết định.

## 2. Bốn lớp cố định

|  Mã | Lớp              | Gán khi nhìn thấy                                       | Không gán vào lớp này                             |
| --: | ---------------- | ------------------------------------------------------- | ------------------------------------------------- |
|   0 | `car` (ô tô con) | sedan, hatchback, SUV, taxi, xe bán tải dùng như xe con | xe có thùng/ben rõ; thân xe buýt; xe van thân hộp |
|   1 | `truck` (xe tải) | thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng      | ô tô con; thân xe buýt; xe van kín một khối       |
|   2 | `bus` (xe buýt)  | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế           | xe van nhỏ; xe tải; ô tô con                      |
|   3 | `van` (xe van)   | thân hộp nhỏ, kín, dùng chở người hoặc hàng             | thân xe buýt; khoang hàng tách biệt như xe tải    |

Thứ tự lớp là cố định: `0 car, 1 truck, 2 bus, 3 van`.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy.
- Không ước lượng phần bị xe khác che.
- Vật thể chạm mép ảnh vẫn được gán nếu đủ bằng chứng phân lớp.
- Không để hộp chứa nhiều nền hoặc nhiều phương tiện.

## 4. Ba thuộc tính

| Thuộc tính                          | Giá trị                                                 | Ý nghĩa                             |
| ----------------------------------- | ------------------------------------------------------- | ----------------------------------- |
| `visibility` (mức nhìn thấy)        | `clear` (rõ), `occluded` (bị che), `unclear` (không rõ) | mức bằng chứng nhìn thấy            |
| `boundary` (quan hệ mép ảnh)        | `inside` (trong ảnh), `truncated` (bị cắt)              | vật thể có bị mép ảnh cắt hay không |
| `review_state` (trạng thái xem lại) | `confident` (tự tin), `needs_review` (cần xem lại)      | đánh dấu quyết định cần quay lại    |

YOLO không lưu ba thuộc tính này. Vì vậy phải xuất thêm `CVAT for images 1.1` từ cùng công việc.

## 5. Ba tình huống mơ hồ

Hoàn thành trước khi xem bài của người khác hoặc bộ nhãn tham chiếu.

### Tình huống A — xe buýt hay xe van?

- Ảnh và mã vật thể: `drive_033` / `obj_03`
- Dấu hiệu nhìn thấy: Khung xe dài, thân cao, kéo dài dọc thân là dãy kính cửa sổ khách nhiều ô nhưng kích thước tổng thể không quá lớn như xe buýt 45 chỗ.
- Quy tắc áp dụng: Phân biệt theo quy định lớp: `bus` (xe buýt) là xe khách dài, nhiều hàng ghế/cửa sổ khách; `van` (xe van) là xe thân hộp nhỏ kín chở người/hàng nhỏ. Nếu khung thân dài có cấu trúc cửa sổ xe khách chở trên 16 người thì gán `bus`.
- Quyết định: Gán lớp `2 bus` (`review_state=confident`, `visibility=clear`, `boundary=inside`).
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Phóng toàn bộ vùng ảnh 100%, kiểm tra tỷ lệ chiều dài/chiều cao thân xe và vị trí các hàng ghế. Nếu ranh giới giữa xe buýt nhỏ (minibus) và xe van 16 chỗ vẫn mơ hồ, đặt `review_state=needs_review` và ghi rõ lý do nghi vấn để thảo luận thống nhất với Lab Coach.

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: `drive_022` / `obj_05`
- Dấu hiệu nhìn thấy: Phần cabin phía trước tách biệt với thùng chở hàng phía sau có cấu trúc hình hộp vuông kim loại bằng phẳng.
- Quy tắc áp dụng: `truck` (xe tải) phải có thùng, ben, sàn chở hàng hoặc thiết bị công vụ rõ ràng. Ngược lại, xe van có thân hộp kín liền khối, không có khoang hàng tách biệt.
- Quyết định: Gán lớp `1 truck` (`review_state=confident`, `visibility=clear`, `boundary=inside`).
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Phóng to 100% ranh giới giữa cabin lái và khoang chứa hàng. Nếu khoang chứa hàng liền một khối với cabin (dạng xe van giao hàng), gán `van`. Nếu có khe hở/khớp nối tách biệt cabin và thùng hàng, gán `truck`.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: `drive_008` / `obj_12`
- Dấu hiệu nhìn thấy khi phóng 100%: Nhìn thấy rõ nắp capo, kính chắn gió trước và 2 bánh trước của ô tô con, phần đuôi bị mép ảnh cắt ngang và một phần bên hông bị ô tô phía trước che khuất khoảng 30%.
- Giá trị `visibility`: `occluded`
- Giá trị `boundary`: `truncated`
- Trạng thái `review_state`: `confident`
- Lý do: Phần thân nhìn thấy (>60%) có đầy đủ dấu hiệu nhận diện đặc trưng của ô tô con (`car`). Hộp giới hạn được vẽ sát viền phần thân thực tế nhìn thấy, không vẽ bao trùm phần khuất ngoài mép ảnh hay phần bị xe trước che.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh.
- [x] Đã kiểm vật thể thiếu và trùng.
- [x] Đã kiểm lớp và hình học từng hộp.
- [x] Mỗi hộp có đủ ba thuộc tính.
- [x] Đã xử lý mọi hộp `needs_review`.
- [x] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu.
- [x] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi.
- [x] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu.
- [x] Số vật thể thực tế: 52 — 40–60 là mục tiêu khối lượng, không phải điểm cắt.
