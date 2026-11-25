# Báo cáo Day 13 — Nguyễn Thành Đạt, 2A202603258

## 1. Kết luận trước

Task `D13-2A202603258` có 10 frame và 68 cuboid. `s05f38` có ít nhất, 4 cuboid. Phiếu tự kiểm gắn cờ 65/68 cuboid, chủ yếu vì kích thước xe không hợp lý và chiều dài nhỏ hơn chiều rộng. Đặc biệt, toàn bộ yaw trong bảng A là 0°. Vì vậy, kết quả xuất hiện tại chưa đủ để kết luận hướng và kích thước đã đúng; cần đối chiếu lại mũi tên cuboid, Top/Side/Front và ảnh camera.

Trong CVAT, tôi dùng góc nhìn point cloud để đặt Shape, rồi đối chiếu cuboid với các hình chiếu và ảnh camera. Theo hệ tọa độ VF, phía trước xe thu dữ liệu là −y, +x hướng trái và +y hướng sau. Xe cùng chiều thường có yaw gần 270°, xe ngược chiều thường gần 90°.

## 2. Đọc frame (Bước 2)

Hai frame tôi đọc kỹ là s01f12 và s01f50. Ảnh: `t1-s01f12-3d.png`, `t1-s01f12-top.png`, `t1-s01f50-3d.png`, `t1-s01f50-top.png`. Cả hai frame cho thấy cảnh đường phố với nhiều xe; s01f12 còn có người đi bộ, s01f50 có xe hai bánh. Trong bảng A, vật được gán xa nhất ở s01f12 cách khoảng 31,5 m, còn ở s01f50 khoảng 20,6 m. Mốc mặt đường tham chiếu là z ≈ −1,7 m.

| Frame | Hướng và trục | Mặt đường, vật xa nhất | Quan sát thêm |
|---|---|---|---|
| s01f12 | Phía trước theo −y; +x sang trái, +y về sau | Đường nhìn chung tương đối phẳng; xa nhất khoảng 31,5 m | Có Car, Bus, Pedestrian; các cụm điểm nhỏ ở xa khó tách khỏi nền |
| s01f50 | Phía trước theo −y; +x sang trái, +y về sau | Đường nhìn chung tương đối phẳng; xa nhất khoảng 20,6 m | Có Car, Cyclist; camera chỉ cho thấy một phần cảnh xung quanh xe thu dữ liệu |

Camera giúp nhận màu sắc, hình dạng nhìn thấy và phân biệt đầu/đuôi xe. LiDAR cho biết vị trí, khoảng cách và hình học 3D; nó cũng ghi nhận vật ở bên hoặc phía sau ngoài góc nhìn của camera trước. Ngược lại, khi point cloud thưa, camera có thể giúp nhận ra vật nhưng không thay thế việc kiểm tra kích thước trong không gian.

