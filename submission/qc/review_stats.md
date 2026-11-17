# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 622; đối tượng sau duyệt: 495; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 403 | 403/622 = 64.8% | 622 box máy |
| edited (sửa) | 36 | 36/622 = 5.8% | 622 box máy |
| deleted (xoá) | 149 | 149/622 = 24.0% | 622 box máy |
| added (vẽ thêm) | 56 | 56/495 = 11.3% | 495 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 46.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 251 | 9 | 98 | 14 |
| bicycle | 4 | 3 | 3 | 1 |
| motorbike | 97 | 14 | 19 | 17 |
| car | 37 | 3 | 4 | 18 |
| bus | 2 | 0 | 1 | 2 |
| truck | 2 | 0 | 2 | 2 |
| three_wheeler | 10 | 7 | 22 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 48 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 41 | 25 | 7 | 62.1% (KTC 95%: 50–73%) | 85.4% (KTC 95%: 73–93%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 1/3
  - wrong_class: 2/3
  - extra: 2/3
  - missing: 3/3
  - lọt: img_006.jpg (loose), img_008.jpg (extra), img_005.jpg (loose), img_022.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
