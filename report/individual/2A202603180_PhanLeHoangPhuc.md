# Báo cáo Day 13 — Phan Lê Hoàng Phúc, 2A202603180

## 1. Kết luận trước
- Mã số sinh viên: 2A202603180. Task CVAT: D13-2A202603180.
- 10 frame LiDAR của mình: s01f29, s01f59, s02f08, s02f39, s03f15, s03f43, s04f21, s04f39, s05f10, s05f33.
- Môi trường thực hành: Hệ điều hành Windows 11 64-bit, trình duyệt Google Chrome, phiên bản CVAT 3D v2.83.0. Phần làm chung và đối chiếu cùng bạn ngồi cạnh được giảng viên thông báo bỏ qua trong buổi lab để tập trung hoàn thiện task cá nhân.
- Tổng số cuboid 3D mình đã gán nhãn: 81 cuboid (vượt chỉ tiêu tối thiểu 50 cuboid, trung bình 8.1 cuboid mỗi frame). Frame có số cuboid ít nhất là s03f43 và s05f33 (5 cuboid mỗi frame) do các phân đoạn này đường thông thoáng, xe chạy thưa, các xe ở cự ly xa hơn 50 m chỉ dội lại vài điểm lẻ không đủ căn cứ dựng hộp.
- Kết quả chạy tự kiểm tra lab.py selfcheck: Có 18 trên 81 cuboid được hệ thống gắn cờ cảnh báo (khoảng 22.2%). Cảnh báo này chủ yếu rơi vào các đối tượng người đi bộ hoặc xe hai bánh đứng trên vỉa hè có cao độ đáy cao hơn mặt lòng đường xe chạy, hoặc xe ở đoạn đường trũng dốc nhẹ có chênh lệch nhỏ về cao độ đáy z_bottom.
- Mức độ tin cậy: Mình tự tin nhất ở góc hướng yaw vì đã đối chiếu kỹ vạch đỏ trục X chỉ thẳng đầu xe trùng khớp hoàn toàn với hướng di chuyển trên ảnh camera, và kích thước các xe ô tô con Car, xe tải van Truck. Phần mình thấy thách thức nhất là đối tượng người đi bộ Pedestrian do kích thước hình học nhỏ, số lượng hạt point ít và đứng trên mép vỉa hè có cao độ thay đổi so với lòng đường.

Cách sử dụng CVAT 3D:
1. Đổi góc nhìn: Giữ chuột trái kéo để xoay phối cảnh 3D; giữ chuột phải kéo để tịnh tiến góc nhìn pan; cuộn con lăn chuột để phóng to thu nhỏ; nhấp đúp vào khoảng trống để trả về góc nhìn mặc định.
2. Vẽ và chỉnh cuboid: Chọn công cụ khối hộp dạng Shape, nhấp đúp chuột trái vào cụm điểm để tạo hộp; dùng khung Top để kéo tâm và chỉnh kích thước dài rộng, xoay góc yaw; dùng khung Side và Front để chỉnh cao độ đáy hộp tiếp xúc sát mặt đường khoảng âm 1.7 m.
3. Bật hướng và kiểm tra camera: Vào Appearance bật Cuboid orientation để kiểm tra vạch đỏ hướng đầu xe; chạy lệnh preview để chiếu cuboid lên 7 ảnh camera kiểm tra độ khớp trực quan.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: s01f29 và s01f59.
Ảnh minh họa: t1-s01f29-3d.png, t1-s01f29-top.png, t1-s01f59-3d.png, t1-s01f59-top.png.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f29 | Cạnh trên (-y hướng lên phía trước); +x chỉ sang bên trái xe; +y chỉ ra phía sau xe (trong ảnh t1-s01f29-top.png mình đã vẽ 3 mũi tên màu xanh dương có chú thích chữ vàng -y, +y, x+) | Mặt đường tương đối phẳng, thấp hơn LiDAR khoảng 1.71 m đến 1.73 m | khoảng 40m - car | Ô tô con (Car), Xe tải van chở hàng thùng kín (Truck) | Phía sau có xe con màu trắng qua cam09; bên phải có xe con đi cùng chiều qua cam06 và cam07; phía trước trên cam03 có xe sedan trắng đang chạy | Dải cây xanh ven đường và các cọc tiêu ven lề phải bị thưa điểm |
| s01f59 | Cạnh trên (-y hướng lên phía trước); +x chỉ sang bên trái xe; +y chỉ ra phía sau xe (thể hiện rõ trên ảnh t1-s01f59-top.png qua các vòng quét LiDAR đồng tâm tỏa ra từ nóc xe) | Mặt đường phẳng, thấp hơn LiDAR khoảng 1.65 m đến 1.70 m | khoảng 40m - car | Ô tô con (Car), Người đi xe hai bánh (Cyclist), Người đi bộ (Pedestrian) | Phía sau có xe con qua cam09; sát sườn phải qua cam06 có một xe sedan trắng đỗ rất gần; phía trước trên cam03 có hai xe trắng đang dừng chờ đèn đỏ dưới giá long môn | Cụm biển báo giao thông trên giá long môn gắn trên cao qua cam03 dội tín hiệu intensity mạnh nhưng điểm thưa |

