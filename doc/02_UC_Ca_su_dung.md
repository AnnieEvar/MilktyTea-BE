# UC — Ca sử dụng

> Phiên bản: 0.1 (bản phân tích ban đầu) · 01/11/2026 · Nguồn: Báo cáo lần 1 và ghi chú OOP.

Luồng chính và ngoại lệ được mô tả trong một dòng bảng để đáp ứng mẫu output yêu cầu.


## AU — Tài khoản & phân quyền

| UC ID | Tên ca sử dụng | Tác nhân chính | Mục tiêu | Điều kiện tiên quyết | Luồng chính | Luồng thay thế / Ngoại lệ | Mã yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- | --- |
| UC-01 | Đăng nhập hệ thống | Nhân viên | Truy cập chức năng được giao | Tài khoản đang hoạt động | 1. Mở trang đăng nhập; 2. Nhập thông tin; 3. Máy chủ xác thực; 4. Cấp phiên và quyền thao tác | Sai mật khẩu, bị khóa hoặc đổi quyền → từ chối; phiên cũ bị thu hồi | FR-01, FR-02, FR-03 |
| UC-02 | Quản lý tài khoản và phân công quyền | Quản trị viên | Cập nhật quyền nhân viên | Có quyền quản trị | 1. Chọn nhân viên; 2. Chọn vai trò/chi nhánh; 3. Kiểm tra xung đột; 4. Lưu; 5. Ghi log | Không đủ quyền hoặc chuyển chi nhánh giữa ca → từ chối/đề nghị xử lý ca | FR-02, FR-03, FR-04 |
| UC-03 | Tra cứu nhật ký nhạy cảm | Chủ doanh nghiệp | Truy vết thao tác | Đã đăng nhập với quyền phù hợp | 1. Chọn khoảng thời gian; 2. Lọc người/chi nhánh/đối tượng; 3. Xem kết quả | Không có quyền xem phạm vi yêu cầu → từ chối | FR-04 |

## BR — Chi nhánh, nhân sự & ca bán

| UC ID | Tên ca sử dụng | Tác nhân chính | Mục tiêu | Điều kiện tiên quyết | Luồng chính | Luồng thay thế / Ngoại lệ | Mã yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- | --- |
| UC-04 | Quản lý chi nhánh và vùng giao | Quản trị viên | Cập nhật điểm bán | Có quyền quản trị chi nhánh | 1. Chọn chi nhánh; 2. Nhập thông tin/vùng phục vụ/năng lực; 3. Lưu; 4. Cập nhật trạng thái | Mã trùng hoặc giờ/vùng không hợp lệ → thông báo | FR-05, FR-08 |
| UC-05 | Phân công quầy và chuyển nhân viên | Quản lý chi nhánh | Quản lý hoạt động quầy | Có quyền chi nhánh | 1. Chọn nhân viên và quầy; 2. Giao nhiệm vụ; 3. Lưu lịch sử | Nhân viên có ca đang mở → xử lý ca trước | FR-06, FR-07 |
| UC-06 | Mở ca và đóng ca bán hàng | Thu ngân | Ghi nhận dòng tiền theo ca | Được phân công quầy | 1. Mở ca với tiền đầu ca; 2. Bán hàng; 3. Đếm tiền cuối ca; 4. Ghi giải trình; 5. Đóng ca | Ca đang mở trùng → từ chối; chênh lệch lớn → chờ duyệt | FR-07 |

## CA — Sản phẩm, giá & công thức

| UC ID | Tên ca sử dụng | Tác nhân chính | Mục tiêu | Điều kiện tiên quyết | Luồng chính | Luồng thay thế / Ngoại lệ | Mã yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- | --- |
| UC-07 | Quản lý menu và biến thể món | Quản trị viên | Quản lý sản phẩm | Có quyền quản lý menu | 1. Tạo/sửa sản phẩm; 2. Định nghĩa size và nhóm tùy chọn; 3. Lưu; 4. Bật/tắt bán | Món đang có đơn → không xóa cứng; lựa chọn không hợp lệ → từ chối | FR-09, FR-10 |
| UC-08 | Duyệt phiên bản giá và công thức | Chủ doanh nghiệp | Thống nhất giá vốn và định lượng | Có phiên bản nháp | 1. Chọn giá/công thức; 2. Kiểm tra phạm vi và hiệu lực; 3. Duyệt; 4. Kích hoạt | Xung đột thời gian hoặc thiếu định mức → không duyệt | FR-11, FR-12, FR-14 |
| UC-09 | Tạm khóa món tại một chi nhánh | Quản lý chi nhánh | Ngăn bán món không khả dụng | Có quyền chi nhánh | 1. Xem mặt hàng; 2. Chọn khóa; 3. Nêu lý do; 4. Cập nhật menu POS/online | Món đã được xác nhận → tiếp tục xử lý theo đơn hiện hữu | FR-13, FR-25 |

