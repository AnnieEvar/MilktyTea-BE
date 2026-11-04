# FR — Yêu cầu chức năng

> Phiên bản: 0.1 (bản phân tích ban đầu) · 01/11/2026 · Nguồn: Báo cáo lần 1 và ghi chú OOP.

Mỗi FR được gắn mã toàn cục và chia theo module.


## AU — Tài khoản & phân quyền

| Mã yêu cầu | Yêu cầu | Mức ưu tiên | Chỉ số / Tiêu chí chấp nhận | Ghi chú |
| --- | --- | --- | --- | --- |
| FR-01 | Nhân viên đăng nhập, đăng xuất tài khoản hệ thống | Bắt buộc | Thông tin hợp lệ tạo phiên; tài khoản bị khóa/thu hồi quyền không được truy cập. | Báo cáo chương II, mục 13. |
| FR-02 | Phân quyền theo vai trò và phạm vi chi nhánh | Bắt buộc | Mọi API nghiệp vụ kiểm tra vai trò và chi nhánh; quản lý không xem/sửa chi nhánh khác. | Nhân viên pha chế chưa bắt buộc có tài khoản riêng. |
| FR-03 | Quản trị viên quản lý trạng thái tài khoản nhân viên | Bắt buộc | Khóa/mở khóa và vô hiệu hóa phiên hiện hành khi thôi việc hoặc đổi quyền. | Không tự mở rộng sang tuyển dụng/bảng lương. |
| FR-04 | Ghi nhật ký thao tác nhạy cảm | Bắt buộc | Lưu người thực hiện, đối tượng, thời gian, lý do và chi nhánh nếu có; không xóa nhật ký. | Áp dụng duyệt giá, kho, hoàn tiền và điểm. |

## BR — Chi nhánh, nhân sự & ca bán

| Mã yêu cầu | Yêu cầu | Mức ưu tiên | Chỉ số / Tiêu chí chấp nhận | Ghi chú |
| --- | --- | --- | --- | --- |
| FR-05 | Quản lý danh sách và trạng thái chi nhánh | Bắt buộc | Thêm/sửa/tạm ngừng; mỗi chi nhánh có mã duy nhất, địa chỉ, giờ mở cửa và trạng thái. | Một doanh nghiệp, 12 chi nhánh mô phỏng. |
| FR-06 | Quản lý quầy POS và phân công nhân viên | Bắt buộc | POS thuộc một chi nhánh; lịch sử điều chuyển nhân viên được giữ lại. | Phân quyền theo chi nhánh. |
| FR-07 | Thu ngân mở/đóng ca và đối chiếu tiền mặt | Bắt buộc | Một quầy chỉ có một ca đang mở; chênh lệch cuối ca có giải trình và quyền phê duyệt. | Không phải module kế toán tổng hợp. |
| FR-08 | Cấu hình khu vực phục vụ và năng lực nhận đơn online | Bắt buộc | Mỗi chi nhánh có phạm vi giao; có thể tạm giới hạn số đơn nhận theo năng lực. | Theo góp ý OOP về chọn chi nhánh và quá tải. |

## CA — Sản phẩm, giá & công thức

| Mã yêu cầu | Yêu cầu | Mức ưu tiên | Chỉ số / Tiêu chí chấp nhận | Ghi chú |
| --- | --- | --- | --- | --- |
| FR-09 | Quản lý nhóm món, sản phẩm và trạng thái bán | Bắt buộc | Món có mã duy nhất; ngừng bán không làm thay đổi chi tiết đơn quá khứ. | Menu lõi toàn chuỗi. |
| FR-10 | Quản lý size, mức đường, đá và topping | Bắt buộc | Khách chọn tổ hợp hợp lệ; hệ thống tính phụ phí và định lượng tương ứng. | OOP: size/đường/đá/topping phải tính được nguyên liệu. |
| FR-11 | Quản lý bảng giá theo thời gian và phạm vi | Bắt buộc | Giá theo chuỗi/chi nhánh/kênh; không có phiên bản giá xung đột cùng phạm vi-thời điểm. | Chi nhánh không tự ý sửa giá chuẩn. |
| FR-12 | Quản lý công thức định lượng có phiên bản, phê duyệt | Bắt buộc | Công thức gắn size và lựa chọn; chỉ phiên bản duyệt, hiệu lực mới được bán. | Công thức lịch sử không ghi đè. |
| FR-13 | Kiểm soát bán món theo chi nhánh | Bắt buộc | Chi nhánh có thể khóa món riêng; đơn mới không chọn món thiếu khả dụng. | Không xóa món hoặc sửa đơn đã chốt. |
| FR-14 | Tính lượng nguyên liệu cần theo món và tùy chọn | Bắt buộc | Tính được định mức cho số lượng ly, size, topping, đường và đá, kèm quy đổi đơn vị. | Không suy diễn lượng pha thực tế bằng định mức tuyệt đối. |

