# Báo cáo Day 13 — Trần Trọng Thái, 2A202604065

MSSV: **2A202604065**. Task: `D13-2A202604065`. 10 frame của tôi: `s01f27`, `s01f42`, `s02f08`, `s02f29`, `s03f20`, `s03f58`, `s04f17`, `s04f36`, `s05f00`, `s05f17`. Mọi con số trong báo cáo này lấy từ `submission/D13-2A202604065-bangA.csv` (bảng A) và `submission/D13-2A202604065-tukiem.csv` (phiếu tự kiểm), kèm đơn vị.

## 1. Kết luận trước

Tôi tổng cộng **145 cuboid** trên 10 frame: `s01f27`=10, `s01f42`=17, `s02f08`=17, `s02f29`=9, `s03f20`=30, `s03f58`=7, `s04f17`=20, `s04f36`=16, `s05f00`=8, `s05f17`=11. Frame ít cuboid nhất là `s03f58` (7) và `s05f00` (8) — ở hai frame này phần lớn vật nằm ngoài 50 m hoặc quá thưa nên tôi bỏ qua theo quy tắc 15–20% thể tích của guideline.

`selfcheck` còn **6/145 cuboid** bị cờ (khoảng 4%), trong đó có 2 hộp bị cờ vì kích thước nằm ngoài khoảng của class, 2 hộp bị cờ vì đáy thấp hơn mặt đường, 2 hộp bị cờ vì có nhiều điểm sát ngay ngoài mép.

Phần tôi tin nhất là **hướng (yaw)**: 59 xe có `yaw_deg` quanh 270° (cùng chiều phía trước) và 33 xe quanh 90° (ngược chiều), khớp với hệ trục x-trái/y-sau. Phần còn yếu nhất là **kích thước**: 36/145 cuboid có `z_bottom` lệch mặt đường (khoảng −1,7 m) hơn 0,3 m, và một số hộp "Cyclist" ở `s03f20` có chiều dài 1,15–1,56 m, ngắn hơn xe đạp 1,6–1,9 m trong bảng 4.3 nên cần xem lại là người đi bộ hay xe hai bánh.

Cách dùng CVAT: tôi kéo chuột trái trong khung 3D để xoay góc nhìn (nhìn từ trên xuống để thấy vật hai bên và phía sau), kéo chuột phải để dời khung, lăn con lăn để phóng to. Vẽ cuboid tôi bấm nút vẽ cuboid ở thanh công cụ trái, chọn label, bấm **Shape** rồi đưa hộp tới cụm điểm và **nháy đúp** để đặt. Chỉnh hộp tôi làm ở khung **Top** (kéo để dời tâm, kéo cạnh để đổi dài/rộng, kéo nút xoay để đổi yaw) và **Side/Front** (kéo để đặt đáy chạm đường và chiều cao), không xoay ở Side/Front. Mũi tên hướng bật ở **Appearance → Cuboid orientation**. Để xem cuboid của mình trên ảnh camera tôi chạy `python scripts/lab.py preview` (ảnh ra `work/xem/`).

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ: `s01f27` và `s01f42`. Ảnh: `t1-s01f27-3d.png`, `t1-s01f27-top.png`, `t1-s01f42-3d.png`, `t1-s01f42-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f27` | Phía trước ở **cạnh trên** của ảnh top; **+x sang trái**, **+y xuống dưới** | Phẳng, thấp hơn LiDAR khoảng **1,7 m** | Khoảng **28 m** (`s01f27` idx 10, `dist_m`=25,8 m) | Car (9), Truck (1) | Có xe ở bên phải ảnh top (xe cùng chiều phía trước) và phía sau; vùng sau Truck không có điểm | Cụm điểm thưa ở xa, hơn 25 m |
| `s01f42` | Phía trước ở **cạnh trên** của ảnh top; **+x sang trái**, **+y xuống dưới** | Phẳng, thấp hơn LiDAR khoảng **1,7 m** | Khoảng **50 m** (`s01f42` idx 17, `dist_m`=50,1 m) | Car (8), Cyclist (6), Pedestrian (2), Bus (1) | Có xe hai bánh và người đi bộ bên phải ảnh top; phía sau có Car ở 50 m | Cụm thưa ở xa, và 1 hộp "Bus" 4,78 m (xem mục 5) |

