# THỰC HÀNH SẮP XẾP VÀ VẼ SƠ ĐỒ TUẦN TỰ CHỨC NĂNG ĐẶT XE RIKKEILOGISTICS

## 1. Phân tích các thành phần tham gia

Sơ đồ tuần tự gồm 4 thành phần:

| Thành phần        | Loại   | Vai trò                                                 |
| ----------------- | ------ | ------------------------------------------------------- |
| Khách hàng        | Actor  | Gửi yêu cầu đặt đơn vận chuyển và nhận kết quả          |
| App Điều Phối     | Object | Tiếp nhận đơn, gửi yêu cầu tìm tài xế và xử lý phản hồi |
| Điện thoại Tài xế | Object | Nhận yêu cầu và phản hồi Đồng ý/Từ chối                 |
| Chuyến Đi (Trip)  | Object | Lưu thông tin chuyến đi, chỉ được tạo khi tài xế đồng ý |

---

# 2. Sắp xếp đúng thứ tự thời gian

Danh sách ban đầu bị xáo trộn. Dựa vào nghiệp vụ, thứ tự đúng là:

### Bước 1 – Khách hàng gửi yêu cầu đặt đơn

```text
Khách hàng → App Điều Phối
taoDonHang()
```

**Loại thông điệp:** `Sync`

Khách hàng gửi yêu cầu đặt đơn cho App Điều Phối và cần chờ hệ thống xử lý để nhận kết quả cuối cùng.

---

### Bước 2 – App Điều Phối gửi yêu cầu đến tài xế

```text
App Điều Phối → Điện thoại Tài xế
yêu cầu nhận đơn
```

**Loại thông điệp:** `Async`

App Điều Phối gửi yêu cầu cho tài xế nhưng **không chờ ngay lập tức**. Tài xế có thể xử lý và phản hồi sau.

---

### Bước 3 – Tài xế phản hồi

```text
Điện thoại Tài xế → App Điều Phối
Đồng ý / Từ chối
```

**Loại thông điệp:** `Async Message`

Đây là một thông điệp độc lập từ Điện thoại Tài xế về App Điều Phối. Vì bước 2 là Async nên phản hồi này **không được xem mặc định là Return Message**.

---

### Bước 4 – Khối rẽ nhánh `alt`

Sau khi App Điều Phối nhận được phản hồi từ tài xế, hệ thống phân nhánh thành 2 trường hợp:

```text
alt
├── [Tài xế đồng ý]
│      → Tạo Chuyến Đi (Trip)
│
└── [Tài xế từ chối]
       → Không tạo Chuyến Đi
```

---

### Bước 5a – Tài xế đồng ý: tạo Chuyến Đi

```text
App Điều Phối → Chuyến Đi (Trip)
Create Trip
```

**Loại thông điệp:** `Create`

Bản ghi Trip mới chỉ được tạo khi tài xế **đồng ý nhận đơn**.

Lifeline của `Trip` phải **bắt đầu tại chính thời điểm nhận Create Message**, không được kéo dài từ đầu sơ đồ.

---

### Bước 5b – Tài xế từ chối

Không tạo `Trip`.

```text
[Tài xế từ chối]

Không tạo Chuyến Đi
```

Sau đó hệ thống đi đến bước báo kết quả cuối cùng.

---

### Bước 6 – App Điều Phối báo kết quả cho Khách hàng

Sau khi hoàn thành một trong hai nhánh của `alt`, App Điều Phối gửi kết quả cuối cùng cho Khách hàng:

```text
App Điều Phối → Khách hàng
Đã tìm thấy xe / Không tìm được tài xế
```

**Loại thông điệp:** `Return`

Đây là phản hồi cuối cùng cho yêu cầu `taoDonHang()` được gửi bằng Sync ở bước 1.

---

# 3. Bảng tổng hợp thứ tự và loại thông điệp

| Bước | Từ                | Đến               | Thông điệp         | Loại       |
| ---: | ----------------- | ----------------- | ------------------ | ---------- |
|    1 | Khách hàng        | App Điều Phối     | `taoDonHang()`     | **Sync**   |
|    2 | App Điều Phối     | Điện thoại Tài xế | `yêu cầu nhận đơn` | **Async**  |
|    3 | Điện thoại Tài xế | App Điều Phối     | `Đồng ý / Từ chối` | **Async**  |
|   4a | App Điều Phối     | Chuyến Đi (Trip)  | `createTrip()`     | **Create** |
|   4b | —                 | —                 | Không tạo Trip     | —          |
|    5 | App Điều Phối     | Khách hàng        | `Đã tìm thấ        |            |