## IN — Kho, sơ chế & cung ứng

| Mã yêu cầu | Yêu cầu | Mức ưu tiên | Chỉ số / Tiêu chí chấp nhận | Ghi chú |
| --- | --- | --- | --- | --- |
| FR-15 | Quản lý nguyên liệu và vật tư đóng gói | Bắt buộc | Mỗi mặt hàng có đơn vị chuẩn, loại (thô/đã sơ chế/bao bì), mức tồn an toàn. | OOP: tách lượng đã chế biến khỏi nguyên liệu thô. |
| FR-16 | Quản lý nhà cung cấp và phiếu mua | Bắt buộc | Phiếu mua chứa nhà cung cấp, nguyên liệu, số lượng, giá và dự kiến giao. | Công nợ nhà cung cấp đầy đủ là ngoài MVP. |
| FR-17 | Nhập kho theo lô, hạn dùng và chất lượng | Bắt buộc | Chỉ nhập phần đạt; lô hết hạn/cách ly không được tính khả dụng. | Theo báo cáo mục 8. |
| FR-18 | Theo dõi tồn thực tế, đã giữ, khóa và khả dụng | Bắt buộc | Tồn khả dụng = tồn vật lý hợp lệ − đã giữ − đang khóa; bán online trừ thêm ngưỡng dự phòng; không âm. | Có ngưỡng riêng hạn chế nhận online. |
| FR-19 | Giữ nguyên liệu có kiểm soát cho đơn đã xác nhận | Bắt buộc | Không giữ vượt tồn khả dụng; giải phóng giữ khi hủy/hết hạn xác nhận. | Chống oversell giữa POS và online. |
| FR-20 | Trừ kho theo lượng sử dụng thực tế và định mức | Bắt buộc | Chứng từ xuất/trừ kho được truy vết tới đơn; chênh lệch định mức được ghi riêng. | OOP: hao hụt không được biến mất khỏi dữ liệu. |
| FR-21 | Ghi nhận mẻ sơ chế, sản lượng thực tế, hạn dùng | Bắt buộc | Mẻ ghi đầu vào, lượng dự kiến/thực thu, lô thành phẩm và thời hạn sử dụng. | Không mặc định xây nhà máy/kho vùng. |
| FR-22 | Ghi nhận hỏng, dư, thiếu và hao hụt đã sơ chế | Bắt buộc | Có nguyên nhân, người tạo, lượng và ảnh hưởng tồn theo chứng từ. | Bao gồm hết hạn, đổ vỡ, pha lỗi, sai định lượng. |
| FR-23 | Chuyển nguyên liệu giữa các chi nhánh | Bắt buộc | Có yêu cầu/duyệt/xuất/nhận; giữ thông tin lô; chỉ tăng tồn kho nhận khi xác nhận thực nhận. | Chuyển khẩn cấp và có phê duyệt. |
| FR-24 | Kiểm kê và xử lý sai lệch tồn kho | Bắt buộc | Một kho một phiên đang mở; điều chỉnh có lý do và duyệt nếu vượt ngưỡng. | Không chỉnh số dư trực tiếp. |
| FR-25 | Cảnh báo tồn thấp, cận hạn và mặt hàng thiếu cho online | Bắt buộc | Cảnh báo theo lô và chi nhánh; có thể ẩn bán online trước khi hết thực tế. | OOP: dự phòng cho đơn tại quầy. |
| FR-26 | Gợi ý nhập hoặc chuyển nguyên liệu | Nên có | Dựa trên tồn an toàn, nhu cầu gần hạn, lô cận hạn; người quản lý quyết định. | Không dự báo AI. |

## OR — Đơn hàng, đặt online & pha chế