## IN — Kho, sơ chế & cung ứng

| UC ID | Tên ca sử dụng | Tác nhân chính | Mục tiêu | Điều kiện tiên quyết | Luồng chính | Luồng thay thế / Ngoại lệ | Mã yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- | --- |
| UC-10 | Quản lý mặt hàng kho và định mức an toàn | Nhân viên kho | Thiết lập theo dõi tồn | Đã đăng nhập đúng chi nhánh | 1. Khai báo mặt hàng/đơn vị; 2. Thiết lập ngưỡng từng chi nhánh; 3. Lưu | Đơn vị quy đổi sai hoặc ngưỡng âm → từ chối | FR-15, FR-18, FR-25 |
| UC-11 | Lập phiếu mua và nhận nguyên liệu | Nhân viên kho | Bổ sung hàng hóa theo lô | Nhà cung cấp và phiếu mua hợp lệ | 1. Lập phiếu mua; 2. Nhận hàng; 3. Kiểm số lượng/chất lượng/hạn; 4. Nhập phần đạt; 5. Ghi chênh lệch | Lô không đạt hoặc quá hạn → cách ly/từ chối; không cộng tồn | FR-16, FR-17 |
| UC-12 | Tạo mẻ sơ chế và ghi nhận thành phẩm | Nhân viên kho | Theo dõi lượng sơ chế | Có công thức và đủ tồn đầu vào | 1. Chọn công thức sơ chế; 2. Ghi lô đầu vào; 3. Nhập sản lượng thực; 4. Tạo lô thành phẩm; 5. Ghi hao hụt | Thiếu đầu vào/hạn dùng không hợp lệ → không hoàn tất mẻ | FR-20, FR-21, FR-22 |
| UC-13 | Giữ và xuất nguyên liệu cho đơn | Hệ thống | Chặn vượt bán | Đơn đang xác nhận | 1. Phân rã công thức; 2. Lọc lô hợp lệ FEFO; 3. Giữ tồn; 4. Khi hoàn tất ghi xuất thực; 5. Giải phóng giữ | Không đủ nguyên liệu → không nhận đơn hoặc đề xuất đổi lựa chọn | FR-18, FR-19, FR-20 |
| UC-14 | Yêu cầu chuyển nguyên liệu giữa chi nhánh | Quản lý chi nhánh | Giải quyết thiếu hàng | Có cảnh báo thiếu | 1. Tạo yêu cầu; 2. Chi nhánh xuất duyệt; 3. Xuất và ghi đang chuyển; 4. Chi nhánh nhận xác nhận; 5. Hoàn tất | Số lượng vượt tồn an toàn kho xuất → từ chối; nhận thiếu → biên bản | FR-23, FR-25 |
| UC-15 | Kiểm kê và điều chỉnh hao hụt | Nhân viên kho | Đối soát tồn thực tế | Có quyền kho, không có phiên kiểm kê trùng | 1. Mở kiểm kê; 2. Đếm lô; 3. So lệch; 4. Ghi nguyên nhân; 5. Duyệt; 6. Ghi chuyển động kho | Vượt ngưỡng → chờ cấp có thẩm quyền; không sửa số dư tùy ý | FR-22, FR-24 |
| UC-16 | Xem cảnh báo và gợi ý bổ sung | Quản lý chi nhánh | Phòng tránh hết hàng | Có dữ liệu tồn và hạn dùng | 1. Xem cảnh báo; 2. Phân loại thiếu/cận hạn; 3. Chọn đề nghị mua/chuyển; 4. Duyệt thủ công | Không đủ dữ liệu nhu cầu → chỉ hiển thị số liệu hiện có | FR-25, FR-26 |

