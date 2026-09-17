# Bài tập 4: Lọc lỗi thông minh - Đừng ngắt mạch oan uổng

## Phần 1 – Thiết kế luồng (Lưu đồ thuật toán)

Dưới đây là lưu đồ thể hiện cách Circuit Breaker phân loại lỗi:

```mermaid
graph TD
    A[Checkout-Service gọi Promo-Service] --> B{Có lỗi xảy ra không?}
    B -- Không --> C[Thành công, trả về kết quả]
    B -- Có --> D{Lỗi này là Lỗi Nghiệp Vụ?<br>(VoucherNotFoundException, 404...)}
    D -- Có --> E[BỎ QUA (Ignore).<br>Không tính vào tỷ lệ lỗi của Circuit Breaker.<br>Trả lỗi về cho Client.]
    D -- Không --> F{Lỗi này là Lỗi Hệ Thống?<br>(Timeout, ConnectException...)}
    F -- Có --> G[GHI NHẬN (Record).<br>Tính vào tỷ lệ lỗi (Failure Rate) của Circuit Breaker.]
    F -- Không --> H[Xử lý theo cấu hình mặc định<br>(Thường là ghi nhận tất cả Exception)]
    
    G --> I{Tỷ lệ lỗi > Ngưỡng (Threshold)?}
    I -- Có --> J[NGẮT MẠCH (Trạng thái OPEN)]
    I -- Không --> K[Tiếp tục hoạt động (Trạng thái CLOSED)]
    
    classDef ignored fill:#a5d6a7,stroke:#2e7d32,stroke-width:2px;
    classDef recorded fill:#ef9a9a,stroke:#c62828,stroke-width:2px;
    class E ignored
    class G recorded
```

**Giải thích luồng:**
- Khi gặp lỗi `VoucherNotFoundException` (nhập sai mã), Circuit Breaker sẽ phớt lờ lỗi này, không tăng bộ đếm lỗi. Hệ thống vẫn an toàn, không bị ngắt mạch oan uổng.
- Khi gặp lỗi mạng `TimeoutException` hoặc `ConnectException`, Circuit Breaker sẽ ghi nhận đây là sự cố hệ thống và tính vào tỷ lệ lỗi. Khi chạm ngưỡng, mạch sẽ mở để bảo vệ hệ thống.