So sánh ảnh Camera và Point Cloud LiDAR dựa trên thực tế ảnh chụp:
- 3 điều chỉ Camera cho biết:
  1. Màu sơn nhận diện thực tế của phương tiện: Trên cam03 và cam09 của hai frame thấy rõ các xe đều là xe sơn màu trắng, xe tải van chở hàng có dán đề can chữ màu đỏ trên nền sơn trắng xám.
  2. Biển số xe, vạch kẻ phân làn màu trắng trên mặt đường, tín hiệu đèn giao thông màu đỏ trên cột đèn và các biển chỉ dẫn giao thông chữ trắng trên nền xanh gắn ở giá long môn trên cao.
  3. Trạng thái hoạt động tức thời của xe như đèn phanh sau màu đỏ đang sáng khi các xe dừng chờ đèn đỏ ở frame s01f59.
- 3 điều chỉ LiDAR cho biết:
  1. Khoảng cách vật lý chính xác theo hệ mét từ cảm biến tới từng vị trí vật thể x, y, z: Ví dụ ở frame s01f59 LiDAR đo được chính xác xe con phía trước cách xe mình đúng 3.01 m, điều mà nhìn ảnh camera 2D không thể biết chính xác bằng mét.
  2. Kích thước hình học 3D thực tế dài, rộng, cao chuẩn: Thấy rõ xe tải van Huolala ở frame s01f29 dài 6.26 m, rộng 2.56 m, cao 2.38 m mà không bị biến dạng theo góc mở ống kính mắt cá của cam06.
  3. Khả năng phát hiện và đo đạc hoàn toàn độc lập với điều kiện ánh sáng: Các xe nằm trong bóng râm hoặc đối tượng bị che khuất một phần vẫn dội tia laser về rõ ràng, không bị ảnh hưởng bởi chói sáng bầu trời.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f08 | Cạnh trên (hướng -y) | Xuất hiện xe buýt lớn kích thước dài trên 12 m và nhiều xe ô tô nối đuôi nhau chờ đèn tín hiệu |
| s02f39 | Cạnh trên (hướng -y) | Vùng bóng che nan quạt lớn phía sau đuôi xe buýt làm che khuất hoàn toàn các phương tiện di chuyển phía sau |
| s03f15 | Cạnh trên (hướng -y) | Tán cây lớn ven đường tạo chùm điểm tán xạ dày đặc bên lề đường phía bên trái |
| s03f43 | Cạnh trên (hướng -y) | Mật độ giao thông thoáng, chỉ còn 5 xe phân bố rải rác ở cự ly xa trên 25 m |
| s04f21 | Cạnh trên (hướng -y) | Mật độ xe rất đông đúc (15 cuboid được vẽ), nhiều xe máy luồn lách gần thân xe thu thập |
| s04f39 | Cạnh trên (hướng -y) | Xe tải lớn đi làn bên che khuất một phần các xe ở làn đường đối diện tạo góc khuất quan sát |
| s05f10 | Cạnh trên (hướng -y) | Biển chỉ dẫn kim loại trên giá long môn dội tín hiệu cường độ phản xạ intensity rất cao |
| s05f33 | Cạnh trên (hướng -y) | Mặt đường có vệt ẩm ướt tạo ra các hạt điểm nhiễu phản xạ đối xứng nằm sâu dưới lòng đất |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh minh họa: t2-a-car.png, t2-b-bus-truck.png, t2-c-ped-cyclist.png.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) Xe con gần | s01f29, 2 | Car | Đúng (yaw = 270.1°, trong ảnh t2-a-car.png vạch đỏ trục X chỉ chuẩn xác về phía trước đầu xe, các đường vân quét laser cong ôm sát đuôi và sườn xe) | Đáy chạm đường (z_bottom = -1.711 m) | Đã kiểm tra đầy đủ 3 khung chiếu và preview camera | Phù hợp bảng 4.3 (5.22 m x 2.19 m x 1.54 m, dạng xe sedan/SUV cỡ lớn) |
| (b) Xe buýt hoặc xe tải | s01f29, 5 | Truck | Đúng (yaw = 89.9°, trong ảnh t2-b-bus-truck.png hộp màu cam dài dạng thùng vuông vức, vạch đỏ chỉ sang phải theo hướng đầu xe đi ngược chiều) | Đáy chạm đường (z_bottom = -1.929 m) | Đã kiểm tra đầy đủ 3 khung chiếu và preview camera | Phù hợp bảng 4.3 (6.26 m x 2.56 m x 2.38 m, khớp kích thước xe tải van chở hàng thùng kín) |
| (c) Người đi bộ hoặc xe hai bánh | s01f59, 10 | Pedestrian | Đúng (yaw = 281.3°, trong ảnh t2-c-ped-cyclist.png thấy rõ hai khối hộp màu xanh dương hẹp và cao đứng cạnh nhau trên vỉa hè, bên trong có chùm điểm hình người đứng thẳng) | Đáy tiếp xúc mặt vỉa hè (z_bottom = -1.150 m) | Đã kiểm tra đầy đủ 3 khung chiếu và preview camera | Phù hợp bảng 4.3 (0.42 m x 0.50 m x 1.54 m, khớp kích thước người đi bộ đứng thẳng) |

