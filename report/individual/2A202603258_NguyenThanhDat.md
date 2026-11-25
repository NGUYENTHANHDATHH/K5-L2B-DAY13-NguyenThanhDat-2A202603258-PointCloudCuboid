# Báo cáo Day 13 — Nguyễn Thành Đạt, 2A202603258

> **Cách dùng mẫu này:** đổi tên file thành `<MSSV>_<HoTen>.md` (họ tên không dấu, viết liền), ví dụ `2A202600123_NguyenVanAn.md`. Đổi tên, đừng chép thành file thứ hai.
>
> Xoá mọi dấu TODO và dòng gợi ý (dòng bắt đầu bằng `>`). Mỗi con số lấy từ **bảng A** hoặc **phiếu tự kiểm** trong `submission/`, kèm đơn vị (m, °). Ảnh để trong `report/individual/images/` (nộp LMS trong file zip, không commit); trong báo cáo chỉ ghi **tên file ảnh**. Báo cáo này nằm trên repo public: không chép biển số, chữ trên biển hiệu hay biển chỉ đường, tên đường, địa danh; nhắc tới bạn khác thì ghi MSSV, không ghi họ tên. Cả báo cáo ~2 trang.

## 1. Kết luận trước

Mã số `2A202603258`, task `D13-2A202603258`, 10 frame: s01f12, s01f50, s02f08, s02f36, s03f19, s03f45, s04f01, s04f22, s05f08, s05f38. Windows 11 Pro, trình duyệt chrome.

Tổng 68 cuboid, frame pointcloud/s03f45.pcd ít nhất vì đoạn đường 1 chiều không giao với nhiều phương tiện; `selfcheck` cờ bao 65 cuboid trên tổng số;

TODO(học viên): cách dùng CVAT, 3 dòng: đổi góc nhìn (nhìn từ trên xuống, xoay, dời); vẽ và chỉnh một cuboid (Shape, nháy đúp, Top/Side/Front); bật mũi tên hướng (Cuboid orientation) và xem cuboid trên ảnh camera (`preview`).

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ. Ảnh: `t1-s01f12.pcd-3d`, `t1-s01f12.pcd-top` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

| Frame  | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m)                     | Vật xa nhất đọc được (~ m)                                          | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
| ------ | ---------------------------------------------------------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------- | ------------- | ---------------------- | ----------------- |
| s01f12 | Phía trước theo −y; +x sang trái, +y về sau                            | Đường tương đối phẳng; vật xa nhất trong bảng A khoảng 31,5 m | Car, Bus, Pedestrian; các cụm điểm mảnh ở rìa khó phân biệt với nền |
| s01f50 | Phía trước theo −y; +x sang trái, +y về sau                            | Đường tương đối phẳng; vật xa nhất trong bảng A khoảng 20,6 m | Car, Cyclist; có xe ở ngoài vùng nhìn của camera trước              |

Camera giúp nhận màu sắc, hình dạng nhìn thấy và phân biệt đầu/đuôi xe. LiDAR cho biết vị trí, khoảng cách và hình học 3D; nó cũng ghi nhận vật ở bên hoặc phía sau ngoài góc nhìn của camera trước. Ngược lại, khi point cloud thưa, camera có thể giúp nhận ra vật nhưng không thay thế việc kiểm tra kích thước trong không gian.

Tám frame lướt:
Tám frame còn lại đã lướt qua: s02f08, s02f36, s03f19, s03f45, s04f01, s04f22, s05f08, s05f38. Quy ước trục không đổi giữa các frame; số lượng và mật độ điểm quanh vật thay đổi.

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
| ----- | ------------------------------------------------ | ---------------- |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật         | Frame, idx | Class      | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
| ----------- | ---------- | ---------- | ----------------------- | --------------- | ----------------------------- | -------------------------- |
| Xe con gần  | s02f36, 1  | Car        | ngược                   | không           | có                            | 2,149 × 4,003 × 1,477      |
| Xe buýt     | s01f12, 2  | Bus        | có                      | không           | có                            | 5,848 × 9,543 × 3,584      |
| Người đi bộ | s01f12, 4  | Pedestrian | có                      | không           | có                            | 0,924 × 0,793 × 1,793      |

