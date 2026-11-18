# Báo cáo Day 08 — 2A202603211 - Võ Duy Thành

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.15 | sweep: recall 16/19 = 84% ở 0.15 so với 15/19 = 79% ở 0.3; không ngưỡng nào ≤ 0.5 đạt 90%. Chọn thấp vì xoá box thừa nhanh hơn vẽ box sót (r02 → r03: lên 0.5 thì img_007 person 15 → 6). **Sau review thấy là sai lầm**: person deleted 98 box (mục 2) |
| bicycle | YOLO26m | 0.15 | recall 4/5 = 80% ở 0.15 so với 3/5 = 60% ở 0.3; chỉ 5 vật gold nên kết luận yếu |
| motorbike | YOLO26m | 0.3 (mặc định) | sweep: ở 0.3 precision 6/8 = 75%, recall 6/8 = 75%, cân bằng nhất |
| car | YOLO26m | 0.4 | recall giữ 7/8 = 88% như ở 0.3, precision tăng 7/14 = 50% → 7/12 = 58% |
| bus / truck | YOLO26m | 0.3 (mặc định) | 1 bus và 0 truck trong gold (< 5 vật): không chọn ngưỡng riêng |
| three_wheeler | YOLOE-26s-seg, prompt: `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15 | sweep: recall 7/7 = 100%, precision 7/9 = 78% ở 0.15 (r04, r07). Ở Part 2 (r04) tôi đoán bằng mắt cần ~0.45 vì FP ở 0.37–0.40; số đo trên gold cho thấy 0.45 sẽ mất 4/7 xe |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15,person=0.15,car=0.4,bicycle=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` → 722 box (24 batch + 4 gold).
- **Phiên bản SAM bạn chọn**: SAM 2.1 base (`sam2.1_b.pt`). Máy chỉ có CPU nhưng 13.6 s/ảnh (r05) vẫn chấp nhận được cho vài ảnh; mask bám biên tốt hơn MobileSAM; SAM 3 cần token HuggingFace + GPU nên không dùng.
- Lỗi **không sửa được bằng đổi ngưỡng**: YOLO26 (danh mục COCO) không có lớp ba bánh nên gọi tuk-tuk là `car` (img_013 0.42, img_004 0.37 / 0.34), xích lô là `bicycle` (img_032 0.70). Lên ngưỡng 0.5 (r03) vẫn còn `bicycle` 0.70 trên xích lô. Sửa bằng **đổi model** cho lớp đó (YOLOE gõ chữ, cho YOLOE thắng khi merge); phần còn sót sửa tay khi review (10 ca đổi lớp trong `review_stats.md`).
- Box **điểm cao mà sai**: img_002 xe bán tải `truck` 0.73 (bán tải là `car`); img_032 xích lô `bicycle` 0.70. **Điểm thấp mà đúng**: img_032 xích lô `three_wheeler` 0.27 (YOLOE). **Điểm thấp mà sai**: img_031 `person` 0.15 là áo khoác treo (QC #477).
- Giới hạn: ngưỡng được chọn **và** chấm trên cùng 4 ảnh gold (48 đối tượng tính trong `review_stats`, 55 kể cả `ignore_region`), nên precision / recall của máy trên gold là lạc quan; bicycle (5 vật), bus (1), truck (0) gần như không có thông tin.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 403 | 64.8% | 622 box máy |
| edited | 36 | 5.8% | 622 box máy |
| deleted | 149 | 24.0% | 622 box máy |
| added | 56 | 11.3% | 495 đối tượng sau duyệt |

(25 đối tượng quanh vùng bẫy bị loại khỏi bảng vì là lỗi nhân tạo, không phản ánh máy. Trong deleted + added có 10 ca đổi lớp, nên số vật máy **sót thật** ≈ 56 − 10 = 46.)

- Lớp máy tệ nhất: **three_wheeler**: 10 accepted / 39 box máy (10 accepted + 7 edited + 22 deleted) = 26%; YOLOE ở ngưỡng 0.15 bắt cả quầy hàng, xe máy chở hàng, xe bán tải (đã thấy ở img_002 trong r04). **person** đóng góp nhiều box rác nhất: 98 deleted / 358 box máy person = 27%, hậu quả trực tiếp của ngưỡng 0.15 (QC còn sót một box áo khoác treo ở img_031). **car** bị sót nhiều nhất: 18 added (trên 60 car trong release); một phần là tuk-tuk / bán tải đổi lớp.
- So với gold: precision / recall của **máy** = 41/66 = 62.1% (KTC 95%: 50–73%) / 41/48 = 85.4% (KTC 95%: 73–93%), mẫu số: 48 đối tượng gold trên 4 ảnh. Recall 85% nghĩa là cứ ~7 vật thì máy sót 1: `added` thấp ở batch không chứng minh máy không sót, chỉ là tôi không thấy chỗ sót.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **8/12 = 66.7%** (KTC 95%: 39–86%, dưới ngưỡng 80%). Theo loại: missing 3/3, extra 2/3, wrong_class 2/3, **loose 1/3**. Lọt: img_005 (loose), img_006 (loose), img_008 (extra), img_022 (wrong_class).
- Vì sao: tôi bắt đủ 3/3 vật bị xoá vì quét ảnh trước khi nhìn nhãn máy (bước quét ảnh và "bỏ sót" của checklist có làm). Nhưng **bước "lỏng"** gần như bị bỏ: box nằm đúng chỗ là tôi chấp nhận, không phóng to kiểm 4 cạnh. QC xác nhận lại đúng lỗi này: img_006 #80 lỏng ~45 px ở cạnh trái. Bẫy extra / wrong_class lọt ở ảnh đông vật, khi đã duyệt được hơn nửa lô (40 phút liền, không nghỉ).
- Accepted 64.8% mà bắt bẫy chỉ 66.7%: tôi không "bấm Accept hết" (đã xoá 149 và vẽ thêm 56), nhưng **mức độ soi từng box không đều**: lỗi lớn (thiếu vật, sai hẳn lớp) tôi bắt được; lỗi nhỏ về hình học thì tin máy. Đó chính là thiên kiến tự động hóa: box máy "trông hợp lý" thì tôi không kiểm.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 8 phút / 4 ảnh = **2.0 phút/ảnh** (55 đối tượng → 0.15 phút/đối tượng) vs duyệt 40 phút / 24 ảnh = **1.67 phút/ảnh** (537 đối tượng → 0.07 phút/đối tượng). Tính theo đối tượng, duyệt nhanh gấp ~2 lần. Thiên lệch: tôi gán gold trước nên đã quen kiểu cảnh và lớp; ảnh batch đông hơn (22 vs 14 đối tượng/ảnh); và 40 phút duyệt nhanh hơn mức GUIDE gợi ý (~3 phút/ảnh), một phần giải thích vì sao bẫy loose bị lọt.
- Kết luận: **có, nhưng không đồng đều theo lớp**. Đáng dùng cho person / motorbike / car (accepted cao, chỉ cần soi cạnh box). **three_wheeler** chỉ 26% accepted: máy chỉ có ích để *gợi ý chỗ có xe*, còn box và lớp gần như phải làm lại. Lô sau: person 0.3 thay vì 0.15 để giảm box rác.
- **Ba quy tắc giảm thiên kiến của tôi** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. **Kiểm 4 cạnh**: với mọi box `three_wheeler`, `car` và mọi box `conf` < 0.5, phóng to và kiểm cả 4 cạnh trước khi chấp nhận. Kiểm bằng: bẫy loose vòng sau bắt 3/3; QC không còn lỗi `sai_vi_tri_kich_thuoc`.
  2. **Đếm trước khi xem nhãn**: ẩn nhãn máy, ghi số vật mỗi lớp mình đếm được, rồi mới hiện nhãn và so với số box máy. Kiểm bằng: mỗi ảnh có một con số đếm tay trong ghi chú; bẫy missing giữ 3/3.
  3. **Nghỉ theo khối**: duyệt từng khối 8 ảnh, nghỉ 2 phút giữa các khối, ghi thời gian từng khối vào `time_log`. Kiểm bằng: không khối nào < 1.5 phút/ảnh; bắt bẫy tổng ≥ 10/12.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk, lần 1) | SAM 2.1 base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | **Có**: mask lan sang người lái (quần, áo) dù đã click âm lên người lái; còn trùm cả 2 người đứng **phía sau** thùng xe (nhìn xuyên qua khung hở) | rò sang vật nhìn xuyên qua khung; 1 click âm không đủ | `tuktuk_img013.jpg` |