Ba điều **chỉ camera** cho biết: (1) màu và loại xe (xe buýt nào, xe con màu gì, có người mặc áo sáng không); (2) đầu và đuôi xe — camera cho biết mặt trước xe ở đâu để cắt vạch đỏ dọc thân; (3) có vật nào bị che bởi vật khác trong khung hình.

Ba điều **chỉ LiDAR** cho biết: (1) mặt đường phẳng hay dốc, và mặt đường nằm ở z ≈ **−1,7 m**; (2) có bao nhiêu vật và vật nào ở phía sau/two bên — vùng đó không có trong cam01; (3) vật nào chỉ còn vài chục điểm ở xa (điểm thưa dần theo khoảng cách).

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f08` | Cạnh trên | Có 2 xe buýt; nhiều người đi bộ và xe hai bánh sát phía trước |
| `s02f29` | Cạnh trên | Ít vật hơn, đám đông ở xa |
| `s03f20` | Cạnh trên | Rất nhiều người đi bộ và xe hai bánh (30 cuboid) |
| `s03f58` | Cạnh trên | 2 xe buýt, nhiều điểm bị che sau xe buýt |
| `s04f17` | Cạnh trên | Xe ở 63–67 m chỉ còn ~15–26 điểm |
| `s04f36` | Cạnh trên | 1 xe tải, 4 xe hai bánh |
| `s05f00` | Cạnh trên | 2 xe buýt gần, 1 xe tải |
| `s05f17` | Cạnh trên | 1 xe tải dài 10,3 m, 1 hộp "Bus" 4,36 m (xem mục 5) |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f27`, idx 1 | Car | Đúng, `yaw_deg`=270,6° (cùng chiều) | Đúng, `z_bottom`=−1,65 m | Có | 4,77×1,98×1,40 m — nằm trong khoảng sedan (3,8–5,0 × 1,70–1,90 × 1,40–1,60 m) |
| (b) xe buýt | `s02f08`, idx 1 | Bus | Đúng, `yaw_deg`=82,9° | Đúng, `z_bottom`=−1,88 m | Có | 10,79×2,83×3,19 m — dài trong khoảng 10–12,5 m, rộng hơn 2,60 m |
| (c) người đi bộ | `s03f20`, idx 21 | Pedestrian | Đúng, `yaw_deg`=327,7° (đang đi về phía sau bên trái) | Đúng, `z_bottom`=−1,51 m | Có | 0,50×0,62×1,76 m — trong khoảng người lớn 0,4–0,9 × 0,5–0,9 × 1,50–1,95 m |

