# Báo cáo Day 13 — Lương Đức Mạnh, 2A202603155

## 1. Kết luận trước
- **Thông tin:** MSSV `2A202603155`, task `D13-2A202603155`, 10 frame: `s01f16`, `s01f58`, `s02f16`, `s02f37`, `s03f02`, `s03f19`, `s04f07`, `s04f57`, `s05f24`, `s05f45`; hệ điều hành Windows, trình duyệt Microsoft Edge, CVAT phiên bản `2.82.1`; đổi kiểm với bạn ngồi cạnh (cập nhật MSSV trước khi nộp chính thức nếu có thay đổi).
- **Tổng quan:** Tổng cộng đã vẽ **95 cuboid** trên 10 frame (`s01f16`: 8, `s01f58`: 9, `s02f16`: 9, `s02f37`: 8, `s03f02`: 10, `s03f19`: 10, `s04f07`: 11, `s04f57`: 9, `s05f24`: 11, `s05f45`: 10). Hai frame ít nhất là `s01f16` và `s02f37` (đều 8 cuboid) do mật độ phương tiện trong tầm 30 m thưa hơn và một số vật ở làn bên bị dải phân cách/cây che khuất. `selfcheck` gắn cờ **5/95 cuboid** (chủ yếu cảnh báo điểm lân cận sát mép hộp hoặc độ dốc mặt đường ở xa). Phần tin cậy nhất là phân loại và hướng (`yaw`) của xe ở tầm gần (< 25 m) nhờ đối chiếu đủ 7 camera; phần còn yếu nhất là ước lượng phần khuất của vật ở xa (> 35 m) khi điểm LiDAR thưa.
- **Cách dùng CVAT 3D:**
  1. Đổi góc nhìn trong khung 3D (Perspective): kéo chuột trái để xoay (xoay thẳng từ trên xuống để nhìn toàn cảnh BEV), kéo chuột phải để dời khung nhìn, lăn con lăn để phóng to/thu nhỏ, nháy đúp chỗ trống để về góc mặc định.
  2. Vẽ và chỉnh cuboid: chọn công cụ vẽ cuboid → chọn đúng class → bấm **Shape** (không dùng Track) → nháy đúp lên cụm điểm để đặt; sau đó chỉnh tâm, dài/rộng và góc xoay ở khung **Top**, chỉnh độ cao đáy chạm đường và nóc ở khung **Side** và **Front** (không xoay ở Side/Front).
  3. Bật mũi tên hướng tại **Appearance → Cuboid orientation** để căn vạch đỏ `+X` hướng về đầu xe, bấm **Save**, rồi chạy `python scripts/lab.py preview` để kiểm tra cuboid chiếu lên 7 ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f16` (`t1-s01f16-3d.png`, `t1-s01f16-top.png`) và `s01f58` (`t1-s01f58-3d.png`, `t1-s01f58-top.png`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f16` | Phía trước (`−y`) hướng lên cạnh trên; `+x` chỉ sang trái, `+y` chỉ xuống cạnh dưới (phía sau) | Khá phẳng, độ dốc rất nhỏ (~0.004); thấp hơn LiDAR ~1.66 m (`z ≈ −1.66 m`) | ~47.4 m (xe con phía sau) và ~30 m phía trước (xe buýt ở 29.0 m) | Car, Bus | Xe con xanh bên trái-sau (`cam05`/`cam09`, ~5.85 m), hai xe con phía sau (`cam09`, ~18–20 m), xe con sát làn trái (`cam06`, ~9.02 m) | Một số cụm điểm thưa phía sau hàng cây/dải phân cách bên phải (> 28 m) không tách rõ được vật thể hay bụi cây |
| `s01f58` | Phía trước (`−y`) hướng lên cạnh trên; `+x` chỉ sang trái, `+y` chỉ xuống cạnh dưới (phía sau) | Phẳng, hơi nghiêng ngang nhẹ (~−0.013); thấp hơn LiDAR ~1.67 m (`z ≈ −1.67 m`) | ~35.5 m (xe tải phía trước) | Car, Truck, Pedestrian | Xe con bạc sát bên trái (`cam05`/`cam06`, ~3.20 m), xe con trắng phía sau (`cam09`, ~6.96 m), người đi bộ bên vỉa hè phải (`cam08`, ~22.46 m) | Một vài cụm nhỏ phía vỉa hè xa bên phải (> 35 m) lẫn giữa gốc cây và xe đạp dựng, không đủ cơ sở phân biệt |

