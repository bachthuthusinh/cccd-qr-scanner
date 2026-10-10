# Quét QR Căn cước (CCCD)

Ứng dụng web chạy trên điện thoại/máy tính để quét mã QR trên thẻ Căn cước công dân / Căn cước, tách ra các trường thông tin, rồi:

- **Lưu vào Google Sheet** (qua Google Apps Script), hoặc
- **Copy trực tiếp** — copy từng trường, copy cả dòng (dán thẳng vào Google Sheet/Excel), copy chuỗi gốc.
- **Dán nội dung thủ công** (một hoặc nhiều dòng) để phân tích và lưu về Sheet.

Không cần cài đặt: mở link là dùng được. Có thể "Thêm vào màn hình chính" để dùng như app.

## Tính năng

| Tính năng | Ghi chú |
|---|---|
| Quét bằng camera | Tối ưu cho QR nhỏ, mờ trên CCCD (xem bên dưới), tự dừng khi đọc được mã |
| Chụp bằng camera máy | Mở app camera gốc (lấy nét tốt hơn), tự tìm mã trong ảnh |
| Quét từ ảnh | Chọn ảnh có sẵn; tự dò vùng chứa mã, kể cả mã rất nhỏ trong ảnh lớn |
| Dán tay | Dán chuỗi QR, nhiều dòng = nhiều người; nội dung khác vẫn lưu nguyên văn |
| Sửa trước khi lưu | Các trường kết quả có thể chỉnh trực tiếp |
| Copy | Từng trường, cả dòng (định dạng tab cho Sheet), văn bản, chuỗi gốc |
| Google Sheet | Lưu thủ công hoặc tự động sau mỗi lần quét, có mã bảo mật tuỳ chọn |
| Lịch sử | Lưu trên máy, đánh dấu đã/chưa lưu, gửi lại, copy tất cả, tải CSV |
| Cảnh báo trùng | Báo khi quét lại số CCCD đã có trong lịch sử |

## Tối ưu quét QR nhỏ & mờ

- **Camera độ phân giải cao** (1080p / 1440p / 4K, chọn trong Cài đặt), lấy nét – phơi sáng liên tục.
- **Zoom** bằng thanh trượt, nút +/− hoặc chụm hai ngón (zoom quang nếu máy hỗ trợ, nếu không thì zoom số). Mẹo: giữ máy cách thẻ ~20 cm rồi zoom 2–3× — camera không lấy nét được khi áp sát.
- **Chạm để lấy nét**, **đèn flash**, **chọn camera** (máy nhiều ống kính).
- **Báo ảnh mờ**: khung chuyển vàng khi ảnh mờ; mờ lâu thì tự kích lấy nét lại.
- **Cắt vùng khung ngắm ở độ phân giải gốc**, tự khoanh sát vùng mã rồi phóng to, tăng tương phản và làm nét trước khi giải mã.
- **Ba bộ giải mã luân phiên**: BarcodeDetector của máy (Android Chrome) → ZXing → jsQR.
- **Dò mã trên toàn khung hình** định kỳ — ngắm lệch khung vẫn đọc được.
- **Chụp nét** (Android Chrome): chụp ảnh tĩnh độ phân giải tối đa của cảm biến rồi quét kỹ.

Kiểm thử trên ảnh thẻ giả lập (QR 160–260 px trong khung 1920×1080, tương phản thấp, có nhiễu): bộ quét mới đọc được 7/12 mức mờ, bộ quét mặc định của html5-qrcode đọc 0/12.

## Định dạng QR CCCD

```
Số CCCD|Số CMND cũ|Họ và tên|Ngày sinh(ddmmyyyy)|Giới tính|Nơi thường trú|Ngày cấp(ddmmyyyy)
```

Các trường thêm (nếu thẻ có) được ghi vào cột **Thông tin thêm**.

Cột trong Google Sheet: `Thời gian quét | Số CCCD | Số CMND cũ | Họ và tên | Ngày sinh | Giới tính | Nơi thường trú | Ngày cấp | Thông tin thêm | Dữ liệu gốc | Nguồn`

## Nối Google Sheet

1. Tạo Google Sheet mới → **Tiện ích mở rộng → Apps Script**.
2. Xoá mã mẫu, dán toàn bộ nội dung [`apps-script/Code.gs`](apps-script/Code.gs) → **Lưu**.
3. (Tuỳ chọn) đặt `TOKEN = 'chuoi-bi-mat'` để chỉ ứng dụng của bạn ghi được.
4. **Triển khai → Tùy chọn triển khai mới → Ứng dụng web**
   - Thực thi với tư cách: **Tôi**
   - Người có quyền truy cập: **Bất kỳ ai**
5. Cấp quyền khi Google hỏi, copy URL dạng `https://script.google.com/macros/s/…/exec`.
6. Mở ứng dụng → tab **Cài đặt** → dán URL (và mã bảo mật nếu có) → **Gửi thử**.

> Mỗi lần sửa `Code.gs`, vào **Triển khai → Quản lý triển khai → Chỉnh sửa → Phiên bản: Mới** để áp dụng.

## Chạy cục bộ

Camera chỉ hoạt động trên HTTPS hoặc `localhost`:

```bash
python3 -m http.server 8080
# mở http://localhost:8080
```

## Bảo mật & dữ liệu cá nhân

- Ứng dụng không có máy chủ riêng; dữ liệu chỉ đi từ trình duyệt tới Google Sheet của bạn.
- Lịch sử lưu trong trình duyệt (localStorage) — xoá bằng nút **Xoá lịch sử**.
- Thông tin CCCD là dữ liệu cá nhân (Nghị định 13/2023/NĐ-CP). Chỉ thu thập khi có sự đồng ý của chủ thẻ, giới hạn quyền truy cập Google Sheet, và dùng `TOKEN` để tránh người lạ ghi dữ liệu.

## Cấu trúc

```
index.html            Giao diện ứng dụng
scanner.js            Bộ quét QR tối ưu (camera, xử lý ảnh, giải mã)
apps-script/Code.gs   Mã Google Apps Script ghi vào Sheet
sw.js, manifest.webmanifest, icon.svg   Cài như app (PWA)
```

Thư viện giải mã đóng gói sẵn trong `vendor/`: [html5-qrcode](https://github.com/mebjas/html5-qrcode) v2.3.8 (ZXing, Apache-2.0) và [jsQR](https://github.com/cozmo/jsQR) (Apache-2.0).