## OR — Đơn hàng, đặt online & pha chế

| UC ID | Tên ca sử dụng | Tác nhân chính | Mục tiêu | Điều kiện tiên quyết | Luồng chính | Luồng thay thế / Ngoại lệ | Mã yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- | --- |
| UC-17 | Lập đơn bán tại quầy | Thu ngân | Bán tại chỗ/mang đi | Ca đang mở | 1. Chọn món/size/tùy chọn; 2. Kiểm tra giá/tồn; 3. Áp ưu đãi; 4. Thanh toán; 5. Tạo phiếu pha | Không đủ topping → gợi ý loại bỏ và tính lại giá; thanh toán lỗi → không pha | FR-27, FR-29, FR-31, FR-32 |
| UC-18 | Đặt hàng online và chọn chi nhánh | Khách hàng | Đặt đồ uống online | Chi nhánh có vùng phục vụ và nhận đơn | 1. Chọn địa chỉ/vùng; 2. Chọn chi nhánh; 3. Chọn món/tùy chọn; 4. Nhập giao nhận; 5. Đặt hàng; 6. Chờ xác nhận | Chi nhánh quá tải/thiếu hàng → gợi ý chi nhánh khác; quá hạn → xử lý theo chính sách | FR-28, FR-29, FR-33, FR-53 |
| UC-19 | Xác nhận đơn và phân phối phiếu pha | Thu ngân | Tiếp nhận đơn đa kênh | Đơn thanh toán hợp lệ hoặc COD được phép | 1. Xem hàng đợi; 2. Xác nhận trong thời hạn; 3. Giữ tồn; 4. Tạo một phiếu pha; 5. Giao pha chế | Không nhận kịp → cảnh báo/hủy; không đủ nguyên liệu → từ chối | FR-29, FR-30, FR-31 |
| UC-20 | Hoàn tất pha chế và giao món | Thu ngân | Kết thúc đơn | Phiếu pha đã tạo | 1. Pha chế nhận phiếu; 2. Xác nhận sẵn sàng; 3. Bàn giao; 4. Đánh dấu hoàn tất; 5. Trừ kho/cộng điểm | Hỏng món → ghi hao hụt và làm lại; thiếu thành phần → UC thay đổi | FR-30, FR-31, FR-33 |
| UC-21 | Đổi/bỏ topping hoặc hủy món | Thu ngân | Xử lý không giao đủ theo đơn | Đơn chưa hoàn tất hoặc phát sinh khiếu nại | 1. Nhận yêu cầu; 2. Xác nhận với khách; 3. Cập nhật chi tiết/giảm giá; 4. Hoàn phần tiền chênh; 5. Cập nhật kho | Khách không đồng ý → hủy/hoàn theo chính sách; hành động nhạy cảm cần duyệt | FR-32, FR-35, FR-38 |
| UC-22 | Mua số lượng lớn với chiết khấu | Khách hàng/Thu ngân | Mua nhiều ly mà không lập đơn sỉ | Có quy tắc giảm theo số lượng | 1. Chọn nhiều ly; 2. Tính ngưỡng giảm; 3. Kiểm tra tồn/công suất; 4. Xác nhận và thanh toán | Không đủ công suất → đề nghị thay chi nhánh/thời gian hoặc giảm số lượng | FR-34, FR-45 |
| UC-23 | Hủy đơn và cập nhật các bút toán | Quản lý chi nhánh | Xử lý đơn không thể hoàn thành | Có quyền duyệt khi cần | 1. Chọn đơn; 2. Ghi lý do; 3. Hủy; 4. Giải phóng giữ; 5. Hoàn tiền và đảo điểm liên quan | Đơn đã hoàn tất → áp quy trình hoàn tiền/khiếu nại thay vì xóa | FR-35, FR-38, FR-42 |

## PA — Thanh toán, hoàn tiền & đối soát

