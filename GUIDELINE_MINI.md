# Mini Guideline: Gán nhãn ADAS 4 Lớp

## 1. Định nghĩa 4 Lớp
- **0 - car:** Các loại xe con 4–7 chỗ, sedan, hatchback, SUV, crossover, xe taxi.
- **1 - truck:** Các loại xe tải chở hàng, xe đầu kéo, xe ben, xe bồn.
- **2 - bus:** Xe buýt công cộng, xe khách du lịch từ 16 chỗ trở lên.
- **3 - van:** Xe chở hàng/chở khách dạng hộp (minivan, xe van chở bưu kiện, cứu thương).

## 2. Quy tắc vẽ Bounding Box
- **Độ khít:** Box phải bám sát mép ngoài cùng của thân xe (cản trước, đuôi xe, nóc mui, mép ngoài lốp xe).
- **Vật thể bị che khuất (Occlusion):** Chỉ vẽ box ôm trọn phần thân xe nhìn thấy được. Bỏ qua các xe bị che khuất trên 80%.
- **Cắt mép ảnh (Truncation):** Vẽ box chạm đúng mép viền ảnh cho phần xe hiển thị trong khung hình.

## 3. Ba trường hợp mơ hồ và quyết định
1. **Xe bán tải (Pickup):** Phân loại vào `0: car` nếu là dòng bán tải cá nhân/gia đình.
2. **Xe đỗ bên lề đường và xe ngược chiều:** Vẫn gắn nhãn đầy đủ theo đúng class nếu nhận diện rõ hình khối.
3. **Xe ở khoảng cách quá xa:** Những xe nhỏ hơn 15x15 pixel hoặc nằm sát đường chân trời không thể nhận rõ loại phương tiện sẽ được bỏ qua để tránh nhiễu mô hình.