| Mã yêu cầu | Yêu cầu | Mức ưu tiên | Chỉ số / Tiêu chí chấp nhận | Ghi chú |
| --- | --- | --- | --- | --- |
| FR-27 | Thu ngân lập đơn tại quầy, tại chỗ hoặc mang đi | Bắt buộc | Tạo đơn gắn chi nhánh, ca, món, tùy chọn và giá hiệu lực; ngăn tạo trùng. | Luồng POS báo cáo mục 3. |
| FR-28 | Khách đặt online, chọn chi nhánh phục vụ | Bắt buộc | Hiển thị chi nhánh phù hợp khu vực, trạng thái nhận đơn và món khả dụng. | OOP: khách được chọn trước; có thể gợi ý chi nhánh khác. |
| FR-29 | Kiểm tra đồng thời tồn nguyên liệu và công suất nhận đơn | Bắt buộc | Chỉ xác nhận nếu đủ tồn dùng được và còn năng lực; từ chối có lý do rõ ràng. | Ưu tiên bảo toàn phục vụ tại quầy. |
| FR-30 | Quản lý vòng đời và lịch sử trạng thái đơn | Bắt buộc | Chờ thanh toán/chờ xác nhận/đang pha/sẵn sàng/hoàn tất/hủy; có lịch sử thời gian/người thao tác. | Không xóa đơn đã phát sinh. |
| FR-31 | Lập phiếu và hàng đợi pha chế chung theo chi nhánh | Bắt buộc | POS và online cùng hàng đợi; không in/pha trùng; xác định thứ tự và độ ưu tiên. | Nhân viên pha chế nhận phiếu hoặc xem hàng đợi. |
| FR-32 | Xử lý thiếu topping hoặc món sau xác nhận | Bắt buộc | Cho phép thay thế hoặc bỏ lựa chọn nếu khách đồng ý; cập nhật tiền và tồn, hoàn phần chênh lệch nếu cần. | OOP: không được thu tiền cho topping không giao. |
| FR-33 | Theo dõi giao nhận và hoàn tất đơn online | Bắt buộc | Lưu thông tin nhận hàng, địa chỉ, trạng thái bàn giao; cập nhật khi hoàn tất. | Đối tác giao hàng mô phỏng. |
| FR-34 | Áp chiết khấu khi đặt số lượng lớn trong đơn bán lẻ | Bắt buộc | Đủ ngưỡng số lượng/giá trị thì áp quy tắc ưu đãi; vẫn dùng quy trình đơn bán lẻ. | Thay cho nghiệp vụ bán sỉ riêng theo OOP. |
| FR-35 | Xử lý hủy/đổi món và giữ nguyên lịch sử đơn | Bắt buộc | Hủy/đổi phải có lý do và quyền hạn; hoàn giữ kho/điểm/tiền phù hợp. | Các khoản đã thu xử lý qua Payments. |

## PA — Thanh toán, hoàn tiền & đối soát

| Mã yêu cầu | Yêu cầu | Mức ưu tiên | Chỉ số / Tiêu chí chấp nhận | Ghi chú |
| --- | --- | --- | --- | --- |
| FR-36 | Ghi nhận thanh toán tiền mặt và phương thức mô phỏng | Bắt buộc | Mỗi giao dịch có mã và trạng thái; chưa thanh toán không chuyển sang pha theo chính sách. | Không tích hợp cổng thanh toán thực. |
| FR-37 | Chống ghi nhận thanh toán hai lần | Bắt buộc | Mã giao dịch ngoài duy nhất khi có; yêu cầu lặp không tạo thêm khoản thu. | Bắt buộc cho tính nhất quán. |
| FR-38 | Hoàn tiền toàn phần hoặc một phần có phê duyệt | Bắt buộc | Không hoàn vượt số đã thu ròng; lưu lý do, người duyệt và giao dịch hoàn. | Hỗ trợ thiếu topping và khiếu nại. |
| FR-39 | Đối soát thanh toán theo đơn và ca | Bắt buộc | Phân biệt thành công, thất bại, chưa xác định; chênh lệch tiền mặt có giải trình. | Đối tác mock; nâng cao tùy thời gian. |

## CR — Khách hàng, thành viên & ưu đãi

| Mã yêu cầu | Yêu cầu | Mức ưu tiên | Chỉ số / Tiêu chí chấp nhận | Ghi chú |
| --- | --- | --- | --- | --- |
| FR-40 | Quản lý hồ sơ khách hàng và đồng ý sử dụng dữ liệu | Bắt buộc | Khách không cần thành viên để mua; hồ sơ thành viên lưu số điện thoại, sự đồng ý và tùy chọn marketing. | Báo cáo mục 7. |
| FR-41 | Đăng ký thành viên bằng số điện thoại xác minh | Bắt buộc | Số điện thoại thành viên duy nhất; chỉ thành viên hợp lệ có tài khoản điểm. | Không mặc định đăng ký email như BandUp. |
| FR-42 | Quản lý sổ điểm cộng/trừ/thu hồi | Bắt buộc | Cộng một lần sau hoàn tất; hoàn tiền thu hồi phần điểm; không sửa số dư trực tiếp. | Sổ cái giao dịch nguồn. |
| FR-43 | Quản lý hạng Member, Silver, Gold, VIP | Bắt buộc | Xét hạng chu kỳ 12 tháng; giữ lịch sử; áp quyền lợi theo thời điểm giao dịch. | Không tự đặt ngưỡng cụ thể khi tài liệu chưa cho. |
| FR-44 | Quản lý chương trình khuyến mãi theo chi nhánh/kênh | Bắt buộc | Khuyến mãi có thời hạn, điều kiện và phạm vi; không áp sai chi nhánh. | OOP: khác nhau theo cửa hàng. |
| FR-45 | Kiểm soát cộng dồn và áp dụng ưu đãi số lượng lớn | Bắt buộc | Chỉ áp ưu đãi đủ điều kiện; các khuyến mãi cấm cộng dồn không kết hợp. | Đơn bán lẻ số lượng lớn, không quy trình sỉ. |
| FR-46 | Quản lý phát hành/sử dụng voucher | Nên có | Voucher có hạn, lượt dùng, chi nhánh/phạm vi; một lần dùng không bị ghi lặp. | Mục nên có theo báo cáo. |
| FR-47 | Theo dõi lịch sử mua và phản hồi khách hàng | Nên có | Khách/nhân viên tra cứu lịch sử; phản hồi có mã đơn, nội dung, bằng chứng và kết quả xử lý. | Không xây hệ thống CSKH chuyên sâu. |

