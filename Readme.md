from pathlib import Path

content = r"""# Hướng dẫn lắp đặt cảm biến độ ẩm cho trang trại Lan

## 1. Mục đích

Tài liệu này hướng dẫn chuẩn bị môi trường, kết nối cảm biến độ ẩm đất và kiểm tra hệ thống tưới tự động. Ví dụ sử dụng Raspberry Pi, thư viện `gpiozero` và một máy bơm được điều khiển qua **module relay phù hợp**.

> **An toàn:** Không kết nối máy bơm công suất lớn trực tiếp với chân GPIO. Hãy dùng relay/driver đúng thông số, nguồn điện phù hợp và nhờ người có chuyên môn kiểm tra phần điện.

## 2. Thiết bị cần chuẩn bị

- [ ] Raspberry Pi đã cài hệ điều hành và kết nối mạng
- [ ] Cảm biến độ ẩm đất tương thích với mạch đọc tín hiệu
- [ ] Module relay hoặc driver tương thích với máy bơm
- [ ] Máy bơm và nguồn điện phù hợp
- [ ] Dây nối, đầu nối và vật tư chống nước
- [ ] Máy tính có Terminal để cài đặt và chạy chương trình

## 3. Cài đặt môi trường

Mở Terminal và kiểm tra phiên bản Python bằng lệnh:

```bash
python3 --version
```

Cài thư viện điều khiển GPIO:

```bash
python3 -m pip install gpiozero
```

Nếu hệ thống của bạn quản lý gói Python bằng trình quản lý hệ điều hành, hãy làm theo hướng dẫn tương ứng với phiên bản Raspberry Pi OS đang sử dụng.

## 4. Kết nối và cấu hình

1. Tắt nguồn trước khi đấu dây.
2. Kết nối cảm biến theo đúng sơ đồ và điện áp mà nhà sản xuất quy định.
3. Kết nối relay/driver với chân GPIO đã chọn.
4. Đảm bảo nguồn máy bơm phù hợp và mạch điều khiển được cách ly đúng cách.
5. Kiểm tra lại dây nối trước khi cấp nguồn.

Trong ví dụ dưới đây, `MOISTURE_THRESHOLD` là ngưỡng độ ẩm minh họa. Cần hiệu chỉnh theo cảm biến thực tế và loại đất tại trang trại.

## 5. Chương trình Python điều khiển máy bơm

Ví dụ minh họa dùng `gpiozero`. Hãy thay `MOISTURE_PIN` và `PUMP_PIN` theo sơ đồ phần cứng thực tế. `MoistureSensor` cung cấp giá trị tương đối từ 0 đến 1; cần xác minh chiều giá trị và hiệu chuẩn cảm biến trước khi vận hành.

```python
from time import sleep
from gpiozero import MoistureSensor, OutputDevice

MOISTURE_PIN = 17
PUMP_PIN = 27
MOISTURE_THRESHOLD = 0.35  # Ngưỡng minh họa, cần hiệu chuẩn

sensor = MoistureSensor(MOISTURE_PIN)
pump = OutputDevice(PUMP_PIN, active_high=True, initial_value=False)

try:
    while True:
        moisture = sensor.value
        print(f"Độ ẩm tương đối: {moisture:.2f}")

        # Ví dụ này giả định giá trị lớn hơn nghĩa là đất ẩm hơn.
        # Hãy xác minh cách đọc của cảm biến trước khi dùng thực tế.
        if moisture < MOISTURE_THRESHOLD:
            pump.on()
            print("Đất khô: bật máy bơm")
        else:
            pump.off()
            print("Độ ẩm đủ: tắt máy bơm")

        sleep(5)

except KeyboardInterrupt:
    print("Đã dừng chương trình.")

finally:
    pump.off()
    pump.close()
    sensor.close()
```

Lưu ý: Một số cảm biến xuất tín hiệu analog và không thể đọc trực tiếp bằng GPIO số của Raspberry Pi nếu không có bộ chuyển đổi ADC. Hãy xác nhận loại cảm biến và cách đấu nối trước khi chạy chương trình.

## 6. So sánh cảm biến cũ và cảm biến mới

| Tiêu chí | Cảm biến loại cũ | Cảm biến loại mới |
|---|---|---|
| Công nghệ | Đầu dò điện trở | Đầu dò điện dung |
| Độ bền | Có thể bị ăn mòn nhanh hơn khi tiếp xúc đất ẩm | Thường chống ăn mòn tốt hơn |
| Bảo trì | Có thể cần thay thế thường xuyên hơn | Thường ít phải bảo trì hơn |
| Chi phí | Thường thấp | Thường cao hơn |
| Lưu ý | Kết quả phụ thuộc đất và tình trạng đầu dò | Vẫn cần hiệu chuẩn theo loại đất |

*Bảng trên là so sánh khái quát; thông số thực tế phụ thuộc từng mẫu sản phẩm.*

## 7. Checklist nghiệm thu

- [ ] Đã xác định đúng loại cảm biến và sơ đồ chân
- [ ] Đã kiểm tra điện áp và nguồn cấp
- [ ] Đã đấu relay/driver đúng thông số của máy bơm
- [ ] Đã cài đặt thư viện `gpiozero`
- [ ] Đã hiệu chuẩn ngưỡng `MOISTURE_THRESHOLD`
- [ ] Đã thử đọc dữ liệu cảm biến khi máy bơm chưa kết nối
- [ ] Đã kiểm tra máy bơm bật/tắt an toàn
- [ ] Đã kiểm tra chương trình tắt máy bơm khi dừng bằng `Ctrl+C`

## 8. Ví dụ ký tự Escape trong Markdown

Thêm dấu gạch chéo ngược `\\` trước ký tự đặc biệt để hiển thị ký tự đó theo nghĩa đen trong Markdown:

- `\\*` hiển thị \*
- `\\#` hiển thị \#

## 9. Kết luận

Sau khi hoàn thành checklist, ghi lại mẫu cảm biến, sơ đồ chân, ngưỡng hiệu chuẩn và kết quả chạy thử. Chỉ vận hành hệ thống tưới tự động khi phần điện, nguồn cấp và cơ chế ngắt máy bơm đã được kiểm tra an toàn.
"""

path = Path("/mnt/data/huong_dan_cam_bien_do_am_trang_trai_lan.md")
path.write_text(content, encoding="utf-8")
print(f"Đã tạo file: {path}")