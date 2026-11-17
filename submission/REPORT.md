# Báo cáo Day 08 — <<MSHV - Họ và tên>>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | <<YOLO26m>> | <<theo lớp, từ threshold_sweep>> | <<>> |
| three_wheeler | <<YOLOE-26s, prompt: ...>> | <<>> | <<>> |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<>>
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: <<chất lượng mask, tốc độ, RAM, quyền truy cập>>
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? <<>>
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): <<>>
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: <<>>

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk, lần 1) | SAM 2.1 base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | **Có**: mask lan sang người lái (quần, áo) dù đã click âm lên người lái; còn trùm cả 2 người đứng **phía sau** thùng xe (nhìn xuyên qua khung hở) | rò sang vật nhìn xuyên qua khung; 1 click âm không đủ | `tuktuk_img013.jpg` |
| img_013 (lần 2) | SAM 2.1 base | 6 (2 dương, 4 âm) | Không | **Sửa quá tay**: thêm 3 click âm thì mask co lại chỉ còn mui + viền khung + chắn bùn (3 polygon, 12% ảnh), mất thân xe, ghế, bánh | `tuktuk_img013_v2.jpg` |
| img_013 (lần 3, chốt) | SAM 2.1 base | 7 (4 dương, 3 âm) | Không: loại được người lái và 2 người phía sau; chỉ còn lẹm ở bàn tay / bàn chân người lái | đủ cả mui, thân, ghế, bánh (1 polygon, 39% ảnh); cần click dương trải đều các phần của xe | `tuktuk_img013_v3.jpg` |
| img_032 (xích lô trước) | SAM 2.1 base | 3 (2 dương, 1 âm) | Có **đệm ghế / tấm phủ hoa** (gắn cố định, đúng là thuộc xe); không có người | không tách khe giữa nan bánh / khung (lỗ lớn bị lấp, trái mục 4.2.2); bánh sau bị xe khác che nên không có | `xichlo_img032.jpg` |
| img_002 (tuk-tuk xanh) | SAM 2.1 base | 3 (2 dương, 1 âm) | Người lái **được loại** nhờ 1 click âm; nhưng mask trùm lên **người đi bộ phía sau** nhìn xuyên qua khoang khách | rò sang nền nhìn xuyên khoang hở (cùng lỗi img_013 lần 1) | `tuktuk_img002.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc **4.2.2 (lỗ thủng)** giúp quyết định khe nan bánh xích lô phải bỏ ra, nhưng SAM không tự làm, phải sửa tay polygon. Quy tắc **4.2.4 (vật chạm nhau, vùng chạm thuộc vật phía trước)** giúp tách người lái khỏi tuk-tuk. **Thiếu:** guideline chưa nói vật **nhìn xuyên qua** khoang hở / kính (người phía sau tuk-tuk) thuộc mask nào (gần với câu hỏi mở 5 về kính xe). Đề xuất v1.1: phần nền nhìn xuyên khoang hở **không** thuộc mask xe; người lái / khách là mask `person` riêng.
- Rút ra: 1 click âm đủ khi vật cần loại nằm **trước** xe (người lái img_002), không đủ khi nền nhìn **xuyên qua** xe; thêm click âm mà thiếu click dương thì mask sụp (img_013 lần 2). Mask đúng hình không sửa được nhãn lớp: ở Part 2, box YOLO26 gọi tuk-tuk img_013 là `car` thì mask SAM cũng mang nhãn `car`.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