| UC ID | Tên ca sử dụng | Tác nhân chính | Mục tiêu | Điều kiện tiên quyết | Luồng chính | Luồng thay thế / Ngoại lệ | Mã yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- | --- |
| UC-24 | Thu tiền và ghi nhận kết quả | Thu ngân | Xác nhận thanh toán đơn | Đơn hợp lệ chưa thu đủ | 1. Chọn phương thức; 2. Gửi giao dịch mock hoặc nhập tiền mặt; 3. Nhận kết quả; 4. Xác nhận đơn | Thất bại → chờ thanh toán; không xác định → chờ xác minh, không tự xác nhận | FR-36, FR-37 |
| UC-25 | Hoàn tiền có phê duyệt | Quản lý chi nhánh | Hoàn khoản đã thu | Đơn đủ điều kiện và có thanh toán | 1. Kiểm tra số đã thu; 2. Nhập lý do/số tiền; 3. Duyệt; 4. Ghi hoàn; 5. Đồng bộ đảo điểm nếu có | Vượt tiền còn lại hoặc yêu cầu trùng → từ chối | FR-38 |
| UC-26 | Đối soát thanh toán và biên bản ca | Quản lý chi nhánh | Kiểm tra thu chi | Có giao dịch và/hoặc ca đóng | 1. Xem giao dịch theo đơn/ca; 2. Đối chiếu tiền; 3. Ghi sai lệch; 4. Duyệt biên bản | Giao dịch chưa xác minh → chưa ghi doanh thu đã thanh toán | FR-39, FR-07 |

## CR — Khách hàng, thành viên & ưu đãi

| UC ID | Tên ca sử dụng | Tác nhân chính | Mục tiêu | Điều kiện tiên quyết | Luồng chính | Luồng thay thế / Ngoại lệ | Mã yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- | --- |
| UC-27 | Đăng ký và cập nhật hồ sơ thành viên | Khách hàng | Tham gia chương trình thành viên | Có số điện thoại xác minh | 1. Cung cấp số điện thoại; 2. Xác minh; 3. Đồng ý xử lý dữ liệu; 4. Tạo hồ sơ; 5. Hiển thị hạng Member | Số trùng → liên kết hồ sơ hợp lệ; không đồng ý → vẫn mua như khách lẻ | FR-40, FR-41 |
| UC-28 | Cộng/trừ điểm từ đơn hoàn tất hoặc hoàn tiền | Hệ thống | Duy trì sổ điểm chính xác | Có đơn đủ điều kiện | 1. Nhận sự kiện hoàn tất/hoàn; 2. Tính điểm; 3. Ghi giao dịch sổ điểm; 4. Cập nhật số dư hiển thị | Sự kiện lặp → không ghi lần hai; không đủ điểm khi đổi → từ chối | FR-42 |
| UC-29 | Xét lại hạng thành viên | Hệ thống/Quản trị viên | Xác định hạng theo năm | Có doanh số đủ dữ liệu | 1. Tổng hợp 12 tháng; 2. Tính hạng theo ngưỡng cấu hình; 3. Lưu lịch sử; 4. Áp quyền lợi từ kỳ mới | Thiếu lịch sử → báo không đủ dữ liệu, không suy diễn hạng | FR-43 |
| UC-30 | Tạo khuyến mãi theo chi nhánh và kênh | Chủ doanh nghiệp | Quản lý ưu đãi | Có quyền tạo chính sách | 1. Nhập điều kiện, thời gian, phạm vi và ngân sách; 2. Kiểm tra; 3. Kích hoạt | Cấu hình chồng lấn không hợp lệ hoặc hết ngân sách → ngừng áp | FR-44, FR-45 |
| UC-31 | Áp voucher và khuyến mãi cho đơn | Khách hàng/Thu ngân | Giảm giá theo quyền lợi hợp lệ | Đơn còn hiệu lực tính giá | 1. Chọn ưu đãi; 2. Kiểm tra hạng, chi nhánh, kênh, số lượng; 3. Tính tiền sau giảm | Không hợp lệ, hết lượt, cấm cộng dồn → không áp và thông báo | FR-44, FR-45, FR-46 |
| UC-32 | Tra cứu lịch sử mua và phản hồi | Khách hàng | Theo dõi giao dịch | Có đơn liên kết quyền sở hữu | 1. Mở tài khoản/tra cứu đơn; 2. Xem lịch sử; 3. Gửi khiếu nại kèm mã đơn và bằng chứng; 4. Theo dõi xử lý | Truy cập đơn người khác → từ chối | FR-47 |

## SY — Gián đoạn vận hành & đồng bộ