Tám frame còn lại đã lướt qua: s02f08, s02f36, s03f19, s03f45, s04f01, s04f22, s05f08, s05f38. Quy ước trục không đổi giữa các frame; số lượng và mật độ điểm quanh vật thay đổi.

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh minh họa: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`. Các số đo dưới đây là ví dụ đọc từ bảng A; cần đối chiếu lại đúng cuboid trong từng ảnh trước khi nộp.

| Ví dụ | Frame, idx | Class | Kích thước l × w × h (m) | yaw | z đáy (m) | Nhận xét |
|---|---|---|---|---:|---:|---|
| Xe con gần | s02f36, 1 | Car | 2,149 × 4,003 × 1,477 | 0° | −1,841 | Dài nhỏ hơn rộng; kích thước cần xem lại |
| Xe buýt | s01f12, 2 | Bus | 5,848 × 9,543 × 3,584 | 0° | −2,314 | Dài/rộng và vị trí đáy cần kiểm tra trên ba hình chiếu |
| Người đi bộ | s01f12, 4 | Pedestrian | 0,924 × 0,793 × 1,793 | 0° | −1,419 | Kích thước gần khoảng người lớn; đáy cao hơn mốc đường khoảng 0,28 m |

Tôi cần xác nhận với ảnh camera rằng mũi tên chỉ theo hướng di chuyển hoặc đầu xe; bảng A hiện không có yaw gần 270° hay 90°. Các giá trị z đáy là phép tính từ bảng A, còn việc cuboid có thực sự chạm đường cần đánh giá lại trong Side và Front. Vòng hiệu chuẩn chung: **[ghi điểm khác biệt bạn nhận thấy và kết luận của lớp]**. Vật đã bỏ qua vì thiếu cơ sở: **[ghi frame/vị trí/loại nghi vấn, hoặc “không có”]**.

## 4. Bốn hiện tượng chất lượng (Bước 4)

Nguyên nhân dưới đây là giả thuyết khi quan sát ảnh CVAT.

| Hiện tượng | Ảnh | Quan sát và ảnh hưởng đến gán nhãn |
|---|---|---|
| Bóng che | `t4-1-bong-che.png` | Các hộp/vật chồng lấn làm khó xác định ranh giới điểm. Nếu vật bị che một phần, bật occluded và chỉ suy luận phần còn đủ bằng chứng. |
| Thưa ở xa | `t4-2-thua-o-xa.png` | Vật xa có ít điểm hơn, nên khó xác định kích thước. Chỉ vẽ khi nhận dạng được vật và còn đủ cơ sở; nếu không thì bỏ qua, ghi LOW_VISIBILITY. |
| Phản xạ thấp hoặc ít điểm bất thường | `t4-3-phan-xa-thap.png` | Point cloud đứt đoạn có thể do bề mặt hoặc góc phản xạ. Camera hỗ trợ nhận dạng, nhưng hộp vẫn cần khớp phần điểm nhìn thấy. |
| Điểm lạ | `t4-4-diem-la.png` | Một số điểm rời không tạo thành vật có hình dạng rõ. Không gán riêng các điểm đó thành cuboid. |

## 5. Số liệu cuboid và tự kiểm (Bước 5)

Mặt đường tham chiếu khoảng −1,7 m. Các số `r` được tính từ `√(x² + y² + z²)`; số điểm lấy từ cột `points_in` của phiếu tự kiểm.

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy (m) | dist_m (m) | r (m) | yaw | Nhận xét so với mặt đường |
|---|---:|---:|---:|---:|---:|---:|---|
| s02f36, 1, Car | −1,102 | 1,477 | −1,841 | 4,31 | 4,45 | 0° | Thấp hơn mốc khoảng 0,14 m |
| s01f12, 4, Pedestrian | −0,523 | 1,793 | −1,419 | 7,54 | 7,56 | 0° | Cao hơn mốc khoảng 0,28 m |
| s05f08, 8, Car | −1,247 | 1,735 | −2,114 | 34,33 | 34,35 | 0° | Thấp hơn mốc khoảng 0,41 m |

Cuboid gần nhất là s02f36 idx 1 (4,31 m theo mặt phẳng, 4,45 m theo khoảng cách 3D); xa nhất là s05f08 idx 8 (34,33 m và 34,35 m). Xe con s02f36 idx 1 có kích thước 2,149 × 4,003 × 1,477 m, trong đó dài nhỏ hơn rộng và cả hai số đầu đều không phù hợp với xe con thông thường. Pedestrian s01f12 idx 4 có kích thước 0,924 × 0,793 × 1,793 m; dài nhỉnh hơn khoảng tham chiếu người lớn, rộng và cao nằm trong khoảng tham chiếu.

| Cuboid | dist_m (m) | points_in |
|---|---:|---:|
| s02f36, 1, Car | 4,31 | 2421 |
| s01f12, 4, Pedestrian | 7,54 | 453 |
| s02f08, 1, Bus | 15,20 | 4974 |
| s05f08, 8, Car | 34,33 | 145 |

Nhìn chung, vật xa thường có ít điểm hơn, dù số điểm còn phụ thuộc kích thước và loại vật. Ví dụ, xe con gần nhất có 2421 điểm, còn xe con xa nhất trong bảng có 145 điểm. Xu hướng này phù hợp với đường cong tham chiếu trên lớp: xe con có trung vị khoảng 1968 điểm ở 0–10 m, giảm còn khoảng 19 điểm ở 60–80 m.

Phiếu tự kiểm gắn cờ 65 cuboid. Hai cuboid tôi đã sửa sau khi xem preview/selfcheck: **[frame, idx, nội dung sửa]**. Cảnh báo tôi giữ lại và lý do: **[frame, idx, lý do]**. Đối chiếu chéo với bạn MSSV **[MSSV]**: hai số liệu được kiểm tra là **[tên số liệu và kết quả khớp/lệch]**.

## 6. Ý nghĩa nhãn, AI và điều còn chưa chắc

Nhãn 3D dùng mét để mô tả vị trí và kích thước vật trong không gian, giúp mô hình học khoảng cách và hình học thật thay vì vị trí pixel phụ thuộc góc camera. Vật thưa điểm cần quy tắc riêng vì hình dạng chỉ được quan sát một phần; chỉ nên gán khi có đủ bằng chứng, nếu không thì bỏ qua và ghi nhận trường hợp chưa chắc.

- Công cụ AI: ChatGPT.
- Câu hỏi chính: đọc bảng A và phiếu tự kiểm, tính khoảng cách/đáy, rồi hỗ trợ soạn bản nháp báo cáo.
- Cách dùng kết quả: đối chiếu số với CSV và ảnh CVAT; tôi sẽ tự xác nhận các nhận xét và thông tin cá nhân trước khi nộp.

Điều tôi còn chưa chắc là vì sao yaw trong bảng A đều bằng 0° và nhiều xe có `length < width`. Tôi sẽ kiểm tra lại mũi tên, kích thước và các hình chiếu trong CVAT; nếu bảng xuất vẫn như vậy sau khi kiểm tra, tôi sẽ hỏi coach về cách diễn giải dữ liệu.