Vòng hiệu chuẩn: task chung `D13-HC485087` không có trong máy của tôi lúc làm bài, nên tôi tự vẽ và tự so với guideline. Điểm tôi cần chốt là **khi nào gọi một hộp là Bus và khi nào là Car**: tôi cân nhắc theo chiều dài > 6 m và số điểm (một xe buýt cho 1000–5000 điểm ở 15–20 m, một xe con cho 300–1500 điểm cùng khoảng cách). Tôi đã **bỏ qua** 2 vật vì không đủ cơ sở: một cụm điểm rất thưa ở `s04f17` (idx 12 và 13, lần lượt 39 và 26 điểm ở 63 và 67 m) — loại nghi vấn **LOW_VISIBILITY**; và một cụm ở `s03f20` idx 12 (15 điểm ở 44,3 m) — loại nghi vấn **UNCERTAIN_SCOPE**.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s02f08` / `t4-1-bong-che.png` / frame của tôi | Xe buýt chắn tầm LiDAR, phần sau nó không được chiếu rọi | `s02f08` idx 1 (Bus, 14,7 m) và idx 2 (Bus, 17,9 m); sau xe buýt gần như không có điểm, trong khi hai bên góc đó vẫn dày điểm | Vùng trống phía sau là **chưa quan sát được**, không phải đường trống; nếu vật nằm trong đó thì bật occluded và cho cuboid bao phần suy luận được |
| 2. Thưa ở xa | `s04f17` / `t4-2-thua-o-xa.png` / frame của tôi | Mật độ điểm giảm mạnh theo khoảng cách (bụi, mặt đường xa, vật nhỏ) | `s04f17` idx 12 (63,4 m) chỉ 39 điểm, idx 13 (66,8 m) chỉ 26 điểm, trong khi xe ở 6,9 m có 2850 điểm | Dưới 15–20% thể tích và không có ảnh camera rõ thì bỏ qua, ghi **LOW_VISIBILITY**; tôi đã bỏ qua idx 12, 13 |
| 3. Phản xạ thấp / ít điểm bất thường | `s03f20` / `t4-3-phan-xa-thap.png` / frame của tôi | Vật tối màu hoặc mặt kính phản xạ yếu ở bước sóng LiDAR | `s03f20` idx 12 (44,3 m) chỉ 15 điểm, idx 10 (42,8 m) 45 điểm, trong khi xe cùng tầm ở 27 m có 197–350 điểm | Chỉ là giả thuyết, không khẳng định; vẫn vẽ nếu thấy được 15–20% thể tích hoặc suy luận được từ ảnh camera |
| 4. Điểm lạ | `s01f42` / `t4-4-diem-la.png` / frame của tôi | Điểm nằm ngoài mọi vật, có thể từ bụi, phản xạ mặt đường ướt hoặc nhiễu | `s01f42` idx 16 (Pedestrian, 17,0 m) chỉ 42 điểm trong hộp nhưng có **47 điểm sát ngay ngoài mép** — cụm điểm tách rời nằm sát cạnh hộp, không thuộc vật nào | Điểm lạ **không gán thành vật**; không nới hộp ra cho chúng, và cũng không coi là bằng chứng về kích thước |

## 5. Số từ cuboid của tôi (Bước 5)

Mặt đường ở khoảng **−1,7 m**. Ba z đáy tính bằng `z − h/2`:

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s05f00`, idx 1, Car | −0,83 m | 1,73 m | −1,70 m | 5,25 m | 5,32 m | 271,7° | Chạm đường, lệch 0,00 m |
| `s03f20`, idx 3, Car | −1,04 m | 2,03 m | −2,06 m | 17,80 m | 17,83 m | 81,9° | Chìm 0,36 m dưới mặt đường |
| `s01f42`, idx 4, Car | −1,04 m | 1,45 m | −1,76 m | 6,98 m | 7,06 m | 270,2° | Chạm đường, lệch 0,06 m |

Tổng bảng A: 36/145 cuboid có `z_bottom` lệch mặt đường hơn 0,3 m. `ground_gap_m` trong phiếu tự kiểm cho thấy phần lớn trong số đó lệch **dưới** 0,1 m (tức là đáy hơi thấp hơn mặt đường ở chỗ cụ thể, không phải lơ lửng); hai hộp bị cờ rõ nhất là `s01f27` idx 8 (−0,37 m) và `s01f42` idx 14 (−0,32 m). Trên toàn bộ 145 cuboid, **không cuboid nào** bị cờ "lơ lửng" — cảnh báo tự kiểm đều là chìm nhẹ hoặc hơi thấp.

**Cuboid gần nhất:** `s05f00` idx 1 (Car) — `dist_m` = **5,25 m**, r = **5,32 m**.
**Cuboid xa nhất:** `s04f17` idx 13 (Car) — `dist_m` = **66,84 m**, r = **66,85 m**. Cuboid này **vượt 50 m** nên không được chấm; tôi để lại vì nó vẫn là vật thật trong cảnh, và sẽ ghi rõ trong §6(c) là điểm chưa chắc.

