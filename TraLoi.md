# Trả lời câu hỏi bổ sung – Buổi 9

Họ tên: Đàm Thái An – MSSV: 23710201

**1. Giá trị dẫn xuất.**
`errors` và `valid` chỉ phụ thuộc vào `name`, `phone`, `address` và giỏ hàng, nên tính trực tiếp mỗi lần render thì luôn khớp với dữ liệu hiện tại. Nếu lưu bằng `useState` riêng, ta phải nhớ cập nhật chúng ở mọi chỗ gọi `setName`, `setPhone`, `setAddress`; quên một chỗ là lỗi đỏ hiển thị sai hoặc nút "Xác nhận đặt hàng" sáng/mờ sai so với nội dung ô nhập.

**2. Thứ tự gọi.**
Nếu `clear()` chạy trước `await createOrder(...)` mà mất mạng thì giỏ đã bị xoá (cả trong store lẫn AsyncStorage) nhưng đơn chưa hề được tạo trên server, người dùng mất hết sản phẩm đã chọn. Đặt `clear()` ngay sau `await createOrder` là đúng vì chỉ khi server đã nhận đơn (không ném lỗi) mới xoá giỏ; nếu lỗi thì code nhảy vào `catch` và giỏ vẫn còn để bấm thử lại.

**3. PUT hay PATCH.**
`PUT /users/1` với `{ name, phone }` sẽ thay cả object, nên user trong `db.json` chỉ còn `id`, `name`, `phone`: mất `email` và `password`, và tài khoản không đăng nhập lại được. `PATCH` chỉ đổi các trường được gửi, các trường còn lại giữ nguyên, nên phù hợp với form chỉ sửa một phần hồ sơ.

**4. Cập nhật lạc quan.**
Nếu bỏ đoạn hoàn lại `read: false` trong `catch` thì khi mất mạng giao diện vẫn báo "đã đọc" trong khi server vẫn là chưa đọc; mở lại màn là thông báo quay về chưa đọc, gây khó hiểu. Nên dùng cho thao tác nhẹ, ít sai và dễ hoàn lại như thả tim / bỏ tim ở danh sách yêu thích. Không nên dùng cho thanh toán / đặt hàng, vì phải chắc server đã nhận đơn mới được báo thành công và xoá giỏ.

**5. Code bổ sung – bộ lọc trạng thái ở màn Đơn hàng** (`src/screens/OrdersScreen.tsx`)

```tsx
// state
const [filter, setFilter] = useState<OrderStatus | "all">("all");

// một chỗ sửa trong dữ liệu truyền cho FlatList
const visibleOrders = filter === "all" ? orders : orders.filter((o) => o.status === filter);
<FlatList data={visibleOrders} ... />

// hàng nút lọc: Tất cả / Chờ xác nhận / Đang giao / Hoàn tất / Đã huỷ
{FILTERS.map((f) => (
  <Pressable key={f.key} onPress={() => setFilter(f.key)} style={[styles.chip, f.key === filter && styles.chipActive]}>
    <Text style={[styles.chipText, f.key === filter && styles.chipTextActive]}>{f.label}</Text>
  </Pressable>
))}
```

Khi lọc ra danh sách rỗng, màn hiện "Không có đơn nào ở trạng thái này" (thay vì "Bạn chưa có đơn hàng nào").