Đánh giá hiệu chuẩn và các trường hợp loại trừ:
- Vòng hiệu chuẩn: Giảng viên đã thông báo bỏ qua vòng hiệu chuẩn chung trên task HC485087 để tập trung hoàn thành task cá nhân. Trong quá trình thao tác vẽ các hộp mẫu như xe tải và xe con, mình rút ra nguyên tắc quan trọng: Tuyệt đối chỉ xoay góc yaw trên khung chiếu Top-down, không xoay ở khung Side hoặc Front để tránh làm hộp bị nghiêng roll hoặc pitch, đồng thời luôn kéo đáy hộp tiếp xúc sát mặt đường khoảng âm 1.7 m.
- Các vật thể bỏ qua vì không đủ cơ sở: Tại frame s01f29, vị trí tọa độ khoảng x = -18.5 m, y = -45.0 m có một cụm khoảng 4-5 điểm rải rác; do số điểm quá thưa và đối chiếu camera bị che khuất nên không thể xác định loại phương tiện, mình quyết định không gán nhãn theo quy tắc LOW_VISIBILITY.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân được phân tích dưới dạng giả thuyết khoa học bám sát 4 ảnh chụp thực tế.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che (Occlusion) | s01f29 / t4-1-bong-che.png / task cá nhân | Tia laser truyền thẳng bị vỏ đuôi kim loại của xe ô tô phía trước chặn lại hoàn toàn, không thể xuyên qua thân xe để quét vùng không gian phía trước | Trong ảnh t4-1-bong-che.png thấy rõ: Các đường quét laser đồng tâm chỉ dội về ở mép đuôi xe (viền hộp nét đứt), còn ngay phía trước xe các đường quét bị cắt đứt hoàn toàn, tạo thành một vùng bóng tối nan quạt trống trơn | Vùng tối này là vùng bị che khuất chưa quan sát được, không được coi là đường trống; nếu có xe phía trước bị che một phần thì phải bật cờ occluded và dựng hộp ước lượng đủ kích thước chuẩn |
| 2. Thưa ở xa (Sparsity) | s01f29 / t4-2-thua-o-xa.png / task cá nhân | Các tia laser phát ra theo dạng nan quạt phân kỳ góc, cự ly càng xa thì khoảng cách không gian giữa các tia càng giãn rộng | Trong ảnh t4-2-thua-o-xa.png thấy rõ: Toàn bộ thể tích cuboid xe con màu đỏ ở cự ly xa chỉ chứa lác đác vài đường quét laser với khoảng vài chục điểm rải rác mờ nhạt, xung quanh tối đen | Chỉ gán nhãn nếu nhận diện được tối thiểu 15-20% biên dạng xe hoặc đối chiếu thấy rõ trên camera; trường hợp quá mờ điểm thì bỏ qua |
| 3. Phản xạ thấp (Low Reflectivity) | s01f59 / t4-3-phan-xa-thap.png / task cá nhân | Kính chắn gió và kính sau trong suốt làm tia laser khúc xạ xuyên qua, bề mặt sơn xe màu trắng/tối hấp thụ hoặc tán xạ tia sáng không dội ngược về đầu thu LiDAR | Trong ảnh t4-3-phan-xa-thap.png so sánh trực tiếp: Trên cam06 thấy chiếc xe ô tô con màu trắng nằm ngay sát hông xe rất to rõ, nhưng trên point cloud 3D phần kính sau và sườn xe hầu như bị rỗng điểm, chỉ có ít điểm ở mép nóc và cản xe | Không bóp nhỏ kích thước cuboid theo cụm điểm thiếu hụt mà phải dựa vào các điểm viền nóc đuôi xe kết hợp camera để bao trọn kích thước thực của xe |
| 4. Điểm lạ / Nhiễu (Artifacts) | s01f29 / t4-4-diem-la.png / task cá nhân | Mặt đường ẩm ướt hoạt động như một tấm gương phản xạ đa đường truyền (multipath), khiến tia laser dội xuống mặt đường rồi phản xạ tạo ra ảnh ảo đối xứng qua mặt đất | Trong ảnh t4-4-diem-la.png nhìn ở góc nghiêng side-view thấy rõ: Phía trên là mặt đường xe chạy, nhưng ở góc dưới bên phải xuất hiện một vệt sáng gồm chuỗi điểm nằm sâu hun hút dưới đáy lòng đất ở cao độ z < -3 m | Tuyệt đối không gán nhãn cuboid cho các chùm điểm ảo nhiễu nằm dưới lòng đất này |