- **3 điều chỉ camera cho biết** (từ `s01f16` và `s01f58`): (1) Màu sắc thân xe, đèn tín hiệu, vạch kẻ đường và biển chỉ dẫn giao thông; (2) Phân biệt rõ đầu xe hay đuôi xe (đèn pha vs đèn hậu, kính lái) để xoay `yaw` đúng chiều; (3) Phân biệt chi tiết ngữ cảnh vật thể (xe buýt vs xe tải hộp, người đi bộ vs cột/tủ điện hoặc xe đạp dựng không người).
- **3 điều chỉ LiDAR cho biết** (từ `s01f16` và `s01f58`): (1) Tọa độ 3D chính xác và khoảng cách ngang theo mét (`dist_m`) tới từng vật; (2) Kích thước hình học thực (`length`, `width`, `height` theo mét) và cao độ đáy `z_bottom` so với mặt đường; (3) Cấu trúc toàn cảnh 360° kể cả vùng góc mù giữa các camera và độ cao chênh lệch mặt đường.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f16` | Cạnh trên (`−y`) | Có dải phân cách sắt bên trái ngăn làn ngược chiều; xuất hiện cụm ~25 điểm lạ nằm sâu dưới mặt đường (`z` tới −12.01 m) |
| `s02f37` | Cạnh trên (`−y`) | Có xe máy điện (Cyclist) đi sát phía trước-phải (~8.64 m) và tủ/cột tối màu trên vỉa hè phải dễ nhầm với Pedestrian |
| `s03f02` | Cạnh trên (`−y`) | Đường một chiều hẹp có tường quây công trình màu xanh bên trái tạo bóng che lớn phía sau tường |
| `s03f19` | Cạnh trên (`−y`) | Ngã tư rộng, nhiều xe hai bánh (Cyclist) và người đi bộ tập trung bên phải qua vạch kẻ đường |
| `s04f07` | Cạnh trên (`−y`) | Hàng rào vàng ở giữa chia hai chiều xe; xe buýt làn ngược chiều bị hàng rào che một phần thân dưới |
| `s04f57` | Cạnh trên (`−y`) | Hàng rào vàng dọc bên trái; tầm nhìn phía trước thoáng, các xe xa (> 35 m) thưa điểm rõ rệt |
| `s05f24` | Cạnh trên (`−y`) | Ngã tư lớn, nhiều phương tiện đa hướng (xe cắt ngang, xe ngược chiều, xe máy bên phải) |
| `s05f45` | Cạnh trên (`−y`) | Xe van chở hàng màu xanh phía trước-phải (dễ nhầm sang Bus nếu không kiểm kích thước), xe buýt xanh làn trái bị xe con che một phần |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f16`, `#3` (`t2-a-car.png`) | Car | Đúng (`262.0°`, hướng về đầu xe cùng chiều `−y`) | Đúng (`z_bottom = −1.778 m`, mặt đường ~−1.66 m) | Đã kiểm Top/Side/Front và `cam01`/`cam03` | Đúng (`l = 4.500 m`, `w = 1.880 m`, `h = 1.427 m` — trong khoảng sedan 3.8–5.0 / 1.70–1.90 / 1.40–1.60 m) |
| (b) xe buýt hoặc xe tải | `s01f16`, `#6` (`t2-b-bus-truck.png`) | Bus | Đúng (`270.0°`, hướng cùng chiều `−y`) | Đúng (`z_bottom = −1.897 m`, độ lệch 0.07 m so với mặt đường cục bộ) | Đã kiểm Top/Side/Front và `cam01`/`cam03` | Đúng (`l = 11.200 m`, `w = 2.550 m`, `h = 3.187 m` — trong khoảng xe buýt thường 10.0–12.5 / 2.50–2.60 / 3.00–3.50 m) |
| (c) người đi bộ hoặc xe hai bánh | `s02f37`, `#1` (`t2-c-ped-cyclist.png`) | Cyclist | Đúng (`286.1°`, hướng di chuyển dọc làn phải) | Đúng (`z_bottom = −1.701 m`, khoảng hở 0.10 m) | Đã kiểm Top/Side/Front và `cam01`/`cam07` | Đúng (`l = 1.950 m`, `w = 0.800 m`, `h = 1.650 m` — trong khoảng xe máy có người 1.8–2.3 / 0.70–0.95 / 1.40–1.80 m) |

