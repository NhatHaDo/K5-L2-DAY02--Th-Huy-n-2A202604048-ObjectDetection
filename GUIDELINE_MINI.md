# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con chở người cỡ nhỏ: sedan, hatchback, SUV, taxi, xe bán tải | Thân thấp, có nắp ca-pô và cốp tách khỏi khoang khách; taxi vàng/xanh vẫn là `car`; bán tải có cabin nhỏ vẫn là `car` |
| `truck` | Xe tải chở hàng hoặc xe chuyên dụng có cabin riêng tách khỏi thùng/thiết bị phía sau, kể cả đầu kéo | Cabin tách rời thùng hàng, bồn chở nước, cần cẩu cứu hộ; xe chở hàng chất cao vẫn là `truck` |
| `bus` | Xe buýt, xe khách, minibus chở khách theo tuyến | Thân dài, nhiều cửa sổ bên hông xếp thành dãy; xe buýt nối toa và xe buýt hai tầng đều là `bus` |
| `van` | Xe van, minivan dạng hộp chở hàng hoặc chở người | Thân hộp, ca-pô rất ngắn, mui cao hơn `car` nhưng ngắn hơn hẳn `bus`; minivan trắng/bạc cỡ nhỏ là `van` |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: 4 cạnh box chạm điểm ngoài cùng của phần thân xe nhìn thấy được (kể cả gương, cản), không lấy thêm bóng xe, mặt đường hay xe bên cạnh. Mỗi xe một box riêng, không gộp hai xe.
- Vật thể bị che khuất: vẫn gán box quanh phần nhìn thấy nếu xác định được lớp; không vẽ phần bị che theo phỏng đoán. Hai xe chồng nhau thì vẽ hai box riêng.
- Vật thể bị cắt bởi rìa ảnh: box chạm đúng rìa ảnh, chỉ bao phần nằm trong khung; vẫn gán nếu phần nhìn thấy đủ để nhận lớp.
- Vật thể rất nhỏ/mơ hồ: xe ở xa hoặc bị nhòe chuyển động vẫn gán nếu nhận ra được lớp; nếu chỉ còn mảnh nhỏ không xác định chắc lớp thì không gán, ghi lại tên ảnh và hỏi giảng viên.
- Ngoài phạm vi: không gán xe máy, xe đạp, xe ba bánh, người đi bộ, biển báo, xe trong gương/kính hoặc trên bảng quảng cáo.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_014` | Xe cứu hộ của công an (cabin trắng, phía sau có cần cẩu kéo xe) ở giữa ảnh | `truck` | Cabin tách rời phần thiết bị phía sau, là xe tải chuyên dụng chứ không phải xe chở khách; không thuộc `van` vì không có thân hộp liền |
| `drive_014` | Xe đỏ thân hộp, mui cao, cỡ lớn hơn van thường ở góc trên bên phải | Tạm gán `van`, đã ghi lại để hỏi giảng viên | Ca van/minibus mơ hồ; GUIDE yêu cầu không tự chọn mà ghi case và hỏi; thân ngắn, ca-pô ngắn nên tạm nghiêng về `van` |
| `drive_042` | Xe chở hàng chất bao tải cao hơn cabin; xe buýt đỏ bị cắt ở rìa dưới bên trái | Xe chở hàng: `truck`; xe buýt bị cắt: `bus`, box chạm rìa ảnh | Hàng hóa chất cao vẫn là `truck` vì có cabin và thùng chở hàng; xe bị cắt khung vẫn gán theo phần nhìn thấy |

## Cập nhật sau gold QC (rework)

- Không gán xe đỗ dọc lề đường/vỉa hè và dòng xe nhỏ ở xa cuối đường (vùng không nhận rõ từng xe); chỉ gán xe đang lưu thông ở cự ly gần và trung bình. Quy tắc này rút ra từ quy ước của gold eval trong repo, khác với câu "xe đang đỗ vẫn là car" trong README — đã báo coach.
- Minivan/MPV thân cao, ca-pô ngắn, đuôi vuông gán `van`, không gán `car`.
- Thứ tự class ID trong gold (train và `labels/eval`) là car=0, van=1, truck=2, bus=3, khác `data.yaml` (car=0, truck=1, bus=2, van=3). Nhãn train sau rework dùng thứ tự của gold để khớp với tập eval khi train; đã báo coach.

## Cập nhật sau gold QC (rework)

- Không gán xe đỗ dọc lề đường/vỉa hè và dòng xe nhỏ ở xa cuối đường (vùng không nhận rõ từng xe); chỉ gán xe đang lưu thông ở cự ly gần và trung bình. Quy tắc này rút ra từ quy ước của gold eval trong repo, khác với câu "xe đang đỗ vẫn là car" trong README — đã báo coach.
- Minivan/MPV thân cao, ca-pô ngắn, đuôi vuông gán `van`, không gán `car`.
- Thứ tự class ID trong gold (train và `labels/eval`) là car=0, van=1, truck=2, bus=3, khác `data.yaml` (car=0, truck=1, bus=2, van=3). Nhãn train sau rework dùng thứ tự của gold để khớp với tập eval khi train; đã báo coach.