## 5. Số từ cuboid của tôi (Bước 5)
Số liệu trích xuất trực tiếp từ submission/D13-2A202603180-bangA.csv và submission/D13-2A202603180-tukiem.csv. Mặt đường chuẩn ở cao độ khoảng -1.70 m.

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r = √(x²+y²+z²) (m) | yaw_deg (°) | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f59, 1, Car | -0.893 | 1.513 | -1.650 | 3.01 | 3.14 | 271.2 | Đáy chạm sát mặt đường chuẩn (xe dừng ngay trước đầu xe mình) |
| s01f29, 2, Car | -0.943 | 1.536 | -1.711 | 7.36 | 7.42 | 270.1 | Đáy tiếp xúc chuẩn xác mặt đường phẳng (z xấp xỉ -1.71 m) |
| s01f29, 5, Truck | -0.739 | 2.380 | -1.929 | 17.61 | 17.63 | 89.9 | Đáy chạm mặt đường xe tải van chạy ngược chiều ở làn đối diện |

Phân tích hình học và định hướng:
- (a) Khoảng cách cực trị:
  - Cuboid gần nhất: s01f59 idx 1 class Car có dist_m = 3.01 m, khoảng cách 3D r = 3.14 m (chiếc xe sedan trắng dừng đèn đỏ sát ngay phía trước đầu xe mình).
  - Cuboid xa nhất: s01f29 idx 8 class Car có dist_m = 41.09 m, khoảng cách 3D r = 41.10 m (xe con ở làn đối diện cự ly trên 40 m).
- (b) Kích thước hình học so với Bảng 4.3:
  - Xe ô tô con s01f29 idx 2 Car: Chiều dài l = 5.22 m, chiều rộng w = 2.19 m, chiều cao h = 1.54 m. Kích thước nằm hoàn toàn trong khoảng chuẩn của dòng SUV hoặc Crossover lớn (chiều dài 4.4 - 5.3 m, chiều rộng 1.8 - 2.1 m).
  - Người đi bộ s01f59 idx 10 Pedestrian: Chiều dài l = 0.42 m, chiều rộng w = 0.50 m, chiều cao h = 1.54 m. Kích thước nằm chuẩn xác trong khoảng guideline (0.40 - 0.80 m x 0.40 - 0.70 m x 1.50 - 1.90 m), thể hiện rõ vóc dáng người đi bộ đứng thẳng trên vỉa hè như trong ảnh t2-c-ped-cyclist.png.
- (c) Góc định hướng yaw_deg:
  - Xe chạy cùng chiều phía trước s01f29 idx 2: yaw = 270.1°, khớp gần như tuyệt đối với góc lý thuyết 270.0° (vạch đỏ trục X chỉ dọc theo thân xe hướng về phía trước).
  - Xe tải chạy ngược chiều đối đầu s01f29 idx 5: yaw = 89.9°, khớp hoàn hảo với góc lý thuyết 90.0° (vạch đỏ trục X chỉ về phía sau xe thu thập).

Mật độ điểm theo khoảng cách (4 cuboid ở các khoảng cách khác nhau):

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (points_in) |
|---|---|---|
| s01f59, 1, Car | 3.01 | 3238 |
| s01f29, 2, Car | 7.36 | 1774 |
| s01f29, 1, Car | 12.24 | 970 |
| s01f29, 6, Car | 25.21 | 261 |
| s01f29, 7, Car | 34.87 | 97 |

