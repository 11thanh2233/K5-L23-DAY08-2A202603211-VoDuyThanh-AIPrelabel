# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603211 - Võ Duy Thành

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)


| #   | Câu hỏi                                                               | Quyết định                                                                                                                                                                                                        | Ảnh ví dụ                 | `case_id` |
| --- | --------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------- | --------- |
| 1   | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`?             | Có **bàn đạp dùng được + khung xe đạp** → `bicycle`. Dáng xe tay ga, không bàn đạp → `motorbike`. Không nhìn thấy bàn đạp → `motorbike` + tick `can_review`.                                                      | img_025                   | CAL-10    |
| 2   | Khối xe máy đỗ dày **gần** camera: từng xe hay `ignore_region`?       | Xe nào **cao ≥ 20 px và thấy ≥ 50% thân xe** → vẽ từng xe. Phần còn lại của khối (bị che > 50% hoặc < 20 px) gộp vào **một** `ignore_region`.                                                                     | img_006, img_018          | CAL-04    |
| 3   | Hàng nhô ra trên xe đạp / ba gác: box xe có gồm hàng? Mask?           | Đồ **gắn cố định** vào xe (giỏ, mẹt buộc vào khung, thùng ba gác) **thuộc** box và mask xe. Hàng **chất rời** nhô ra > 20% chiều dài xe **không** tính. Mask theo cùng quy tắc.                                   | img_031                   | CAL-05    |
| 4   | Người ngồi trong xe ba bánh có mui, chỉ thấy thấp thoáng: ngưỡng 20%? | **Giữ ngưỡng 20%**, đo bằng phần thân nhìn thấy (thấy rõ **đầu + vai** là đủ để gán). Không chắc → gán `person` + tick `can_review`.                                                                              | img_002, img_013          | CAL-07    |
| 5   | Kính chắn gió / cửa sổ xe: thuộc mask xe?                             | Tấm **kính thật** thuộc mask xe (kể cả khi nhìn xuyên thấy nội thất). Khoang **hở không kính** (tuk-tuk, xích lô): nền và người nhìn xuyên qua **không** thuộc mask xe; người lái / khách là mask `person` riêng. | img_013, img_002          | CAL-06    |
| 6   | Box máy chồng nhau quá nhiều: gộp hay giữ?                            | **Một vật = một box.** Box máy cùng lớp chồng nhau IoU ≥ 0.6 trên cùng một vật → giữ box ôm sát nhất, xoá box còn lại.                                                                                            | img_015                   | CAL-08    |
| 7   | Mức tin cậy tối thiểu để chấp nhận box máy không cần sửa?             | **Không đặt.** `conf` chỉ để xếp thứ tự duyệt. Box điểm cao vẫn có thể lỏng / sai (bẫy loose lọt ở img_005, img_006). Box `conf` < 0.3 phải phóng to kiểm cả 4 cạnh.                                              | img_005, img_006, img_031 | CAL-09    |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013, img_004, img_032 | Tuk-tuk, xích lô | YOLO26m gọi tuk-tuk là `car` (0.42; 0.37 / 0.34), xích lô là `bicycle` (0.70) | `three_wheeler` | **Mục 2, lớp `three_wheeler`**: xe ba bánh có mui / thùng **luôn** là `three_wheeler`, không theo lớp máy đề xuất; khi duyệt, mọi box `car` / `truck` / `bicycle` trùng xe ba bánh phải **đổi lớp** (CAL-01) |
| img_002 | Xe bán tải đỗ bên phải | YOLO26m gọi `truck` 0.73 | `car` | **Mục 2, lớp `car` / `truck`**: thêm ảnh ví dụ ranh giới: bán tải (cabin + thùng hở, cỡ xe con) = `car`; `truck` khi có buồng lái riêng + thùng hàng lớn (CAL-02) |
| img_031 | Áo khoác đỏ treo trên giá | YOLO26m `person` 0.15 (giữ lại do ngưỡng person 0.15) | xoá | **Mục 2, lớp `person`, cột "Không gồm"**: thêm "quần áo treo trên giá / ma-nơ-canh" (CAL-03) |
| img_013, img_002 | Mask tuk-tuk trùm người đứng phía sau khoang hở | SAM 2.1 base, prompt click | loại nền nhìn xuyên khoang hở khỏi mask | **Mục 4.2, thêm quy tắc 4.2.5**: vùng nhìn xuyên qua khoang hở không kính không thuộc mask xe (CAL-06) |


## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi                                                                                                                                                                                                                                                            | Lý do (ca nào, `case_id`) |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------- |
| v1.0      | Bản giảng viên                                                                                                                                                                                                                                                      | —                         |
| v1.1      | Chốt 7 câu hỏi mở (xe đạp điện, khối xe gần camera, hàng nhô ra, người trong xe có mui, kính / khoang hở, box chồng, conf tối thiểu); thêm 4 ca khó: three_wheeler luôn thắng car/truck/bicycle, bán tải = car, quần áo treo ≠ person, quy tắc mask 4.2.5 khoang hở | CAL-01 … CAL-10           |