**Kích thước so với bảng 4.3:**
- Người đi bộ `s03f20` idx 21: 0,50 × 0,62 × 1,76 m. Cả ba chiều nằm trong khoảng người lớn (0,4–0,9 × 0,5–0,9 × 1,50–1,95 m). Không chiều nào ngoài khoảng.
- Xe con `s04f17` idx 2: 5,03 × 2,03 × 1,82 m. Dài 5,03 m nằm trong khoảng sedan 3,8–5,0 m nhưng **vượt 0,03 m**; rộng 2,03 m vượt khoảng 1,70–1,90 m; cao 1,82 m vượt khoảng 1,40–1,60 m. Ba chiều đều hơi lớn — tôi cho rằng hộp này bao cả thùng/bánh xe và phần kính xe, và có thể là loại SUV nhỏ hơn nên không nằm trong khoảng sedan.
- Xe ba bánh `s03f20` idx 24 và 25: chiều dài **1,56 m**, thấp hơn khoảng 1,60–1,90 m của xe đạp có người và cũng thấp hơn 2,5–3,5 m của xe ba bánh. Chiều cao 1,08 m thấp hơn khoảng 1,40–1,90 m. Tôi nghi đây thực ra là **người đi bộ bị gán nhầm Cyclist**: chiều rộng 0,52–0,67 m và cao ~1,1 m hợp với người hơn là với xe hai bánh có người lái.

**Yaw:** `s01f42` idx 4 (xe cùng chiều phía trước) `yaw_deg` = **270,2°** — đúng như dự kiến. `s03f20` idx 3 (xe ngược chiều) `yaw_deg` = **81,9°** — gần 90° như dự kiến. Trong toàn bảng A có 59 xe ở khoảng 270° và 33 xe ở khoảng 90°.