## 4. Bốn hiện tượng chất lượng (Bước 4)

Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng                           | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết)                                                                                                                     | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
| ------------------------------------ | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------ | ------------------------------- |
| Bóng che                             | `t4-1-bong-che.png`     | Các hộp/vật chồng lấn làm khó xác định ranh giới điểm. Nếu vật bị che một phần, bật occluded và chỉ suy luận phần còn đủ bằng chứng.         |
| Thưa ở xa                            | `t4-2-thua-o-xa.png`    | Vật xa có ít điểm hơn, nên khó xác định kích thước. Chỉ vẽ khi nhận dạng được vật và còn đủ cơ sở; nếu không thì bỏ qua, ghi LOW_VISIBILITY. |
| Phản xạ thấp hoặc ít điểm bất thường | `t4-3-phan-xa-thap.png` | Point cloud đứt đoạn có thể do bề mặt hoặc góc phản xạ. Camera hỗ trợ nhận dạng, nhưng hộp vẫn cần khớp phần điểm nhìn thấy.                 |
| Điểm lạ                              | `t4-4-diem-la.png`      | Một số điểm rời không tạo thành vật có hình dạng rõ. Không gán riêng các điểm đó thành cuboid.                                               |

## 5. Số từ cuboid của tôi (Bước 5)

Lấy từ `submission/D13-<MÃ SỐ>-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z      | h     | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
| -------------------------- | ------ | ----- | --------------- | ------ | --------------- | ------- | -------------------------------------- |
| s02f36, 1, Car             | −1,102 | 1,477 | −1,841          | 4,31   | 4,45            | 0°      | Thấp hơn mốc khoảng 0,14 m             |
| s01f12, 4, Pedestrian      | −0,523 | 1,793 | −1,419          | 7,54   | 7,56            | 0°      | Cao hơn mốc khoảng 0,28 m              |
| s05f08, 8, Car             | −1,247 | 1,735 | −2,114          | 34,33  | 34,35           | 0°      | Thấp hơn mốc khoảng 0,41 m             |

Cuboid gần nhất là s02f36 idx 1 (4,31 m theo mặt phẳng, 4,45 m theo khoảng cách 3D); xa nhất là s05f08 idx 8 (34,33 m và 34,35 m). Xe con s02f36 idx 1 có kích thước 2,149 × 4,003 × 1,477 m, trong đó dài nhỏ hơn rộng và cả hai số đầu đều không phù hợp với xe con thông thường. Pedestrian s01f12 idx 4 có kích thước 0,924 × 0,793 × 1,793 m; dài nhỉnh hơn khoảng tham chiếu người lớn, rộng và cao nằm trong khoảng tham chiếu.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
| -------------------------- | ------ | --------------------- |
| s02f08, 1, Bus             | 15,20  | 4974                  |
| s05f08, 8, Car             | 34,33  | 145                   |

Nhìn chung, vật xa thường có ít điểm hơn, dù số điểm còn phụ thuộc kích thước và loại vật. Ví dụ, xe con gần nhất có 2421 điểm, còn xe con xa nhất trong bảng có 145 điểm. Xu hướng này phù hợp với đường cong tham chiếu trên lớp: xe con có trung vị khoảng 1968 điểm ở 0–10 m, giảm còn khoảng 19 điểm ở 60–80 m.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Nhãn 3D dùng mét để mô tả vị trí và kích thước vật trong không gian, giúp mô hình học khoảng cách và hình học thật thay vì vị trí pixel phụ thuộc góc camera. Vật thưa điểm cần quy tắc riêng vì hình dạng chỉ được quan sát một phần; chỉ nên gán khi có đủ bằng chứng, nếu không thì bỏ qua và ghi nhận trường hợp chưa chắc.