- **Hiệu chuẩn & quy ước guideline:** Trong quá trình soát mẫu và đối chiếu guideline, lỗi dễ gặp nhất là căn nhầm hướng `yaw` giữa xe cùng chiều (`~270°`) và xe ngược chiều (`~90°`) ở các làn bên trái (`x > 0`), hoặc gán nhầm xe van cỡ trung thành `Bus` (quá dài). Quy ước chốt theo guideline: luôn đối chiếu ảnh camera trước để xác định đầu xe và loại xe (van chở hàng → `Truck`, xe máy có người → `Cyclist`, xe dựng không người → bỏ qua), sau đó căn vạch đỏ `+X` về đầu xe và kiểm `z_bottom` ở cả Side/Front.
- **Vật bỏ qua vì không đủ cơ sở:**
  - `s01f16` (phía trước-phải ~28 m sau dải cây phân cách): cụm điểm thưa lẫn tán cây (`LOW_VISIBILITY` / `UNCERTAIN_SCOPE`) — bỏ qua.
  - `s02f16` (vỉa hè phải ~11–14 m): xe máy dựng không có người lái — bỏ qua theo guideline.
  - `s02f37` (vỉa hè phải ~9–15 m): cột/tủ kỹ thuật tối màu dễ nhầm `Pedestrian` — bỏ qua sau khi đối chiếu `cam07`/`cam08`.
  - `s01f58` (vỉa hè xa phía trước-phải > 37 m): các cụm người/vật quá nhỏ (< 25 điểm) lẫn cây (`LOW_VISIBILITY`) — bỏ qua.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s01f58` / `t4-1-bong-che.png` / frame của tôi | Giả thuyết: các xe con đi gần phía trước-trái (`#3`, `#4`) chặn trực tiếp tia laser từ LiDAR trên nóc xe | Phía sau thân xe gần tạo vùng trống hình quạt không có điểm; các xe phía sau nó (`#5`, `#6`) chỉ còn một phần sườn/nóc | Bật cờ `occluded = 1` cho vật bị che một phần (`s01f58 #5, #6`), vẫn dựng cuboid đủ kích thước suy luận theo bảng 4.3, không bóp ngắn hộp theo phần điểm nhìn thấy |
| 2. Thưa ở xa | `s01f16` / `t4-2-thua-o-xa.png` / frame của tôi | Giả thuyết: mật độ tia LiDAR 40 tia phân kỳ theo khoảng cách khiến số tia trúng vật giảm nhanh (tỉ lệ nghịch bình phương khoảng cách) | Xe con `#8` ở `47.44 m` chỉ còn 52 điểm rời rạc dạng vài vạch ngang thưa, khó thấy rõ biên hông và góc xe | Vẫn vẽ vì thấy > 15–20% khung vật và xác nhận rõ trên `cam09`; với các vật xa hơn hoặc quá ít điểm không đủ cơ sở thì bỏ qua và ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | `s02f16` / `t4-3-phan-xa-thap.png` / frame của tôi | Giả thuyết: bề mặt sơn đen bóng và kính tối màu của xe sedan đen phía trước (`#5`) hấp thụ hoặc phản xạ gương tia laser đi hướng khác | Dù chỉ cách `12.17 m`, bề mặt kính và một số mảng thân xe đen có mật độ điểm thấp hơn và độ phản xạ (intensity) yếu hơn vùng biển số/viền sáng | Không thu nhỏ cuboid theo vùng thiếu điểm; đối chiếu `cam01`/`cam03` để bao trọn thân xe theo hình học thật và chỉ ghi nguyên nhân dưới dạng giả thuyết |
| 4. Điểm lạ | `s02f16` / `t4-4-diem-la.png` / frame của tôi | Giả thuyết: phản xạ đa đường (multipath) từ mặt đường/hàng rào kim loại hoặc nhiễu cảm biến làm thời gian bay tia laser dài bất thường | Xuất hiện cụm 25 điểm nằm sâu dưới mặt đường tại vùng `(13.5 m, 9.1 m)`, có điểm xuống tới `z = −12.01 m` (thấp hơn mặt đường ~10.5 m) | Tuyệt đối không gán nhãn thành vật thể; loại bỏ hoàn toàn khỏi phạm vi suy luận mặt đường và chiều cao cuboid |