| UC ID | Tên ca sử dụng | Tác nhân chính | Mục tiêu | Điều kiện tiên quyết | Luồng chính | Luồng thay thế / Ngoại lệ | Mã yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- | --- |
| UC-33 | Bán hàng tại quầy khi không có Internet | Thu ngân | Duy trì bán trong điều kiện an toàn | POS còn dùng được; đã có dữ liệu menu ngoại tuyến | 1. POS báo offline; 2. Tạo đơn mã riêng; 3. Ghi thanh toán offline theo chính sách; 4. Lưu bền hàng đợi | Chưa đủ điều kiện an toàn → dừng vận hành; ưu đãi/điểm cần xác minh sau | FR-48, FR-49 |
| UC-34 | Báo sự cố và thay đổi nhận đơn của một chi nhánh | Quản lý chi nhánh | Giảm gián đoạn nhận đơn | Có sự cố mạng/điện/quá tải | 1. Ghi nhận sự cố; 2. Đánh giá kết nối dự phòng và an toàn; 3. Chỉ tạm dừng online nếu không đáp ứng; 4. Gợi ý nơi khác | Chi nhánh vẫn đáp ứng → tiếp tục online; sự cố khác chi nhánh không bị ảnh hưởng | FR-49, FR-50 |
| UC-35 | Đồng bộ và đối soát đơn offline | Thu ngân/Quản lý chi nhánh | Khôi phục sổ giao dịch chính xác | Có mạng trở lại và hàng đợi cục bộ | 1. Gửi các đơn theo thứ tự; 2. Máy chủ đối chiếu key; 3. ACK từng đơn; 4. Xử lý lỗi/chưa xác minh; 5. Đối soát tồn/doanh thu | Trùng key → trả kết quả cũ; xung đột tồn → ghi sai lệch để duyệt, không xóa đơn | FR-51, FR-52 |
| UC-36 | Xử lý đơn online quá hạn chưa được nhận | Hệ thống/Quản lý chi nhánh | Không để đơn chờ vô thời hạn | Có đơn chờ xác nhận | 1. Theo dõi thời hạn; 2. Cảnh báo; 3. Chuyển hướng hoặc hủy theo chính sách; 4. Hoàn nếu đã thu | Chi nhánh phản hồi kịp → giữ đơn; giao dịch chưa xác minh → kiểm tra trước hoàn | FR-53 |

## RP — Báo cáo & đánh giá chi nhánh

| UC ID | Tên ca sử dụng | Tác nhân chính | Mục tiêu | Điều kiện tiên quyết | Luồng chính | Luồng thay thế / Ngoại lệ | Mã yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- | --- |
| UC-37 | Xem báo cáo vận hành toàn chuỗi | Chủ doanh nghiệp | Đánh giá kết quả kinh doanh | Có dữ liệu giao dịch | 1. Chọn thời gian/chi nhánh; 2. Xem doanh thu, đơn, AOV; 3. So sánh kênh | Dữ liệu offline chưa đồng bộ → gắn nhãn chưa đầy đủ | FR-54 |
| UC-38 | Xem báo cáo kho và hiệu quả món | Quản lý chi nhánh | Kiểm soát tổn thất và bổ sung | Có dữ liệu lô, xuất kho, bán hàng | 1. Lọc chi nhánh và kỳ; 2. Xem cận hạn/hao hụt; 3. Xem món bán chạy; 4. Xuất báo cáo | Không đủ số liệu tồn → báo trạng thái đối soát | FR-55, FR-56 |
| UC-39 | Đánh giá chi nhánh và cảnh báo hiệu quả thấp | Chủ doanh nghiệp | Ra quyết định quản trị | Có chi phí kỳ và số liệu bán | 1. Nhập/đối chiếu chi phí; 2. Xem biên đóng góp ước tính; 3. So xu hướng 4 tuần/3 tháng; 4. Đưa ra đánh giá | Không đủ chi phí → không kết luận hòa vốn; hệ thống không tự đóng chi nhánh | FR-57, FR-58 |
| UC-40 | Xuất báo cáo có phân quyền | Quản lý chi nhánh | Lưu trữ số liệu | Có quyền xem dữ liệu | 1. Chọn kỳ/chi nhánh; 2. Xuất CSV; 3. Lưu thông tin truy vấn/thời điểm | Vượt phạm vi chi nhánh → từ chối | FR-59 |