## SY — Gián đoạn vận hành & đồng bộ

| Mã yêu cầu | Yêu cầu | Mức ưu tiên | Chỉ số / Tiêu chí chấp nhận | Ghi chú |
| --- | --- | --- | --- | --- |
| FR-48 | POS tạo và lưu đơn tại thiết bị khi mất kết nối | Bắt buộc | Tải lại ứng dụng vẫn còn hàng đợi đơn chưa được máy chủ xác nhận. | Lưu bền trên thiết bị; yêu cầu trình duyệt tương thích. |
| FR-49 | Duy trì vận hành chi nhánh khi mất điện có phương án thay thế | Bắt buộc | Không tự khóa online chỉ vì mất điện; dựa trạng thái mạng, thiết bị, an toàn và năng lực thực tế. | Ưu tiên phản biện OOP; chỉ ảnh hưởng chi nhánh gặp sự cố. |
| FR-50 | Cho quản lý tạm ngừng/khôi phục nhận online theo chi nhánh | Bắt buộc | Tạm ngừng một chi nhánh không ảnh hưởng chi nhánh khác; khách được gợi ý thay thế. | Có thể kích hoạt khi không thể xác nhận đơn. |
| FR-51 | Đồng bộ đơn offline chống trùng và bảo toàn thứ tự | Bắt buộc | Mỗi client_order_key chỉ tạo một đơn server; bản ghi chỉ xóa khỏi queue sau ACK. | Đối soát lại khi có mạng. |
| FR-52 | Đối soát đơn thanh toán chưa xác minh sau sự cố | Bắt buộc | Không tự xác nhận thanh toán QR/chuyển khoản chưa kiểm chứng; quản lý ghi nhận kết quả có chứng cứ. | Theo báo cáo tình huống mạng/điện. |
| FR-53 | Xử lý đơn online không được xác nhận đúng hạn | Bắt buộc | Quá ngưỡng cấu hình thì cảnh báo/chuyển hoặc hủy có hoàn tiền phù hợp; tránh nhận ngầm. | Báo cáo có ví dụ 5 phút; để cấu hình. |

## RP — Báo cáo & đánh giá chi nhánh

| Mã yêu cầu | Yêu cầu | Mức ưu tiên | Chỉ số / Tiêu chí chấp nhận | Ghi chú |
| --- | --- | --- | --- | --- |
| FR-54 | Xem báo cáo doanh thu, số đơn, AOV theo chi nhánh | Bắt buộc | Lọc theo thời gian/kênh/chi nhánh; loại trừ hoặc điều chỉnh hủy/hoàn tiền đúng quy tắc. | Chỉ xem chi nhánh được phân quyền. |
| FR-55 | Xem tồn kho, nguyên liệu hao hụt và cận hạn | Bắt buộc | Đối chiếu theo lô, chi nhánh và nguyên nhân điều chỉnh. | Hỗ trợ lập kế hoạch phân bổ. |
| FR-56 | Theo dõi sản phẩm bán chạy, nguyên liệu tiêu hao | Bắt buộc | Tổng hợp số ly và lượng tiêu hao định mức/thực tế theo kỳ. | OOP: đánh giá món và phân bổ đầu ngày. |
| FR-57 | Đánh giá hiệu quả chi nhánh với chi phí kỳ | Bắt buộc | Hiển thị doanh thu thuần, chi phí kỳ, biên đóng góp ước tính; phân biệt số liệu nhập tay và giao dịch. | Không phải kế toán chuẩn mực. |
| FR-58 | Cảnh báo chi nhánh hiệu quả thấp qua nhiều kỳ | Nên có | Cảnh báo theo ngưỡng cấu hình và kỳ liên tiếp; không tự động quyết định đóng cửa. | Báo cáo trung bình trượt 4 tuần và 3 tháng. |
| FR-59 | Xuất báo cáo và chứng từ dữ liệu | Bắt buộc | Xuất theo quyền; báo cáo lịch sử giữ nguyên nguồn và thời điểm kết xuất. | Không xuất hóa đơn điện tử thật. |