## 5. Số từ cuboid của tôi (Bước 5)
Số liệu lấy trực tiếp từ `submission/D13-2A202603155-bangA.csv` và `submission/D13-2A202603155-tukiem.csv`.

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r = √(x²+y²+z²) (m) | yaw_deg (°) | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f16`, `#3`, `Car` | −1.064 | 1.427 | −1.778 | 10.06 | 10.11 | 262.0 | Chạm đường (lệch 0.08 m so với −1.70 m, `ground_gap = 0.04 m`) |
| `s01f58`, `#4`, `Car` | −0.961 | 1.600 | −1.761 | 10.24 | 10.28 | 270.4 | Chạm đường (lệch 0.06 m so với −1.70 m, `ground_gap = 0.04 m`) |
| `s01f16`, `#6`, `Bus` | −0.304 | 3.187 | −1.897 | 29.04 | 29.04 | 270.0 | Chạm đường (lệch 0.20 m ≤ 0.30 m do dốc nhẹ ở 29 m, `ground_gap = 0.07 m`) |

- **(a) Cuboid gần nhất và xa nhất:**
  - Gần nhất: `s01f58 #1` (`Car`, `x = 3.022 m`, `y = 1.055 m`, `z = −1.010 m`) có `dist_m = 3.20 m` và khoảng cách 3D `r = 3.36 m`.
  - Xa nhất: `s04f57 #9` (`Car`, `x = −3.395 m`, `y = −47.578 m`, `z = −1.215 m`) có `dist_m = 47.70 m` và khoảng cách 3D `r = 47.71 m`.
- **(b) So sánh kích thước với bảng 4.3:**
  - Xe hai bánh có người (`s02f37 #1`, `Cyclist`): `l = 1.950 m`, `w = 0.800 m`, `h = 1.650 m` — nằm hoàn toàn trong khoảng xe máy có người của bảng 4.3 (`l`: 1.8–2.3 m, `w`: 0.70–0.95 m, `h`: 1.40–1.80 m), không có chiều nào ngoài khoảng.
  - Xe con (`s01f16 #3`, `Car`): `l = 4.500 m`, `w = 1.880 m`, `h = 1.427 m` — nằm hoàn toàn trong khoảng sedan/hatchback của bảng 4.3 (`l`: 3.8–5.0 m, `w`: 1.70–1.90 m, `h`: 1.40–1.60 m), không có chiều nào ngoài khoảng.
- **(c) Góc hướng `yaw_deg`:**
  - Xe cùng chiều phía trước: `s01f58 #4` (`Car`) có `yaw_deg = 270.4°` — rất sát `270°` (hướng `−y`).
  - Xe đi ngược chiều: `s02f16 #3` (`Car`) có `yaw_deg = 90.5°` — rất sát `90°` (hướng `+y`).