| img_013 (lần 2) | SAM 2.1 base | 6 (2 dương, 4 âm) | Không | **Sửa quá tay**: thêm 3 click âm thì mask co lại chỉ còn mui + viền khung + chắn bùn (3 polygon, 12% ảnh), mất thân xe, ghế, bánh | `tuktuk_img013_v2.jpg` |
| img_013 (lần 3, chốt) | SAM 2.1 base | 7 (4 dương, 3 âm) | Không: loại được người lái và 2 người phía sau; chỉ còn lẹm ở bàn tay / bàn chân người lái | đủ cả mui, thân, ghế, bánh (1 polygon, 39% ảnh); cần click dương trải đều các phần của xe | `tuktuk_img013_v3.jpg` |
| img_032 (xích lô trước) | SAM 2.1 base | 3 (2 dương, 1 âm) | Có **đệm ghế / tấm phủ hoa** (gắn cố định, đúng là thuộc xe); không có người | không tách khe giữa nan bánh / khung (lỗ lớn bị lấp, trái mục 4.2.2); bánh sau bị xe khác che nên không có | `xichlo_img032.jpg` |
| img_002 (tuk-tuk xanh) | SAM 2.1 base | 3 (2 dương, 1 âm) | Người lái **được loại** nhờ 1 click âm; nhưng mask trùm lên **người đi bộ phía sau** nhìn xuyên qua khoang khách | rò sang nền nhìn xuyên khoang hở (cùng lỗi img_013 lần 1) | `tuktuk_img002.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc **4.2.2 (lỗ thủng)** giúp quyết định khe nan bánh xích lô phải bỏ ra, nhưng SAM không tự làm, phải sửa tay polygon. Quy tắc **4.2.4 (vật chạm nhau, vùng chạm thuộc vật phía trước)** giúp tách người lái khỏi tuk-tuk. **Thiếu:** guideline chưa nói vật **nhìn xuyên qua** khoang hở / kính (người phía sau tuk-tuk) thuộc mask nào (gần với câu hỏi mở 5 về kính xe). Đã chốt trong v1.1 (CAL-06): phần nền nhìn xuyên khoang hở **không** thuộc mask xe; người lái / khách là mask `person` riêng.
- Rút ra: 1 click âm đủ khi vật cần loại nằm **trước** xe (người lái img_002), không đủ khi nền nhìn **xuyên qua** xe; thêm click âm mà thiếu click dương thì mask sụp (img_013 lần 2). Mask đúng hình không sửa được nhãn lớp: ở Part 2, box YOLO26 gọi tuk-tuk img_013 là `car` thì mask SAM cũng mang nhãn `car`.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không thử (chỉ làm offline bằng `sam2_points.py`).

## 6. Calibration (Part 6)

- Số quyết định: **12** (`calibration_decisions.csv`): 3 `loi_may`, 7 `cau_hoi_mo`, 1 `bay_lot`, 1 từ QC. Quan trọng nhất: **CAL-01** (xe ba bánh luôn là `three_wheeler`, đổi lớp mọi box car/truck/bicycle của YOLO26 trên tuk-tuk / xích lô) vì đây là lỗi hệ thống không ngưỡng nào sửa được; và **CAL-06** (khoang hở không kính: nền nhìn xuyên qua không thuộc mask xe) vì SAM 2.1 mắc lỗi này ở cả img_013 và img_002.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt cả 7 câu hỏi mở (mục 1); thêm 4 ca khó (mục 2): three_wheeler thắng car/truck/bicycle, bán tải = car (img_002), quần áo treo không phải person (img_031), quy tắc mask 4.2.5 cho khoang hở (img_013, img_002).
- Nguyên nhân gốc: **guideline mơ hồ** ở 7 câu hỏi mở + ca khoang hở (CAL-04 … CAL-10, CAL-06); **máy** ở CAL-01 … CAL-03 (thiếu lớp ba bánh, ngưỡng person quá thấp); **tôi** ở CAL-11 (box lỏng lọt) và CAL-12 (vẽ `ignore_region` lên giá quần áo): hai ca này quy tắc v1.0 đã rõ nên **không** đổi guideline.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` vì tôi không tick ca nào + 5 ngẫu nhiên, seed 42): img_003, img_006, img_014, img_031, img_035; 89 đối tượng. Người kiểm: AI (Claude) xem ảnh `work/qc/review/` và tôi đối chiếu (**hạn chế**: không phải người kiểm độc lập, và tôi vừa là người duyệt).
- Lỗi theo **đối tượng**: 5/89 = **5.6%** · theo **ảnh**: 2/5 = **40%** (ngưỡng Release ≤ 2%).
- Top lỗi: **thừa** 3 (img_031: box person 0.15 trên áo khoác treo #477; 2 `ignore_region` vẽ lên giá quần áo #474, #475); **sai vị trí / kích thước** 2 (img_006 #80 motorbike lỏng ~45 px; img_031 #479 person cắt mất phần chân còn nhìn thấy). Không gặp bỏ sót, sai lớp, sai thuộc tính trong mẫu.
- **Giới hạn mẫu nhỏ**: 2 ảnh lỗi / 5 ảnh, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **77%** (`qc_report_auto.md`). Với 5 ảnh, kể cả 0 lỗi cũng không chứng minh được ≤ 2% (quy tắc 3: 0/5 vẫn có thể tới ~60%). Ở đây số liệu đã vượt ngưỡng rõ ràng nên kết luận Rework không phụ thuộc vào cỡ mẫu.
- Script kiểm được: lớp hợp lệ, đủ ảnh, accepted/edited/deleted/added, bắt bẫy, tỉ lệ lỗi theo phiếu. **Không** kiểm được: box có ôm sát vật thật không (phải nhìn ảnh, như img_006 #80), vật mà cả tôi lẫn máy cùng bỏ sót ngoài 4 ảnh gold, và tôi có thật sự theo checklist hay không.

**Quyết định: Rework**, ngày 2026-11-17. Lý do (đi theo Hold → Rework → Release): (1) **Hold?** Không: cả 7 câu hỏi mở đã chốt trong v1.1, dữ liệu dùng được. (2) **Rework?** Có, cả ba điều kiện: ảnh lỗi 2/5 = 40% > 2%; lỗi lặp "thừa" 3/5 lỗi (≥ 3 lỗi và ≥ 50%); bắt bẫy 8/12 = 67% < 80%.
Nếu Rework / Hold, việc cần làm tiếp: (a) sửa 5 lỗi QC (img_006 #80; img_031 #474, #475, #477, #479) và áp quy tắc v1.1 cho cả lô (đổi lớp xe ba bánh / bán tải, xoá `ignore_region` không có vật); (b) duyệt lại **toàn bộ** 24 ảnh chỉ cho bước "lỏng" (quy tắc 1 ở mục 4) vì loose là điểm yếu hệ thống; (c) export `submission/labels/qc_fixed.json`, chạy lại `build_release.py` và `qc_sample.py --force` với mẫu mới (≥ 10 ảnh), ghi vòng 2 vào báo cáo; (d) lô sau dùng person 0.3.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Claude AI.
- Gold có bị nhiễm không: **không**. Tôi gán xong và export `gold_manual.json` trước khi chạy bất kỳ model nào lên ảnh gold; AI không xem ảnh gold trước đó.
- Máy / GPU / CVAT: Windows 10, **chỉ CPU** (torch 2.15 CPU), YOLO26m ~1.7–2.8 s/ảnh, SAM 2.1 base ~13.6 s/ảnh. CVAT chạy bằng Docker. Vướng mắc: lần đầu chạy YOLOE phải tải thêm `mobileclip2_b.ts` (~242 MB) và cài `clip`; script in tiếng Việt lỗi trên console Windows (cp1252) → đặt `PYTHONIOENCODING=utf-8`; Excel lưu `time_log.csv` bằng dấu `;` → sửa lại bằng trình soạn thảo.
- Dùng cache `data/cache/`: **không**, tự chạy cả 36 ảnh (r06, r07).
- Lỗi / bug script: `check_submission.py` chỉ nhận MSHV toàn chữ số (regex `\d{5,}`); MSHV của tôi là `2A202603211` nên script báo lỗi tiêu đề README dù tiêu đề đúng quy định.
- Một điều làm khác lần sau: duyệt chậm lại (~3 phút/ảnh thay vì 1.7) và kiểm 4 cạnh mọi box ba bánh / box điểm thấp; tick `can_review` khi phân vân (lần này không tick ca nào nên mẫu QC không có ca khó).