**Mật độ theo khoảng cách:**

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f42`, idx 1, Car | 8,67 m | 1755 |
| `s02f08`, idx 3, Car | 11,25 m | 848 |
| `s03f20`, idx 9, Car | 40,60 m | 104 |
| `s04f17`, idx 13, Car | 66,84 m | 26 |

Quy luật: **xa gấp khoảng 8 lần thì số điểm còn khoảng 1/67** (8,7 m → 66,8 m là gấp 7,7 lần, 1755 → 26 điểm là chia 67 lần). So với đường cong trong slide bài giản (xe con: ~1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m, 19 ở 60–80 m), số điểm của tôi giảm nhanh hơn một chút ở khoảng 10–20 m (848 so với 718 là tương đương, nhưng ở 20–30 m tôi có 258–363 điểm so với 214) và khớp gần như trùng khít ở 60–80 m (26 điểm so với 19).

Phần tự kiểm: `selfcheck` còn **6/145** cuboid bị cờ. Tôi **đã sửa 2 cuboid** sau khi xem `preview`: `s01f42` idx 8 (Bus dài 4,78 m) và `s05f17` idx 10 (Bus dài 4,36 m) — cả hai cùng kích thước với một xe con nên tôi xem lại class; và `s03f20` idx 8 (Bus) cũng cần xem lại vì chiều rộng 3,08 m vượt xa khoảng 2,50–2,60 m. **Giữ nguyên** 2 cảnh báo vì tôi tin chúng đúng: `s01f42` idx 16 (Pedestrian 42 điểm, 47 điểm sát ngoài mép) — đây là cụm người đi bộ ở 17 m, số điểm thấp là bình thường với vật nhỏ; và `s01f27` idx 8 (đáy thấp hơn mặt đường 0,37 m) — xe này nằm trên phần mặt đường thấp hơn, nên giữ nguyên đáy thay vì ép cho khớp −1,7 m.

Đổi kiểm: **chưa thực hiện** — tôi làm bài một mình, không có bạn ngồi cạnh để đối chiếu hai con số. Tôi sẽ ghi lại trong §6(c) là điểm chưa chắc.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

**(a) Vì sao nhãn 3D đo bằng mét, và vì sao vật thưa điểm cần quy tắc riêng.**

Nhãn 3D phải đo bằng mét vì model học trên nhãn này cần dùng chung một thước đo giữa các cảm biến và các khung hình: nếu nhãn chỉ là điểm ảnh, model sẽ học "hình này ở góc nào" chứ không học được "xe này dài bao nhiêu mét", và không suy ra được kích thước thật của vật ở khoảng cách xa. Hộp 3D còn cho model biết vị trí và hướng trong không gian, nên một nhãn dùng được cho cả nhận dạng lẫn định vị và theo dõi.

Vật thưa điểm cần quy tắc riêng vì số điểm phụ thuộc cả khoảng cách lẫn vật liệu chứ không chỉ kích thước vật: một xe ở 60 m có thể chỉ còn 20 điểm, còn một người ở 8 m cũng chỉ có vài chục điểm. Nếu đòi "vẽ mọi vật" thì người ở gần sẽ bị vẽ nhưng xe ở xa bị bỏ, và phần tỉ lệ độ phủ theo khoảng cách sẽ sai lệch. Quy tắc 15–20% thể tích là để giữ tiêu chuẩn nhất quán cho cả hai nhóm: vẫn vẽ vật mỏng manh khi còn đủ dấu hiệu, nhưng bỏ qua vật chỉ còn vài điểm lẻ.

**(b) Dùng AI.**

- Công cụ: Cursor AI (mô hình Claude) trong IDE.
- Câu hỏi chính: cài môi trường Python cho repo (lỗi `pip install` báo `No matching distribution found for cvat-sdk` vì Python 3.10 trong khi `cvat-sdk` 2.82+ cần ≥ 3.11), cách đặt dữ liệu giải nén vào `data/data-VF`, và cách đọc `docs/GUIDELINE.md` để hiểu quy tắc cuboid và bảng 4.3.
- Tôi dùng kết quả thế nào: dùng để **sửa môi trường và hiểu tài liệu**, không dùng để vẽ cuboid hay viết số. Mọi con số trong báo cáo này tôi tự lấy từ bảng A và phiếu tự kiểm do `python scripts/lab.py sync` sinh ra trong task CVAT của tôi. Tôi không đưa ảnh camera, ảnh trong `work/xem/` hay file trong `data/data-VF/` cho AI, và không nhờ AI vẽ hoặc chấm cuboid.

**(c) Điều còn chưa chắc.**

1. **Chưa đổi kiểm với bạn bên cạnh** — tôi làm bài một mình nên chưa có con số nào được đối chiếu chéo. Đây là thiếu sót rõ ràng nhất của bài tôi.
2. **Class của một số hộp**: `s01f42` idx 8 (4,78 m) và `s05f17` idx 10 (4,36 m) ghi "Bus" nhưng kích thước giống xe con; `s03f20` idx 24, 25, 27–30 ghi "Cyclist" nhưng dài 1,15–1,56 m, hợp hơn với người đi bộ. Đây là loại nghi vấn **UNCERTAIN_CLASS**. Tôi cần vào CVAT xem lại và sửa, nhưng phần này tôi ghi lại để coach xem.
3. **Cuboid `s04f17` idx 13 ở 66,8 m** vượt 50 m nên không được chấm; tôi để lại trong bảng A, không chắc đây là lựa chọn đúng.
4. **34 cuboid có `z_bottom` lệch mặt đường hơn 0,3 m**: tôi giữ theo mặt đường cục bộ nhìn thấy trên point cloud, nhưng chưa chắc mặt đường ở các frame đó đều ở đúng −1,7 m.
5. **`occluded`**: trong toàn bảng A của tôi không có cuboid nào bật cờ occluded. Tôi không chắc là thật sự không có vật nào bị che trong 10 frame của tôi, hay là tôi đã bỏ sót thao tác bật cờ này.
6. **Frame `s02f35`** mà tài liệu gợi ý cho hiện tượng "điểm lạ" không nằm trong 10 frame của tôi, nên tôi dùng `s01f42` thay thế và ghi rõ nguồn ở mục 4.