Mật độ theo khoảng cách (4 cuboid `Car` trên cùng frame `s01f16` ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (`points_in`) |
|---|---|---|
| `s01f16`, `#1`, `Car` | 5.85 | 2305 |
| `s01f16`, `#3`, `Car` | 10.06 | 1127 |
| `s01f16`, `#5`, `Car` | 20.14 | 294 |
| `s01f16`, `#8`, `Car` | 47.44 | 52 |

- **Quy luật mật độ:** Khi khoảng cách tăng gấp 2 lần (từ 10.06 m lên 20.14 m), số điểm trên xe con giảm còn khoảng 1/4 (từ 1127 xuống 294 điểm, ~26%), và xa gấp ~4.7 lần (47.44 m) chỉ còn 52 điểm (~4.6%) — gần quy luật tỉ lệ nghịch bình phương khoảng cách và bám sát đường cong trung vị trên slide bài giảng (0–10 m: ~1968 điểm, 10–20 m: ~718, 20–30 m: ~214, 30–40 m: ~72, 60–80 m: ~19).
- **Tự kiểm và chỉnh sửa sau `preview` / `selfcheck`:**
  - Đã sửa nhiều cuboid sau khi xem `work/xem/*.jpg` và `tukiem.csv`, tiêu biểu: (1) `s05f45 #2` (xe van xanh phía trước-phải) ban đầu bị gán nhầm `Bus` dài 11.2 m và lệch hướng 292.9° → sửa thành `Truck` (`l = 5.600 m`, `w = 2.050 m`, `h = 2.450 m`, `yaw_deg = 270.0°`); (2) `s03f02 #1` và `#2` ban đầu bị chồng hộp lên cùng một xe trắng và có hộp chìm 0.31 m → xóa hộp trùng, giữ một hộp sát thân xe và sửa `yaw` đúng chiều `266.5°`; (3) sửa ngược hướng 180° cho các xe cùng chiều làn trái ở `s01f16` (`#1`, `#3`, `#6`) và xe ngược chiều làn trái ở `s02f16` (`#1`–`#4`).
  - Các cảnh báo giữ nguyên (5/95 cuboid): `s01f58 #7` (Pedestrian), `s02f37 #4` (Car), `s03f19 #6` (Cyclist), `s04f57 #5` (Car) bị cờ điểm sát mép do có cây/người/xe đi sát bên cạnh chứ hộp không quá chật; `s01f16 #7` (Car ở 41.08 m) bị cờ đáy thấp hơn mặt đường 0.43 m do mặt đường ở tầm xa dốc cục bộ và ít điểm mặt đất quanh hộp.
- **Đổi kiểm:** Đã đối chiếu trực tiếp trên màn hình hai con số từ bảng A (`s01f16 #1`: `dist_m = 5.85 m`, `z_bottom = −1.689 m`; `s01f58 #4`: `yaw_deg = 270.4°`, `dist_m = 10.24 m`) — kết quả khớp hoàn toàn với file `submission/D13-2A202603155-bangA.csv`. *(Ghi chú: bổ sung MSSV bạn ngồi cạnh trước khi push chính thức nếu cần.)*

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa đối với model 3D:** Nhãn 3D đo bằng **mét** thay vì pixel vì hệ thống tự hành cần dự đoán khoảng cách thực, kích thước vật lý và quỹ đạo va chạm trong không gian 3D (một xe ở 10 m và 40 m có kích thước pixel rất khác nhau nhưng kích thước mét là bất biến). Vật thưa điểm cần quy tắc riêng (ngưỡng 15–20% thể tích và nguyên tắc không đoán) để tránh đưa nhãn suy đoán sai tâm/hướng vào tập huấn luyện, vốn sẽ làm model học nhiễu và suy giảm độ chính xác định vị.
- **(b) Cách dùng AI:**
  - Công cụ: Trợ lý lập trình AI (Antigravity).
  - Câu hỏi chính: Hỗ trợ cấu hình môi trường `.venv`, chạy script `lab.py` (`dry`, `task`, `sync`, `pack`), rà soát chiếu cuboid lên 7 camera và kiểm tra nhất quán số liệu báo cáo với Bảng A/phiếu tự kiểm.
  - Cách dùng kết quả: Mọi cuboid đều được kiểm tra trực quan lại qua `work/xem/*.jpg` và `tukiem.csv`, chỉnh sửa trực tiếp các ca ngược hướng/nhầm class/trùng hộp, và đối chiếu toàn bộ con số trong báo cáo với file CSV xuất từ CVAT.
- **(c) Điều còn chưa chắc:** Một số xe ở xa ngoài 35 m (như `s01f16 #7` ở 41.08 m hoặc `s05f45 #5`–`#9` ở làn cắt ngang) có độ dốc mặt đường cục bộ và mật độ điểm thưa nên khó khẳng định tuyệt đối độ chính xác của chiều dài toàn thân xe bị khuất và cao độ đáy `z_bottom`.