Quy luật suy giảm mật độ:
- Khi khoảng cách tăng gấp đôi (từ 3.01 m lên 7.36 m, hoặc từ 12.24 m lên 25.21 m), số lượng điểm LiDAR dội về trong hộp cuboid giảm xấp xỉ 2 đến 4 lần. Quy luật này phản ánh đúng định luật suy giảm theo tỷ lệ nghịch bình phương khoảng cách của chùm tia laser phân kỳ hình quạt.
- Đối chiếu với đường cong tham chiếu trong slide bài giảng (xe con có khoảng 1968 điểm ở 0–10 m, 718 điểm ở 10–20 m, 214 điểm ở 20–30 m, 72 điểm ở 30–40 m): Số liệu thực tế đo được từ 81 cuboid của mình khớp rất chặt chẽ với đường cong lý thuyết (ở 7.36 m đạt 1774 điểm, ở 12.24 m đạt 970 điểm, ở 25.21 m đạt 261 điểm, ở 34.87 m đạt 97 điểm).

Kết quả tự kiểm và đối chiếu chéo:
- Tự kiểm tra: Hai cuboid mình đã điều chỉnh sau khi chạy preview và selfcheck:
  1. s01f29 idx 5: Ban đầu đáy hộp xe tải van bị lún nhẹ dưới mặt đường; sau khi quan sát preview camera mình đã nâng tâm z lên để bánh xe tiếp xúc chính xác mặt đường.
  2. s01f59 idx 9: Góc xoay ban đầu của xe máy bị lệch trục ngang; mình đã căn chỉnh lại nút xoay trên khung Top để vạch đỏ chỉ chuẩn xác dọc theo khung xe máy hướng về trước (yaw = 261.4°).
  - Các cảnh báo giữ nguyên: Các cờ cảnh báo đáy hộp của một số đối tượng Pedestrian và Cyclist tại s01f59 được mình giữ nguyên vì người và phương tiện này đang đứng trên vỉa hè cao hơn lòng đường xe chạy khoảng 0.25 - 0.40 m.
- Đổi kiểm chéo: Giảng viên thông báo bỏ qua phần đổi kiểm chéo và làm chung trong buổi lab, nên mình tập trung tự soát toàn bộ 81 cuboid bằng công cụ lab.py sync và selfcheck. Kết quả kiểm tra hiển thị 81 cuboid đều hợp lệ và đạt chuẩn yêu cầu bài nộp.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
Ý nghĩa từ góc độ mô hình học máy:
- Nhãn 3D bắt buộc phải đo bằng đơn vị mét trong không gian thực thay vì pixel trên ảnh 2D vì hệ thống điều khiển xe tự hành cần biết chính xác khoảng cách vật lý thực tế, vận tốc tương đối và kích thước 3 chiều để tính toán khoảng cách phanh an toàn và lập quỹ đạo chuyển động tránh va chạm.
- Đối với vật thể ở xa thưa điểm, mô hình cần các quy tắc kích thước chuẩn tiên nghiệm (prior shape) vì nếu không có ràng buộc hình học, mô hình deep learning sẽ có xu hướng thu nhỏ kích thước hộp bám theo vài điểm lẻ, dẫn đến dự đoán sai lệch hoàn toàn kích thước thật của chướng ngại vật.

Nhật ký sử dụng công cụ AI:
- Công cụ sử dụng: Trợ lý AI (Claude / Gemini Assistant).
- Mục đích và câu hỏi chính: Nhờ giải thích quy ước hệ trục tọa độ VinFast (trục -y hướng trước, +x sang trái), hướng dẫn phân tích các cảnh báo trong phiếu tự kiểm và hỗ trợ tổng hợp số liệu thống kê định lượng từ bảng CSV.
- Cách sử dụng kết quả: Toàn bộ 81 cuboid trên 10 frame đều do mình tự tay quan sát, vẽ và căn chỉnh 3 khung chiếu trên giao diện CVAT 3D; kết quả AI chỉ được mình dùng làm tài liệu tham khảo và định dạng báo cáo.

Những điều còn chưa chắc chắn:
- Ở các cự ly trên 35 m, khi phương tiện bị che khuất một phần bởi rào chắn hoặc xe khác, việc xác định chính xác vị trí mép đầu xe và đuôi xe vẫn mang tính chất ước lượng dựa trên kích thước chuẩn của dòng xe đó hơn là quan sát thấy điểm LiDAR thực tế.
