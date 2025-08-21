
# 🚖 Đặt Xe Taxi Điện Nam Thắng

Form web **đặt cuốc nhanh** tối ưu mobile. Frontend **HTML + Tailwind**, backend **Google Apps Script** ghi vào **Google Sheet**.

## 💡 Tính năng
- Tiêu đề có **icon xe điện** xanh nón chuối + hiệu ứng nhẹ.
- **CTA tải app** hiển thị trong popup thành công: `taxinamthang.vn/taiapp` (link click được, hiệu ứng nổi bật).
- **Searchable select** cho Tỉnh/Quận/Phường (gõ để lọc, chọn → cascade).
- **Lấy GPS** + **reverse geocode** Nominatim (miễn phí) để gợi ý địa chỉ.
- **Loại cuốc** & **Loại xe** dạng **nút bấm**; mặc định: _Đi ngắn_ + _4 chỗ_ (nếu có).
- **Popup xác nhận** trước khi gửi; **popup thành công** kèm **số tổng đài theo Tỉnh**.
- **LocalStorage** giữ SĐT, địa chỉ (và các trường cấu hình) để đặt lại nhanh.
- Né lỗi CORS bằng `fetch(..., { mode: "no-cors" })`.

---

---------------------
equenceDiagram
autonumber
actor KH as Khách hàng
participant FE as Trình duyệt (Form)
participant GAS as Apps Script WebApp (/exec)
participant SHEET as Google Sheet (DON_HANG)


KH->>FE: Mở trang
FE->>FE: Tải locations.json & cau_hinh_xe.json<br/>Khôi phục localStorage (SĐT, address, ...)


KH->>FE: Chọn Tỉnh → Quận/Huyện → Phường/Xã (searchable)
FE->>FE: Render "Loại xe" theo Tỉnh/Huyện<br/>Auto mặc định "4 chỗ" nếu có
KH->>FE: (Tuỳ chọn) Bấm 📍 Lấy GPS
FE->>FE: Geolocation → Reverse geocode (Nominatim)<br/>Gợi ý điền "Địa chỉ chi tiết"


KH->>FE: Điền Điểm đến (tuỳ chọn), Ghi chú (tuỳ chọn)
KH->>FE: Bấm "🚕 Đặt cuốc ngay"
FE->>FE: Validate form
FE-->>KH: Popup Xác nhận (bảng tóm tắt)


KH->>FE: Bấm "✅ Xác nhận đặt xe"
FE->>GAS: fetch POST (mode="no-cors")


GAS->>GAS: Parse JSON
GAS->>SHEET: Ghi vào dòng kế tiếp (giữ 0 đầu SĐT)
SHEET-->>GAS: OK


FE-->>KH: Ẩn popup xác nhận → Mở popup Thành công + CTA tải app
FE->>FE: reset form (giữ phone/address/...)
---------------
flowchart TD
A([Mở trang]) --> B[loadData(): tải locations.json & cau_hinh_xe.json]
B --> C[Khôi phục localStorage]
C --> D{Chọn Tỉnh?}
D -->|Có| E[Đổ Quận/Huyện]
D -->|Chưa| D
E --> F{Chọn Quận/Huyện?}
F -->|Có| G[Đổ Phường/Xã + render Loại xe]
F -->|Chưa| F
G --> H{Chọn Phường/Xã?}
H -->|Có| I[Chọn/giữ Loại xe (default 4 chỗ nếu có)]
H -->|Chưa| H
I --> J{Bấm 📍 GPS?}
J -->|Có| K[Geolocation + Nominatim → gợi ý địa chỉ]
J -->|Không| L[Nhập tay địa chỉ]
K --> M
L --> M
M --> N[Bấm "🚕 Đặt cuốc ngay"]
N --> O{Validate}
O -->|Sai| P[Hiện cảnh báo]
O -->|Đúng| Q[Popup xác nhận]
Q -->|Không| N
Q -->|✅ Có| R[POST WebApp (no-cors)]
R --> S[Popup thành công + CTA]
S --> T[Reset giữ phone/address/...]
