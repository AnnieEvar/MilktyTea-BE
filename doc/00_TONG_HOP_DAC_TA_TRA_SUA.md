# Bộ đặc tả hệ thống quản lý chuỗi cửa hàng trà sữa

> Bản đề xuất v0.1 — 01/11/2026. Đây là đặc tả được suy ra từ báo cáo và ghi chú OOP, chưa phải tài liệu được giảng viên phê duyệt.

## Quy ước và quyết định phạm vi

**Nguồn nghiệp vụ:** *Đã-sửa-phần-mất-điện.docx* (19 trang, báo cáo lần 1) và *OOP.txt* (ghi chú phản hồi/định hướng sửa của giảng viên). Phần dưới là **đề xuất đặc tả kỹ thuật dựa trên tài liệu**, không phải các quyết định đã được giảng viên duyệt. Những thực thể và API cụ thể là thiết kế đề xuất nhằm hiện thực hóa yêu cầu đã mô tả.

1. **Mô hình:** một doanh nghiệp với 12 chi nhánh ở TP.HCM, mỗi chi nhánh có kho riêng; dữ liệu và menu lõi dùng chung.
2. **Điều chỉnh theo OOP:** thay nghiệp vụ **bán sỉ/đặt cọc/công nợ khách sỉ** bằng **đơn bán lẻ số lượng lớn có chiết khấu**. Không xây module Wholesale trong MVP.
3. **Vận hành:** chỉ tạm dừng tiếp nhận online ở **chi nhánh thực sự mất khả năng nhận/phục vụ** (thiếu kết nối dự phòng, không an toàn, quá tải hoặc thiếu hàng); **không tự dừng chỉ vì mất điện**.
4. **Tồn kho:** tách **nguyên liệu thô / sơ chế / bao bì**, ghi mẻ sơ chế, hao hụt thực tế, hạn dùng; theo FEFO. Giữ tồn cho đơn được xác nhận; có ngưỡng tồn dự phòng để không bán online hết phần dành cho quầy.
5. **Pha chế:** công thức theo size, đường, đá, topping; chi tiết đơn lưu snapshot giá và phiên bản công thức. Nếu bỏ topping sau khi thu tiền phải giảm/hoàn phần không phục vụ.
6. **Sơ chế phân phối:** hỗ trợ ghi nhận mẻ tại một chi nhánh và **chuyển lô hàng đã sơ chế giữa hai chi nhánh**; chưa thêm **nhà máy/kho trung tâm** do báo cáo loại trừ việc tự sản xuất nguyên liệu tại nhà máy và OOP còn yêu cầu khảo sát thực tế.
7. **Thanh toán/hóa đơn/đối tác giao hàng:** mô phỏng; không tích hợp thật, không làm kế toán tổng hợp, không dự báo AI.
8. **Gián đoạn:** lưu đơn offline **trên thiết bị POS** (ví dụ IndexedDB) và đồng bộ lên máy chủ bằng khóa chống trùng. `sync_batches` ở server chỉ ghi nhận kết quả tiếp nhận, không thay thế kho lưu cục bộ.
9. **Xác thực khách hàng:** khách thành viên dùng OTP xác minh số điện thoại, có thể cấp phiên bằng token ngắn hạn; OTP/challenge có TTL trong Redis, không cần bảng tài chính/phức tạp. Khách vãng lai truy cập đơn qua mã theo dõi bí mật đủ mạnh, không thể dùng một mã đơn công khai để xem đơn người khác.
10. **Vai trò đề xuất:** OWNER, ADMIN, BRANCH_MANAGER, CASHIER, INVENTORY_STAFF. Theo báo cáo, nhân viên pha chế **không bắt buộc** có tài khoản riêng; khách có thể đặt không đăng ký, thành viên dựa trên số điện thoại xác minh. Đây là thiết kế phân quyền đề xuất, có thể tinh chỉnh.
11. **Mức ưu tiên:** `Bắt buộc` tương ứng MVP; `Nên có` là mở rộng trong đồ án. FR chưa tự đặt các con số ngưỡng tồn, điểm, chiết khấu, SLA khi nguồn chưa quy định.

### Chưa chốt và cần xác nhận trước khi triển khai thực tế

- Sơ chế tập trung ở một chi nhánh và phân phối đi nơi khác là **tình huống mở rộng được mô hình hỗ trợ**, không khẳng định mọi cửa hàng vận hành như vậy.
- Chính sách số lượng lớn (ngưỡng bao nhiêu ly, % chiết khấu), thứ tự ưu tiên khách tại quầy với đơn online, ngưỡng tồn dự phòng và thời gian nhận đơn phải **cấu hình**; báo cáo chỉ có ví dụ quá hạn 5 phút.
- Đồng nhất dữ liệu tại quầy và online cần kiểm thử cạnh tranh cập nhật, ngoại lệ tồn âm, tồn sai do hao hụt thực tế và đối soát sau offline; cách trừ tồn ở các bước có thể điều chỉnh sau khi chốt chính sách nghiệp vụ.
- Dữ liệu phòng bếp/pha chế theo báo cáo hiện có thể vận hành qua phiếu in và trạng thái do thu ngân/quản lý cập nhật, không bắt buộc bổ sung vai trò BARISTA vào hệ thống.

## Sơ đồ module và phạm vi

| Mã module | Module | Phạm vi chính | Bảng DB | FR | UC | API |
| --- | --- | --- | --- | --- | --- | --- |
| AU | Tài khoản & phân quyền | Đăng nhập, tài khoản nhân viên, phân quyền và audit | 4 | 4 | 3 | 8 |
| BR | Chi nhánh, nhân sự & ca bán | Chi nhánh, vùng giao, quầy POS, ca bán và sự cố | 5 | 4 | 3 | 7 |
| CA | Sản phẩm, giá & công thức | Menu, biến thể, đường/đá/topping, giá, công thức | 10 | 6 | 3 | 10 |
| IN | Kho, sơ chế & cung ứng | Nhập hàng, kho theo lô, sơ chế, điều chuyển, kiểm kê | 17 | 12 | 7 | 17 |
| OR | Đơn hàng, đặt online & pha chế | POS, online, hàng đợi pha chế, đơn nhiều ly | 6 | 9 | 7 | 12 |
| PA | Thanh toán, hoàn tiền & đối soát | Thu tiền, hoàn tiền, đối soát thanh toán và ca | 3 | 4 | 3 | 5 |
| CR | Khách hàng, thành viên & ưu đãi | Thành viên, hạng, điểm và khuyến mãi | 9 | 8 | 6 | 13 |
| SY | Gián đoạn vận hành & đồng bộ | Offline POS, tiếp nhận online theo chi nhánh, đồng bộ | 2 | 6 | 4 | 7 |
| RP | Báo cáo & đánh giá chi nhánh | Doanh thu, tồn, hao hụt, hiệu quả chi nhánh | 1 | 6 | 4 | 6 |

## Quy ước mã và liên kết

- **FR-xx**: số thứ tự toàn cục của yêu cầu chức năng.
- **UC-xx**: số thứ tự toàn cục của ca sử dụng, tham chiếu FR.
- **API-AU/BR/CA/IN/OR/PA/CR/SY/RP-xx**: mã API theo module, liên kết UC.
- **DB**: tên bảng, cột và quan hệ giống hoàn toàn với ERD `.puml`.
- **Trạng thái thanh toán/đơn**: danh sách trạng thái mang tính sơ bộ; cần đóng băng quy tắc chuyển trạng thái trước khi làm OpenAPI và code.


---

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


---

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


---

# DB — Bảng dữ liệu & ràng buộc

> Phiên bản: 0.1 (bản phân tích ban đầu) · 01/11/2026 · Nguồn: Báo cáo lần 1 và ghi chú OOP.

Lược đồ logic đề xuất; tên khóa và bảng là căn cứ đồng nhất cho ERD PlantUML. Ràng buộc NULL/NOT NULL thể hiện chi tiết trên ERD (dấu ?).


## AU — Tài khoản & phân quyền

| Bảng/Thực thể | Mục đích | Khóa chính | Khóa ngoại | Trường quan trọng | Ràng buộc | Lớp/Yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| users | Tài khoản nhân viên vận hành hệ thống | user_id | — | email, phone, password_hash, full_name, role, status, created_at, updated_at | UNIQUE(email); role ∈ OWNER/ADMIN/BRANCH_MANAGER/CASHIER/INVENTORY_STAFF; thu hồi quyền ngay. | Tài khoản & phân quyền; FR-01–04 |
| user_sessions | Quản lý phiên đăng nhập và thu hồi | session_id | user_id → users.user_id | user_id, refresh_token_hash, device_info, expires_at, revoked_at | Không lưu refresh token thô; phiên hết hạn/thu hồi không còn hợp lệ. | Tài khoản & phân quyền; FR-01, FR-03 |
| staff_assignments | Lịch sử phân công nhân viên theo chi nhánh | assignment_id | user_id → users.user_id; branch_id → branches.branch_id | user_id, branch_id, started_at, ended_at, status | Không cấp quyền chi nhánh ngoài phân công có hiệu lực; giữ lịch sử điều chuyển. | Tài khoản & phân quyền; FR-02, FR-06 |
| audit_logs | Nhật ký giao dịch nhạy cảm | audit_id | actor_id → users.user_id; branch_id → branches.branch_id | actor_id, branch_id, action, entity_type, entity_id, reason, created_at | Bất biến/append-only; actor_id hoặc branch_id có thể NULL cho hành động hệ thống. | Tài khoản & phân quyền; FR-04 |

## BR — Chi nhánh, nhân sự & ca bán

| Bảng/Thực thể | Mục đích | Khóa chính | Khóa ngoại | Trường quan trọng | Ràng buộc | Lớp/Yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| branches | Thông tin và trạng thái từng cửa hàng | branch_id | — | branch_code, name, address, operating_hours, branch_status, online_acceptance, max_pending_online, created_at | UNIQUE(branch_code); trạng thái ONLINE tách với trạng thái vận hành chi nhánh. | Chi nhánh, nhân sự & ca bán; FR-05, FR-08, FR-49–50 |
| delivery_zones | Vùng giao nhận chi nhánh có thể phục vụ | zone_id | branch_id → branches.branch_id | branch_id, zone_name, ward_or_area_code, active, fee_hint | Một vùng có thể có nhiều chi nhánh; không mặc định thuật toán tuyến giao hàng. | Chi nhánh, nhân sự & ca bán; FR-08, FR-28 |
| pos_terminals | Quầy/thiết bị POS đặt tại chi nhánh | terminal_id | branch_id → branches.branch_id | branch_id, code, status | UNIQUE(branch_id, code); mỗi terminal chỉ có một ca mở. | Chi nhánh, nhân sự & ca bán; FR-06–07 |
| cash_shifts | Ca bán hàng tại quầy | shift_id | terminal_id → pos_terminals.terminal_id; cashier_id → users.user_id | terminal_id, cashier_id, opened_at, closed_at, opening_cash, status | UNIQUE một ca OPEN/terminal; không đóng ca nếu chưa giải quyết các giao dịch bắt buộc. | Chi nhánh, nhân sự & ca bán; FR-07, FR-39 |
| branch_incidents | Sự cố mạng/điện/công suất theo chi nhánh | incident_id | branch_id → branches.branch_id; reported_by → users.user_id | branch_id, reported_by, incident_type, offline_since, recovered_at, impact_level, action_taken | Không suy ra mất điện = không thể nhận online; chỉ thay đổi đúng chi nhánh. | Chi nhánh, nhân sự & ca bán; FR-49–50 |

## CA — Sản phẩm, giá & công thức

| Bảng/Thực thể | Mục đích | Khóa chính | Khóa ngoại | Trường quan trọng | Ràng buộc | Lớp/Yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| products | Danh mục đồ uống toàn chuỗi | product_id | — | code, name, category, description, status | UNIQUE(code); không xóa sản phẩm đã xuất hiện trong đơn. | Sản phẩm, giá & công thức; FR-09 |
| product_variants | Biến thể món theo size | variant_id | product_id → products.product_id | product_id, size_code, display_name, active | UNIQUE(product_id, size_code); size ảnh hưởng công thức và giá. | Sản phẩm, giá & công thức; FR-10, FR-14 |
| option_groups | Nhóm tùy chọn đường/đá/topping | option_group_id | — | code, name, selection_mode, max_selection | UNIQUE(code); quy định số lựa chọn và tính bắt buộc. | Sản phẩm, giá & công thức; FR-10 |
| option_values | Giá trị lựa chọn cho từng nhóm | option_value_id | option_group_id → option_groups.option_group_id | option_group_id, code, display_name, active | UNIQUE(option_group_id, code); topping có thể là phần tiêu hao nguyên liệu. | Sản phẩm, giá & công thức; FR-10, FR-14 |
| variant_option_groups | Nhóm lựa chọn áp dụng cho từng size | variant_id, option_group_id | variant_id → product_variants.variant_id; option_group_id → option_groups.option_group_id | required, max_selection | PK(variant_id, option_group_id); không chọn option ngoài phạm vi món. | Sản phẩm, giá & công thức; FR-10 |
| price_lists | Phiên bản bảng giá theo phạm vi và hiệu lực | price_list_id | branch_id → branches.branch_id; approved_by → users.user_id | branch_id, scope, channel, effective_from, effective_to, status, approved_by | scope ∈ CHAIN/BRANCH/CHANNEL/BRANCH_CHANNEL; branch_id tùy phạm vi; không chồng lấn cùng khóa phạm vi. | Sản phẩm, giá & công thức; FR-11 |
| price_list_items | Giá từng size hoặc phụ phí lựa chọn | price_item_id | price_list_id → price_lists.price_list_id; variant_id → product_variants.variant_id; option_value_id → option_values.option_value_id | price_list_id, variant_id, option_value_id, unit_price | CHECK đúng một trong variant_id/option_value_id khác NULL; giá không âm; khóa duy nhất theo loại mục. | Sản phẩm, giá & công thức; FR-11 |
| recipes | Phiên bản công thức cho size/lựa chọn/mẻ sơ chế | recipe_id | variant_id → product_variants.variant_id; option_value_id → option_values.option_value_id; prepared_item_id → inventory_items.item_id; approved_by → users.user_id | variant_id, option_value_id, prepared_item_id, version_no, output_quantity, status, valid_from, valid_to, approved_by | CHECK đúng một chủ thể variant/option/prepared_item; phiên bản APPROVED không sửa nội dung. | Sản phẩm, giá & công thức; FR-12, FR-14, FR-21 |
| recipe_components | Định lượng nguyên liệu theo công thức | component_id | recipe_id → recipes.recipe_id; item_id → inventory_items.item_id | recipe_id, item_id, quantity, unit | UNIQUE(recipe_id, item_id) trong cùng công đoạn; quantity ≥ 0; quy đổi về đơn vị kho. | Sản phẩm, giá & công thức; FR-12, FR-14 |
| branch_menu_availability | Trạng thái bán size tại từng chi nhánh | branch_id, variant_id | branch_id → branches.branch_id; variant_id → product_variants.variant_id | is_enabled, reason, changed_at | PK(branch_id,variant_id); chỉ chi nhánh được tạm khóa, không sửa giá gốc. | Sản phẩm, giá & công thức; FR-13, FR-25 |

## IN — Kho, sơ chế & cung ứng

| Bảng/Thực thể | Mục đích | Khóa chính | Khóa ngoại | Trường quan trọng | Ràng buộc | Lớp/Yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| inventory_items | Nguyên liệu thô, thành phẩm sơ chế, bao bì | item_id | — | sku, name, item_type, base_unit, shelf_life_hours, storage_note | UNIQUE(sku); item_type ∈ RAW/PREPARED/PACKAGING; đơn vị lượng > 0. | Kho, sơ chế & cung ứng; FR-15, FR-21 |
| branch_stock_policies | Ngưỡng tồn tối thiểu/dự phòng online | branch_id, item_id | branch_id → branches.branch_id; item_id → inventory_items.item_id | safety_qty, online_buffer_qty, reorder_point | PK(branch_id,item_id); ngưỡng không âm; online_available = max(0, available − online_buffer_qty). | Kho, sơ chế & cung ứng; FR-18, FR-25 |
| suppliers | Nhà cung cấp nguyên liệu | supplier_id | — | name, contact_info, status | Theo dõi lịch sử nhà cung cấp; không cho xóa khi còn phiếu mua. | Kho, sơ chế & cung ứng; FR-16 |
| purchase_orders | Phiếu mua nguyên liệu theo kho chi nhánh | purchase_id | branch_id → branches.branch_id; supplier_id → suppliers.supplier_id; created_by → users.user_id | branch_id, supplier_id, created_by, ordered_at, expected_at, status | Chỉ nhận theo đơn mua hợp lệ hoặc luồng nhập điều chỉnh được duyệt. | Kho, sơ chế & cung ứng; FR-16 |
| purchase_order_lines | Dòng nguyên liệu của phiếu mua | purchase_line_id | purchase_id → purchase_orders.purchase_id; item_id → inventory_items.item_id | purchase_id, item_id, ordered_qty, unit_price | ordered_qty > 0; unit_price ≥ 0; không nhận vượt quy định. | Kho, sơ chế & cung ứng; FR-16 |
| stock_lots | Lô tồn theo chi nhánh, nguồn gốc và hạn dùng | lot_id | branch_id → branches.branch_id; item_id → inventory_items.item_id; supplier_id → suppliers.supplier_id; source_lot_id → stock_lots.lot_id | branch_id, item_id, supplier_id, source_lot_id, lot_code, manufactured_at, expires_at, quality_status, unit_cost | UNIQUE(branch_id, item_id, lot_code); source_lot_id khi chuyển kho; FEFO; lô QUARANTINED/EXPIRED không được bán. | Kho, sơ chế & cung ứng; FR-17–18, FR-23 |
| stock_balances | Số dư vật lý, giữ chỗ và khóa kiểm kê theo lô | lot_id | lot_id → stock_lots.lot_id | qty_on_hand, qty_reserved, qty_locked, updated_at | PK(lot_id); qty_on_hand ≥ qty_reserved + qty_locked; dùng khóa giao dịch khi giữ/trừ. | Kho, sơ chế & cung ứng; FR-18–19 |
| stock_movements | Sổ giao dịch biến động tồn kho bất biến | movement_id | lot_id → stock_lots.lot_id; actor_id → users.user_id | lot_id, actor_id, movement_type, qty_delta, source_kind, source_id, created_at | Append-only; nguồn giao dịch chống ghi lặp; không sửa số dư không có movement. | Kho, sơ chế & cung ứng; FR-17, FR-20, FR-22–24 |
| goods_receipts | Phiếu nhận hàng từ nhà cung cấp | receipt_id | purchase_id → purchase_orders.purchase_id; received_by → users.user_id | purchase_id, received_by, received_at, status | Trạng thái nhận và chấp nhận chứng từ, chỉ nhập phần chất lượng đạt. | Kho, sơ chế & cung ứng; FR-17 |
| goods_receipt_lines | Mặt hàng/lô thực nhận | receipt_line_id | receipt_id → goods_receipts.receipt_id; purchase_line_id → purchase_order_lines.purchase_line_id; lot_id → stock_lots.lot_id | receipt_id, purchase_line_id, lot_id, accepted_qty, rejected_qty, rejection_reason | accepted_qty/rejected_qty ≥ 0; chỉ accepted_qty tạo movement nhập. | Kho, sơ chế & cung ứng; FR-17 |
| stock_reservations | Giữ lô cho chi tiết đơn đã xác nhận | reservation_id | order_item_id → order_items.order_item_id; lot_id → stock_lots.lot_id | order_item_id, lot_id, reserved_qty, status, created_at, released_at | UNIQUE(order_item_id, lot_id, status ACTIVE theo logic); chỉ giữ trên tồn khả dụng. | Kho, sơ chế & cung ứng; FR-19, FR-29 |
| preparation_batches | Mẻ sơ chế/pha sẵn tạo hàng tồn mới | prep_batch_id | branch_id → branches.branch_id; recipe_id → recipes.recipe_id; output_lot_id → stock_lots.lot_id | branch_id, recipe_id, output_lot_id, planned_yield, actual_yield, wasted_qty, prepared_at, status | Mỗi mẻ sinh một lô thành phẩm; hạn dùng từ quy tắc bảo quản; đầu ra không thể vượt thực thu. | Kho, sơ chế & cung ứng; FR-21–22 |
| preparation_inputs | Lô nguyên liệu thực dùng của mẻ | prep_input_id | prep_batch_id → preparation_batches.prep_batch_id; lot_id → stock_lots.lot_id | prep_batch_id, lot_id, consumed_qty | consumed_qty > 0; nguyên liệu đầu vào trừ kho qua movement một lần. | Kho, sơ chế & cung ứng; FR-20–22 |
| stock_transfers | Điều chuyển kho giữa hai chi nhánh | transfer_id | source_branch_id → branches.branch_id; target_branch_id → branches.branch_id; requested_by → users.user_id; approved_by → users.user_id | source_branch_id, target_branch_id, requested_by, approved_by, status, created_at | CHECK source_branch_id ≠ target_branch_id; chuyển hoàn tất chỉ khi kho nhận xác nhận. | Kho, sơ chế & cung ứng; FR-23 |
| stock_transfer_lines | Chi tiết lô và lượng gửi/nhận | transfer_line_id | transfer_id → stock_transfers.transfer_id; item_id → inventory_items.item_id; source_lot_id → stock_lots.lot_id; target_lot_id → stock_lots.lot_id | transfer_id, item_id, source_lot_id, target_lot_id, requested_qty, shipped_qty, received_qty | Không sửa shipped_qty lịch sử; nhận thiếu lập chênh lệch, tạo lô đích giữ hạn dùng. | Kho, sơ chế & cung ứng; FR-23 |
| stock_counts | Đợt kiểm kê kho chi nhánh | stock_count_id | branch_id → branches.branch_id; opened_by → users.user_id | branch_id, opened_by, opened_at, closed_at, status | UNIQUE một phiên OPEN trên một chi nhánh; khóa kiểm kê theo chính sách. | Kho, sơ chế & cung ứng; FR-24 |
| stock_count_lines | Kết quả đếm và xử lý chênh lệch | count_line_id | stock_count_id → stock_counts.stock_count_id; lot_id → stock_lots.lot_id; approved_by → users.user_id | stock_count_id, lot_id, system_qty, counted_qty, reason, approved_by | UNIQUE(stock_count_id, lot_id); chênh lệch được duyệt trước khi tạo movement. | Kho, sơ chế & cung ứng; FR-22, FR-24 |

## OR — Đơn hàng, đặt online & pha chế

| Bảng/Thực thể | Mục đích | Khóa chính | Khóa ngoại | Trường quan trọng | Ràng buộc | Lớp/Yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| orders | Đơn bán đa kênh, tất cả thuộc một chi nhánh | order_id | branch_id → branches.branch_id; customer_id → customers.customer_id; shift_id → cash_shifts.shift_id | branch_id, customer_id, shift_id, client_order_key, channel, fulfillment, status, subtotal, discount_total, final_total, placed_at, completed_at | UNIQUE(branch_id,client_order_key); mỗi đơn thuộc một chi nhánh; không có đơn sỉ riêng. | Đơn hàng, đặt online & pha chế; FR-27–30, FR-34, FR-48, FR-51 |
| order_items | Chi tiết món, số ly và giá/công thức chốt | order_item_id | order_id → orders.order_id; variant_id → product_variants.variant_id; recipe_id → recipes.recipe_id | order_id, variant_id, recipe_id, quantity, size_snapshot, unit_price_snapshot, line_total, item_status | quantity>0; lưu snapshot giá và công thức; thay đổi phải có log. | Đơn hàng, đặt online & pha chế; FR-27, FR-30, FR-32 |
| order_item_options | Lựa chọn đường/đá/topping đã bán | order_option_id | order_item_id → order_items.order_item_id; option_value_id → option_values.option_value_id; recipe_id → recipes.recipe_id | order_item_id, option_value_id, recipe_id, extra_price_snapshot, quantity | Lưu tên/lựa chọn qua tham chiếu và giá tại thời điểm bán; option thiếu được hoàn chênh. | Đơn hàng, đặt online & pha chế; FR-10, FR-32 |
| order_status_events | Lịch sử chuyển trạng thái đơn | order_event_id | order_id → orders.order_id; actor_id → users.user_id | order_id, actor_id, from_status, to_status, reason, occurred_at | Append-only; chuyển trạng thái theo state machine, lưu lý do hủy. | Đơn hàng, đặt online & pha chế; FR-30, FR-35 |
| kitchen_tickets | Phiếu pha chế/hàng đợi chung tại chi nhánh | ticket_id | order_id → orders.order_id | order_id, queue_number, status, printed_at, ready_at | UNIQUE(order_id) cho phiếu hiện hành; POS và online dùng chung; chống tạo trùng. | Đơn hàng, đặt online & pha chế; FR-31 |
| order_deliveries | Thông tin nhận/giao đơn online | delivery_id | order_id → orders.order_id | order_id, receiver_name, phone, delivery_address, delivery_fee, delivery_status, partner_ref | UNIQUE(order_id) nếu fulfillment DELIVERY; không tối ưu tuyến chuyên sâu. | Đơn hàng, đặt online & pha chế; FR-28, FR-33 |

## PA — Thanh toán, hoàn tiền & đối soát

| Bảng/Thực thể | Mục đích | Khóa chính | Khóa ngoại | Trường quan trọng | Ràng buộc | Lớp/Yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| payments | Các lần thu tiền theo đơn | payment_id | order_id → orders.order_id; shift_id → cash_shifts.shift_id | order_id, shift_id, method, amount, status, provider_transaction_id, paid_at | UNIQUE(provider_transaction_id) khi khác NULL; trạng thái UNKNOWN phải xác minh; không lưu số thẻ đầy đủ. | Thanh toán, hoàn tiền & đối soát; FR-36–37, FR-39 |
| refunds | Hoàn tiền liên kết giao dịch đã thu | refund_id | payment_id → payments.payment_id; order_id → orders.order_id; approved_by → users.user_id | payment_id, order_id, approved_by, amount, reason, status, refunded_at | Tổng refund thành công ≤ tổng tiền đã thu hợp lệ; idempotency cho hoàn tiền. | Thanh toán, hoàn tiền & đối soát; FR-38 |
| cash_shift_closures | Biên bản kiểm tiền và chênh lệch cuối ca | closure_id | shift_id → cash_shifts.shift_id; approved_by → users.user_id | shift_id, expected_cash, counted_cash, difference, explanation, approved_by, closed_at | UNIQUE(shift_id); chênh lệch phải có giải trình và phê duyệt khi vượt ngưỡng. | Thanh toán, hoàn tiền & đối soát; FR-07, FR-39 |

## CR — Khách hàng, thành viên & ưu đãi

| Bảng/Thực thể | Mục đích | Khóa chính | Khóa ngoại | Trường quan trọng | Ràng buộc | Lớp/Yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| customers | Hồ sơ khách và thông tin đồng ý | customer_id | — | phone, full_name, phone_verified_at, personal_data_consent_at, marketing_opt_in, member_since | UNIQUE(phone) nếu có; khách lẻ không bắt buộc có customer; chỉ xử lý dữ liệu đã đồng ý. | Khách hàng, thành viên & ưu đãi; FR-40–41 |
| customer_complaints | Khiếu nại về sản phẩm/giao nhận/thanh toán | complaint_id | order_id → orders.order_id; customer_id → customers.customer_id; handled_by → users.user_id | order_id, customer_id, category, description, evidence_url, status, resolution, handled_by, resolved_at | Có mã đơn; ghi bằng chứng, người giải quyết và kết quả làm lại/voucher/hoàn tiền. | Khách hàng, thành viên & ưu đãi; FR-35, FR-38, FR-47 |
| membership_tiers | Cấu hình 4 hạng thành viên | tier_id | — | code, min_annual_spend, point_multiplier, active | UNIQUE(code); code ∈ MEMBER/SILVER/GOLD/VIP; ngưỡng do quản trị cấu hình. | Khách hàng, thành viên & ưu đãi; FR-43 |
| customer_tier_history | Lịch sử xét hạng khách hàng | tier_history_id | customer_id → customers.customer_id; tier_id → membership_tiers.tier_id | customer_id, tier_id, qualified_spend, evaluated_at, valid_from, valid_to | Một hạng có hiệu lực tại một thời điểm; xét theo chu kỳ 12 tháng. | Khách hàng, thành viên & ưu đãi; FR-43 |
| loyalty_transactions | Sổ điểm cộng/trừ/đảo | point_txn_id | customer_id → customers.customer_id; order_id → orders.order_id; approved_by → users.user_id | customer_id, order_id, txn_type, points_delta, source_key, expires_at, approved_by, created_at | UNIQUE(source_key); không sửa số dư trực tiếp; một đơn hoàn tất chỉ EARN một lần. | Khách hàng, thành viên & ưu đãi; FR-42 |
| promotions | Chính sách khuyến mãi, chiết khấu số lượng | promotion_id | — | code, name, discount_type, discount_value, min_order_quantity, min_order_value, stackable, budget_total, starts_at, ends_at, status | UNIQUE(code); kiểm tra ngân sách, hiệu lực; có thể áp theo kênh được cấu hình. | Khách hàng, thành viên & ưu đãi; FR-34, FR-44–45 |
| promotion_branches | Phạm vi chi nhánh của khuyến mãi | promotion_id, branch_id | promotion_id → promotions.promotion_id; branch_id → branches.branch_id |  | PK(promotion_id,branch_id); không có dòng phạm vi = áp toàn chuỗi khi promotion cấu hình GLOBAL. | Khách hàng, thành viên & ưu đãi; FR-44 |
| vouchers | Voucher theo khuyến mãi | voucher_id | promotion_id → promotions.promotion_id; customer_id → customers.customer_id | promotion_id, customer_id, code, max_redemptions, redeemed_count, expires_at | UNIQUE(code); redeemed_count ≤ max_redemptions; khi hoàn hủy cần quy tắc trả lượt. | Khách hàng, thành viên & ưu đãi; FR-46 |
| order_promotions | Ưu đãi đã áp trên đơn | applied_promo_id | order_id → orders.order_id; promotion_id → promotions.promotion_id; voucher_id → vouchers.voucher_id | order_id, promotion_id, voucher_id, discount_amount, applied_at | Không cộng chồng nếu cấm; không áp voucher ngoài phạm vi, tránh ghi lặp. | Khách hàng, thành viên & ưu đãi; FR-34, FR-44–46 |

## SY — Gián đoạn vận hành & đồng bộ

| Bảng/Thực thể | Mục đích | Khóa chính | Khóa ngoại | Trường quan trọng | Ràng buộc | Lớp/Yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| sync_batches | Lần gửi đồng bộ offline từ một POS | sync_batch_id | terminal_id → pos_terminals.terminal_id; submitted_by → users.user_id | terminal_id, submitted_by, started_at, finished_at, status | Một batch có kết quả riêng; không xóa queue cục bộ khi chưa có ACK. | Gián đoạn vận hành & đồng bộ; FR-48, FR-51 |
| sync_batch_orders | Kết quả tiếp nhận từng mã đơn offline | sync_item_id | sync_batch_id → sync_batches.sync_batch_id; order_id → orders.order_id | sync_batch_id, order_id, client_order_key, payload_hash, outcome, failure_reason | UNIQUE(sync_batch_id,client_order_key); server đối chiếu UNIQUE(branch_id,client_order_key) ở orders. | Gián đoạn vận hành & đồng bộ; FR-51–52 |

## RP — Báo cáo & đánh giá chi nhánh

| Bảng/Thực thể | Mục đích | Khóa chính | Khóa ngoại | Trường quan trọng | Ràng buộc | Lớp/Yêu cầu liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| branch_period_costs | Chi phí kỳ phục vụ ước tính biên đóng góp | cost_id | branch_id → branches.branch_id; recorded_by → users.user_id | branch_id, period_start, period_end, rent_cost, staff_cost, utilities_cost, other_cost, recorded_by | UNIQUE(branch_id,period_start,period_end); dữ liệu khai báo, không thay kế toán tài chính. | Báo cáo & đánh giá chi nhánh; FR-57–58 |


---

# API — Contract tổng quan

> Phiên bản: 0.1 (bản phân tích ban đầu) · 01/11/2026 · Nguồn: Báo cáo lần 1 và ghi chú OOP.

Phần này là danh mục API mức tổng quan, **chưa phải** OpenAPI chi tiết gồm request/response DTO và error code.


## AU — Tài khoản & phân quyền

| Mã API | Tên API | Method | URL | Mục đích | Xác thực / Phân quyền | UC liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| API-AU-01 | Đăng nhập | POST | `/api/v1/auth/login` | Xác thực nhân viên, cấp phiên | Public; tài khoản nhân viên hợp lệ | UC-01 |
| API-AU-02 | Gia hạn phiên | POST | `/api/v1/auth/refresh` | Cấp lại access token | Refresh session hợp lệ | UC-01 |
| API-AU-03 | Đăng xuất | POST | `/api/v1/auth/logout` | Thu hồi phiên hiện tại | Authenticated | UC-01 |
| API-AU-04 | Thông tin phiên | GET | `/api/v1/auth/me` | Trả thông tin người dùng và phạm vi | Authenticated | UC-01 |
| API-AU-05 | Danh sách nhân viên | GET | `/api/v1/users` | Tìm nhân viên theo phạm vi | ADMIN/OWNER; manager chỉ chi nhánh | UC-02 |
| API-AU-06 | Đổi quyền nhân viên | PATCH | `/api/v1/users/{id}/access` | Cập nhật role/trạng thái, thu hồi phiên | ADMIN/OWNER | UC-02 |
| API-AU-07 | Phân công nhân viên | POST | `/api/v1/staff-assignments` | Giao nhân viên vào chi nhánh | ADMIN/OWNER | UC-02 |
| API-AU-08 | Tra cứu nhật ký | GET | `/api/v1/audit-logs` | Lọc thao tác nhạy cảm | ADMIN/OWNER, theo phạm vi | UC-03 |

## BR — Chi nhánh, nhân sự & ca bán

| Mã API | Tên API | Method | URL | Mục đích | Xác thực / Phân quyền | UC liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| API-BR-01 | Danh sách chi nhánh | GET | `/api/v1/branches` | Tìm chi nhánh và trạng thái | Authenticated; Public có thể xem chi nhánh hoạt động | UC-04 |
| API-BR-02 | Thêm chi nhánh | POST | `/api/v1/branches` | Tạo điểm bán mới | OWNER/ADMIN | UC-04 |
| API-BR-03 | Cập nhật chi nhánh | PATCH | `/api/v1/branches/{id}` | Cập nhật thông tin và nhận đơn | OWNER/ADMIN | UC-04 |
| API-BR-04 | Cập nhật vùng giao | PUT | `/api/v1/branches/{id}/delivery-zones` | Cấu hình địa bàn nhận online | OWNER/ADMIN | UC-04 |
| API-BR-05 | Tạo quầy POS | POST | `/api/v1/pos-terminals` | Khai báo thiết bị/quầy | ADMIN/BRANCH_MANAGER theo chi nhánh | UC-05 |
| API-BR-06 | Mở ca | POST | `/api/v1/cash-shifts/open` | Mở ca cho quầy | CASHIER được phân công | UC-06 |
| API-BR-07 | Đóng ca | POST | `/api/v1/cash-shifts/{id}/close` | Đối chiếu tiền và chốt ca | CASHIER; quản lý duyệt khi lệch | UC-06 |

## CA — Sản phẩm, giá & công thức

| Mã API | Tên API | Method | URL | Mục đích | Xác thực / Phân quyền | UC liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| API-CA-01 | Tra cứu menu | GET | `/api/v1/catalog/products` | Liệt kê món và size | Public; lọc branch/channel | UC-07 |
| API-CA-02 | Tạo món | POST | `/api/v1/catalog/products` | Thêm đồ uống | ADMIN/OWNER | UC-07 |
| API-CA-03 | Sửa món | PATCH | `/api/v1/catalog/products/{id}` | Chỉnh thông tin chung | ADMIN/OWNER | UC-07 |
| API-CA-04 | Thêm biến thể | POST | `/api/v1/catalog/variants` | Khai báo size | ADMIN/OWNER | UC-07 |
| API-CA-05 | Cấu hình lựa chọn | PUT | `/api/v1/catalog/variants/{id}/option-groups` | Khai báo đường, đá, topping hợp lệ | ADMIN/OWNER | UC-07 |
| API-CA-06 | Tạo bảng giá | POST | `/api/v1/price-lists` | Tạo phiên bản giá theo phạm vi | ADMIN/OWNER | UC-08 |
| API-CA-07 | Duyệt giá | POST | `/api/v1/price-lists/{id}/approve` | Kích hoạt bảng giá có hiệu lực | OWNER hoặc quyền duyệt | UC-08 |
| API-CA-08 | Tạo công thức | POST | `/api/v1/recipes` | Định lượng cho món, topping hoặc sơ chế | ADMIN/OWNER | UC-08 |
| API-CA-09 | Duyệt công thức | POST | `/api/v1/recipes/{id}/approve` | Duyệt phiên bản định mức | OWNER hoặc quyền duyệt | UC-08 |
| API-CA-10 | Khóa/mở món | PUT | `/api/v1/branches/{id}/menu-availability/{variantId}` | Cập nhật bán món theo chi nhánh | BRANCH_MANAGER tại chi nhánh | UC-09 |

## IN — Kho, sơ chế & cung ứng

| Mã API | Tên API | Method | URL | Mục đích | Xác thực / Phân quyền | UC liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| API-IN-01 | Danh mục nguyên liệu | GET | `/api/v1/inventory/items` | Tra cứu vật tư và tồn an toàn | INVENTORY_STAFF/BRANCH_MANAGER | UC-10 |
| API-IN-02 | Thêm nguyên liệu | POST | `/api/v1/inventory/items` | Tạo mặt hàng tồn | ADMIN/OWNER | UC-10 |
| API-IN-03 | Ngưỡng tồn | PUT | `/api/v1/branches/{id}/stock-policies/{itemId}` | Đặt tồn an toàn/dự phòng online | BRANCH_MANAGER tại chi nhánh | UC-10 |
| API-IN-04 | Thêm nhà cung cấp | POST | `/api/v1/suppliers` | Quản lý đầu mối nhập hàng | ADMIN/OWNER | UC-11 |
| API-IN-05 | Lập phiếu mua | POST | `/api/v1/purchase-orders` | Tạo đề nghị mua nguyên liệu | INVENTORY_STAFF, duyệt theo quyền | UC-11 |
| API-IN-06 | Nhận hàng | POST | `/api/v1/goods-receipts` | Xác nhận phần đạt/chưa đạt theo lô | INVENTORY_STAFF tại chi nhánh | UC-11 |
| API-IN-07 | Xem tồn chi nhánh | GET | `/api/v1/branches/{id}/inventory/stock` | Lấy tồn theo lô và khả dụng | INVENTORY_STAFF/BRANCH_MANAGER | UC-13 |
| API-IN-08 | Tạo mẻ sơ chế | POST | `/api/v1/preparation-batches` | Khai báo mẻ, nguyên liệu đầu vào | INVENTORY_STAFF tại chi nhánh | UC-12 |
| API-IN-09 | Kết thúc sơ chế | POST | `/api/v1/preparation-batches/{id}/complete` | Chốt sản lượng, hao hụt và lô thành phẩm | INVENTORY_STAFF tại chi nhánh | UC-12 |
| API-IN-10 | Yêu cầu chuyển kho | POST | `/api/v1/stock-transfers` | Đề nghị hàng từ chi nhánh khác | BRANCH_MANAGER tại chi nhánh | UC-14 |
| API-IN-11 | Duyệt chuyển kho | POST | `/api/v1/stock-transfers/{id}/approve` | Kiểm tra tồn an toàn và duyệt | BRANCH_MANAGER kho xuất | UC-14 |
| API-IN-12 | Xuất điều chuyển | POST | `/api/v1/stock-transfers/{id}/ship` | Đưa lô sang đang chuyển | INVENTORY_STAFF kho xuất | UC-14 |
| API-IN-13 | Nhận điều chuyển | POST | `/api/v1/stock-transfers/{id}/receive` | Ghi nhận lô, lượng thực nhận | INVENTORY_STAFF kho nhận | UC-14 |
| API-IN-14 | Mở kiểm kê | POST | `/api/v1/stock-counts` | Tạo phiên đếm lô | BRANCH_MANAGER | UC-15 |
| API-IN-15 | Chốt kiểm kê | POST | `/api/v1/stock-counts/{id}/close` | Duyệt và tạo movement điều chỉnh | BRANCH_MANAGER hoặc cấp cao hơn | UC-15 |
| API-IN-16 | Cảnh báo kho | GET | `/api/v1/branches/{id}/inventory/alerts` | Xem tồn thấp/cận hạn/online buffer | INVENTORY_STAFF/BRANCH_MANAGER | UC-16 |
| API-IN-17 | Gợi ý bổ sung | GET | `/api/v1/branches/{id}/inventory/replenishment-suggestions` | Đề xuất mua/chuyển để quản lý duyệt | BRANCH_MANAGER | UC-16 |

## OR — Đơn hàng, đặt online & pha chế

| Mã API | Tên API | Method | URL | Mục đích | Xác thực / Phân quyền | UC liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| API-OR-01 | Tạo đơn POS | POST | `/api/v1/orders/pos` | Tạo đơn bán trực tiếp | CASHIER có ca mở | UC-17 |
| API-OR-02 | Đặt hàng online | POST | `/api/v1/orders/online` | Tạo yêu cầu đặt đồ uống | Public (guest) hoặc token thành viên; chống trùng key | UC-18 |
| API-OR-03 | Chi nhánh phục vụ | GET | `/api/v1/online/available-branches` | Đề xuất chi nhánh theo vùng/năng lực | Public | UC-18 |
| API-OR-04 | Xác nhận đơn | POST | `/api/v1/orders/{id}/confirm` | Kiểm tra tồn và giữ nguyên liệu | CASHIER/BRANCH_MANAGER chi nhánh | UC-19 |
| API-OR-05 | Hàng đợi pha chế | GET | `/api/v1/branches/{id}/kitchen-queue` | Xem phiếu theo thứ tự ưu tiên | CASHIER/BRANCH_MANAGER; màn hình bếp theo thiết kế | UC-19, UC-20 |
| API-OR-06 | Hoàn tất pha | POST | `/api/v1/kitchen-tickets/{id}/ready` | Cập nhật món sẵn sàng | CASHIER/BRANCH_MANAGER chi nhánh | UC-20 |
| API-OR-07 | Hoàn tất đơn | POST | `/api/v1/orders/{id}/complete` | Ghi nhận giao món, tiêu hao, điểm | CASHIER/BRANCH_MANAGER chi nhánh | UC-20 |
| API-OR-08 | Thay đổi món/topping | PATCH | `/api/v1/orders/{id}/items/{itemId}` | Ghi nhận đổi/bỏ lựa chọn có đồng ý | CASHIER; duyệt nếu đã thanh toán | UC-21 |
| API-OR-09 | Tính tiền và ưu đãi | POST | `/api/v1/orders/quote` | Kiểm tra số lượng, khả dụng, tổng tiền | Public hoặc CASHIER | UC-22 |
| API-OR-10 | Hủy đơn | POST | `/api/v1/orders/{id}/cancel` | Hủy có lý do, hoàn giữ tồn | CASHIER theo chính sách; MANAGER duyệt | UC-23 |
| API-OR-11 | Chi tiết đơn | GET | `/api/v1/orders/{id}` | Xem đơn và lịch sử trạng thái | Token đơn cho khách vãng lai, token thành viên hoặc nhân viên đúng chi nhánh | UC-17, UC-18 |
| API-OR-12 | Cập nhật giao nhận | PATCH | `/api/v1/orders/{id}/delivery-status` | Ghi trạng thái bàn giao đơn online | CASHIER/BRANCH_MANAGER | UC-20 |

## PA — Thanh toán, hoàn tiền & đối soát

| Mã API | Tên API | Method | URL | Mục đích | Xác thực / Phân quyền | UC liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| API-PA-01 | Thanh toán đơn | POST | `/api/v1/orders/{id}/payments` | Tạo giao dịch thu tiền | CASHIER hoặc khách thanh toán mock | UC-24 |
| API-PA-02 | Lịch sử thanh toán | GET | `/api/v1/orders/{id}/payments` | Xem thanh toán và trạng thái xác minh | Token đơn cho khách vãng lai, token thành viên hoặc nhân viên đúng chi nhánh | UC-24 |
| API-PA-03 | Xác minh thanh toán | POST | `/api/v1/payments/{id}/verify` | Kết thúc giao dịch UNKNOWN bằng đối soát | BRANCH_MANAGER | UC-26, UC-35 |
| API-PA-04 | Hoàn tiền | POST | `/api/v1/payments/{id}/refunds` | Tạo hoàn một phần/toàn phần | BRANCH_MANAGER theo ngưỡng | UC-25 |
| API-PA-05 | Đối chiếu ca | GET | `/api/v1/cash-shifts/{id}/reconciliation` | Đối soát tiền mặt, các hình thức thu | BRANCH_MANAGER/CASHIER đúng ca | UC-26 |

## CR — Khách hàng, thành viên & ưu đãi

| Mã API | Tên API | Method | URL | Mục đích | Xác thực / Phân quyền | UC liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| API-CR-01 | Yêu cầu OTP | POST | `/api/v1/customers/otp/request` | Tạo thử thách xác minh số điện thoại | Public; rate limit; OTP hash/TTL ở Redis | UC-27 |
| API-CR-02 | Kiểm tra OTP | POST | `/api/v1/customers/otp/verify` | Xác minh số điện thoại và cấp phiên khách | Public; OTP hợp lệ và còn hạn | UC-27 |
| API-CR-03 | Đăng ký thành viên | POST | `/api/v1/customers/membership` | Lưu đồng ý và hạng khởi đầu | Public; đã xác minh số điện thoại | UC-27 |
| API-CR-04 | Hồ sơ thành viên | GET | `/api/v1/customers/me` | Tra cứu hồ sơ/đồng ý | Khách được xác thực | UC-27 |
| API-CR-05 | Sổ điểm cá nhân | GET | `/api/v1/customers/me/points` | Xem số dư và giao dịch nguồn | Khách được xác thực | UC-28 |
| API-CR-06 | Hạng và quyền lợi | GET | `/api/v1/customers/me/tier` | Xem hạng hiện tại, lịch sử | Khách được xác thực | UC-29 |
| API-CR-07 | Xét hạng chu kỳ | POST | `/api/v1/loyalty/recalculate-tiers` | Đánh giá lại hạng thành viên | ADMIN/OWNER; tác vụ có phân quyền | UC-29 |
| API-CR-08 | Tạo khuyến mãi | POST | `/api/v1/promotions` | Tạo ưu đãi toàn chuỗi/chi nhánh | OWNER/ADMIN | UC-30 |
| API-CR-09 | Cập nhật khuyến mãi | PATCH | `/api/v1/promotions/{id}` | Cập nhật hiệu lực/ngân sách | OWNER/ADMIN | UC-30 |
| API-CR-10 | Áp ưu đãi | POST | `/api/v1/orders/{id}/apply-promotion` | Thêm khuyến mãi/voucher hợp lệ | Khách sở hữu hoặc CASHIER chi nhánh | UC-31 |
| API-CR-11 | Lịch sử đơn | GET | `/api/v1/customers/me/orders` | Trả các giao dịch của khách | Khách được xác thực | UC-32 |
| API-CR-12 | Tạo khiếu nại | POST | `/api/v1/orders/{id}/complaints` | Ghi phản hồi kèm mã đơn và bằng chứng | Khách có quyền với đơn hoặc CASHIER hỗ trợ | UC-32 |
| API-CR-13 | Xử lý khiếu nại | POST | `/api/v1/complaints/{id}/resolve` | Ghi kết quả xử lý/hoàn tiền được duyệt | BRANCH_MANAGER chi nhánh | UC-32 |

## SY — Gián đoạn vận hành & đồng bộ

| Mã API | Tên API | Method | URL | Mục đích | Xác thực / Phân quyền | UC liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| API-SY-01 | Dữ liệu POS offline | GET | `/api/v1/offline-pos/bootstrap` | Tải menu, giá, phiên bản công thức, quyền và cache an toàn trước sự cố | CASHIER được phân công; cần mạng khi tải | UC-33 |
| API-SY-02 | Đồng bộ đơn offline | POST | `/api/v1/offline-sync/batches` | Nhận queue đơn và trả ACK từng phần | CASHIER theo terminal/chi nhánh | UC-35 |
| API-SY-03 | Kết quả đồng bộ | GET | `/api/v1/offline-sync/batches/{id}` | Xem accepted/duplicate/conflict | CASHIER/BRANCH_MANAGER đúng chi nhánh | UC-35 |
| API-SY-04 | Báo sự cố | POST | `/api/v1/branches/{id}/incidents` | Khai báo điện, mạng, quá tải | BRANCH_MANAGER chi nhánh | UC-34 |
| API-SY-05 | Điều khiển nhận online | PATCH | `/api/v1/branches/{id}/online-acceptance` | Tạm ngừng/khôi phục có lý do | BRANCH_MANAGER chi nhánh | UC-34 |
| API-SY-06 | Đối soát đơn offline | POST | `/api/v1/offline-sync/orders/{clientKey}/reconcile` | Ghi xử lý thanh toán/tồn xung đột | BRANCH_MANAGER chi nhánh | UC-35 |
| API-SY-07 | Xử lý quá hạn | POST | `/api/v1/orders/{id}/timeout-review` | Cảnh báo/hủy đơn chờ xác nhận | Hệ thống hoặc BRANCH_MANAGER | UC-36 |

## RP — Báo cáo & đánh giá chi nhánh

| Mã API | Tên API | Method | URL | Mục đích | Xác thực / Phân quyền | UC liên quan |
| --- | --- | --- | --- | --- | --- | --- |
| API-RP-01 | Báo cáo doanh số | GET | `/api/v1/reports/sales` | Doanh thu, số đơn, AOV theo kỳ | OWNER/ADMIN; manager giới hạn chi nhánh | UC-37 |
| API-RP-02 | Báo cáo tồn và hao hụt | GET | `/api/v1/reports/inventory` | Theo dõi lô, điều chỉnh, cận hạn | OWNER/ADMIN/BRANCH_MANAGER | UC-38 |
| API-RP-03 | Hiệu quả sản phẩm | GET | `/api/v1/reports/products` | Lượng bán, nguyên liệu sử dụng | OWNER/ADMIN/BRANCH_MANAGER | UC-38 |
| API-RP-04 | Nhập chi phí kỳ | POST | `/api/v1/reports/branch-period-costs` | Ghi số liệu phục vụ ước tính lợi nhuận | OWNER/ADMIN hoặc BRANCH_MANAGER được giao | UC-39 |
| API-RP-05 | Hiệu quả chi nhánh | GET | `/api/v1/reports/branch-performance` | Biên đóng góp ước tính và cảnh báo | OWNER/ADMIN; manager theo chi nhánh | UC-39 |
| API-RP-06 | Xuất báo cáo | GET | `/api/v1/reports/export` | Tải CSV theo quyền và bộ lọc | Vai trò có quyền xem báo cáo | UC-40 |


---

# ERD PlantUML

Nguồn: `05_ERD_PlantUML.puml`.

```plantuml
@startuml
title Hệ thống quản lý chuỗi cửa hàng trà sữa — ERD theo module
left to right direction
hide circle
skinparam shadowing false
skinparam linetype ortho
skinparam packageStyle rectangle
skinparam entity {
  BackgroundColor White
  BorderColor #64748B
  FontSize 11
}
skinparam ArrowColor #64748B

package "AU — Tài khoản & phân quyền" {
  entity "users" as users {
    * user_id : uuid <<PK>>
      email : varchar
      phone : varchar
      password_hash : varchar
      full_name : varchar
      role : enum
      status : enum
      created_at : timestamptz
      updated_at : timestamptz
  }
  entity "user_sessions" as user_sessions {
    * session_id : uuid <<PK>>
      user_id : uuid <<FK>>
      refresh_token_hash : varchar
      device_info : varchar
      expires_at : timestamptz
      revoked_at : timestamptz
  }
  entity "staff_assignments" as staff_assignments {
    * assignment_id : uuid <<PK>>
      user_id : uuid <<FK>>
      branch_id : uuid <<FK>>
      started_at : timestamptz
      ended_at : timestamptz
      status : enum
  }
  entity "audit_logs" as audit_logs {
    * audit_id : uuid <<PK>>
      actor_id : uuid? <<FK>>
      branch_id : uuid? <<FK>>
      action : varchar
      entity_type : varchar
      entity_id : uuid
      reason : text
      created_at : timestamptz
  }
}

package "BR — Chi nhánh, nhân sự & ca bán" {
  entity "branches" as branches {
    * branch_id : uuid <<PK>>
      branch_code : varchar
      name : varchar
      address : text
      operating_hours : json
      branch_status : enum
      online_acceptance : bool
      max_pending_online : int
      created_at : timestamptz
  }
  entity "delivery_zones" as delivery_zones {
    * zone_id : uuid <<PK>>
      branch_id : uuid <<FK>>
      zone_name : varchar
      ward_or_area_code : varchar
      active : bool
      fee_hint : decimal
  }
  entity "pos_terminals" as pos_terminals {
    * terminal_id : uuid <<PK>>
      branch_id : uuid <<FK>>
      code : varchar
      status : enum
  }
  entity "cash_shifts" as cash_shifts {
    * shift_id : uuid <<PK>>
      terminal_id : uuid <<FK>>
      cashier_id : uuid <<FK>>
      opened_at : timestamptz
      closed_at : timestamptz
      opening_cash : decimal
      status : enum
  }
  entity "branch_incidents" as branch_incidents {
    * incident_id : uuid <<PK>>
      branch_id : uuid <<FK>>
      reported_by : uuid? <<FK>>
      incident_type : enum
      offline_since : timestamptz
      recovered_at : timestamptz
      impact_level : enum
      action_taken : text
  }
}

package "CA — Sản phẩm, giá & công thức" {
  entity "products" as products {
    * product_id : uuid <<PK>>
      code : varchar
      name : varchar
      category : varchar
      description : text
      status : enum
  }
  entity "product_variants" as product_variants {
    * variant_id : uuid <<PK>>
      product_id : uuid <<FK>>
      size_code : varchar
      display_name : varchar
      active : bool
  }
  entity "option_groups" as option_groups {
    * option_group_id : uuid <<PK>>
      code : varchar
      name : varchar
      selection_mode : enum
      max_selection : int
  }
  entity "option_values" as option_values {
    * option_value_id : uuid <<PK>>
      option_group_id : uuid <<FK>>
      code : varchar
      display_name : varchar
      active : bool
  }
  entity "variant_option_groups" as variant_option_groups {
    * variant_id : uuid <<PK, FK>>
    * option_group_id : uuid <<PK, FK>>
      required : bool
      max_selection : int
  }
  entity "price_lists" as price_lists {
    * price_list_id : uuid <<PK>>
      branch_id : uuid? <<FK>>
      scope : enum
      channel : enum
      effective_from : timestamptz
      effective_to : timestamptz
      status : enum
      approved_by : uuid? <<FK>>
  }
  entity "price_list_items" as price_list_items {
    * price_item_id : uuid <<PK>>
      price_list_id : uuid <<FK>>
      variant_id : uuid? <<FK>>
      option_value_id : uuid? <<FK>>
      unit_price : decimal
  }
  entity "recipes" as recipes {
    * recipe_id : uuid <<PK>>
      variant_id : uuid? <<FK>>
      option_value_id : uuid? <<FK>>
      prepared_item_id : uuid? <<FK>>
      version_no : int
      output_quantity : decimal
      status : enum
      valid_from : timestamptz
      valid_to : timestamptz
      approved_by : uuid? <<FK>>
  }
  entity "recipe_components" as recipe_components {
    * component_id : uuid <<PK>>
      recipe_id : uuid <<FK>>
      item_id : uuid <<FK>>
      quantity : decimal
      unit : varchar
  }
  entity "branch_menu_availability" as branch_menu_availability {
    * branch_id : uuid <<PK, FK>>
    * variant_id : uuid <<PK, FK>>
      is_enabled : bool
      reason : text
      changed_at : timestamptz
  }
}

package "IN — Kho, sơ chế & cung ứng" {
  entity "inventory_items" as inventory_items {
    * item_id : uuid <<PK>>
      sku : varchar
      name : varchar
      item_type : enum
      base_unit : varchar
      shelf_life_hours : int
      storage_note : text
  }
  entity "branch_stock_policies" as branch_stock_policies {
    * branch_id : uuid <<PK, FK>>
    * item_id : uuid <<PK, FK>>
      safety_qty : decimal
      online_buffer_qty : decimal
      reorder_point : decimal
  }
  entity "suppliers" as suppliers {
    * supplier_id : uuid <<PK>>
      name : varchar
      contact_info : text
      status : enum
  }
  entity "purchase_orders" as purchase_orders {
    * purchase_id : uuid <<PK>>
      branch_id : uuid <<FK>>
      supplier_id : uuid <<FK>>
      created_by : uuid <<FK>>
      ordered_at : timestamptz
      expected_at : timestamptz
      status : enum
  }
  entity "purchase_order_lines" as purchase_order_lines {
    * purchase_line_id : uuid <<PK>>
      purchase_id : uuid <<FK>>
      item_id : uuid <<FK>>
      ordered_qty : decimal
      unit_price : decimal
  }
  entity "stock_lots" as stock_lots {
    * lot_id : uuid <<PK>>
      branch_id : uuid <<FK>>
      item_id : uuid <<FK>>
      supplier_id : uuid? <<FK>>
      source_lot_id : uuid? <<FK>>
      lot_code : varchar
      manufactured_at : timestamptz
      expires_at : timestamptz
      quality_status : enum
      unit_cost : decimal
  }
  entity "stock_balances" as stock_balances {
    * lot_id : uuid <<PK, FK>>
      qty_on_hand : decimal
      qty_reserved : decimal
      qty_locked : decimal
      updated_at : timestamptz
  }
  entity "stock_movements" as stock_movements {
    * movement_id : uuid <<PK>>
      lot_id : uuid <<FK>>
      actor_id : uuid <<FK>>
      movement_type : enum
      qty_delta : decimal
      source_kind : varchar
      source_id : uuid
      created_at : timestamptz
  }
  entity "goods_receipts" as goods_receipts {
    * receipt_id : uuid <<PK>>
      purchase_id : uuid <<FK>>
      received_by : uuid <<FK>>
      received_at : timestamptz
      status : enum
  }
  entity "goods_receipt_lines" as goods_receipt_lines {
    * receipt_line_id : uuid <<PK>>
      receipt_id : uuid <<FK>>
      purchase_line_id : uuid <<FK>>
      lot_id : uuid? <<FK>>
      accepted_qty : decimal
      rejected_qty : decimal
      rejection_reason : text
  }
  entity "stock_reservations" as stock_reservations {
    * reservation_id : uuid <<PK>>
      order_item_id : uuid <<FK>>
      lot_id : uuid <<FK>>
      reserved_qty : decimal
      status : enum
      created_at : timestamptz
      released_at : timestamptz
  }
  entity "preparation_batches" as preparation_batches {
    * prep_batch_id : uuid <<PK>>
      branch_id : uuid <<FK>>
      recipe_id : uuid <<FK>>
      output_lot_id : uuid? <<FK>>
      planned_yield : decimal
      actual_yield : decimal
      wasted_qty : decimal
      prepared_at : timestamptz
      status : enum
  }
  entity "preparation_inputs" as preparation_inputs {
    * prep_input_id : uuid <<PK>>
      prep_batch_id : uuid <<FK>>
      lot_id : uuid <<FK>>
      consumed_qty : decimal
  }
  entity "stock_transfers" as stock_transfers {
    * transfer_id : uuid <<PK>>
      source_branch_id : uuid <<FK>>
      target_branch_id : uuid <<FK>>
      requested_by : uuid <<FK>>
      approved_by : uuid? <<FK>>
      status : enum
      created_at : timestamptz
  }
  entity "stock_transfer_lines" as stock_transfer_lines {
    * transfer_line_id : uuid <<PK>>
      transfer_id : uuid <<FK>>
      item_id : uuid <<FK>>
      source_lot_id : uuid? <<FK>>
      target_lot_id : uuid? <<FK>>
      requested_qty : decimal
      shipped_qty : decimal
      received_qty : decimal
  }
  entity "stock_counts" as stock_counts {
    * stock_count_id : uuid <<PK>>
      branch_id : uuid <<FK>>
      opened_by : uuid <<FK>>
      opened_at : timestamptz
      closed_at : timestamptz
      status : enum
  }
  entity "stock_count_lines" as stock_count_lines {
    * count_line_id : uuid <<PK>>
      stock_count_id : uuid <<FK>>
      lot_id : uuid <<FK>>
      system_qty : decimal
      counted_qty : decimal
      reason : enum
      approved_by : uuid? <<FK>>
  }
}

package "OR — Đơn hàng, đặt online & pha chế" {
  entity "orders" as orders {
    * order_id : uuid <<PK>>
      branch_id : uuid <<FK>>
      customer_id : uuid? <<FK>>
      shift_id : uuid? <<FK>>
      client_order_key : varchar
      channel : enum
      fulfillment : enum
      status : enum
      subtotal : decimal
      discount_total : decimal
      final_total : decimal
      placed_at : timestamptz
      completed_at : timestamptz
  }
  entity "order_items" as order_items {
    * order_item_id : uuid <<PK>>
      order_id : uuid <<FK>>
      variant_id : uuid <<FK>>
      recipe_id : uuid <<FK>>
      quantity : int
      size_snapshot : varchar
      unit_price_snapshot : decimal
      line_total : decimal
      item_status : enum
  }
  entity "order_item_options" as order_item_options {
    * order_option_id : uuid <<PK>>
      order_item_id : uuid <<FK>>
      option_value_id : uuid <<FK>>
      recipe_id : uuid? <<FK>>
      extra_price_snapshot : decimal
      quantity : int
  }
  entity "order_status_events" as order_status_events {
    * order_event_id : uuid <<PK>>
      order_id : uuid <<FK>>
      actor_id : uuid? <<FK>>
      from_status : enum
      to_status : enum
      reason : text
      occurred_at : timestamptz
  }
  entity "kitchen_tickets" as kitchen_tickets {
    * ticket_id : uuid <<PK>>
      order_id : uuid <<FK>>
      queue_number : int
      status : enum
      printed_at : timestamptz
      ready_at : timestamptz
  }
  entity "order_deliveries" as order_deliveries {
    * delivery_id : uuid <<PK>>
      order_id : uuid <<FK>>
      receiver_name : varchar
      phone : varchar
      delivery_address : text
      delivery_fee : decimal
      delivery_status : enum
      partner_ref : varchar
  }
}

package "PA — Thanh toán, hoàn tiền & đối soát" {
  entity "payments" as payments {
    * payment_id : uuid <<PK>>
      order_id : uuid <<FK>>
      shift_id : uuid? <<FK>>
      method : enum
      amount : decimal
      status : enum
      provider_transaction_id : varchar
      paid_at : timestamptz
  }
  entity "refunds" as refunds {
    * refund_id : uuid <<PK>>
      payment_id : uuid <<FK>>
      order_id : uuid <<FK>>
      approved_by : uuid? <<FK>>
      amount : decimal
      reason : text
      status : enum
      refunded_at : timestamptz
  }
  entity "cash_shift_closures" as cash_shift_closures {
    * closure_id : uuid <<PK>>
      shift_id : uuid <<FK>>
      expected_cash : decimal
      counted_cash : decimal
      difference : decimal
      explanation : text
      approved_by : uuid? <<FK>>
      closed_at : timestamptz
  }
}

package "CR — Khách hàng, thành viên & ưu đãi" {
  entity "customers" as customers {
    * customer_id : uuid <<PK>>
      phone : varchar
      full_name : varchar
      phone_verified_at : timestamptz
      personal_data_consent_at : timestamptz
      marketing_opt_in : bool
      member_since : timestamptz
  }
  entity "customer_complaints" as customer_complaints {
    * complaint_id : uuid <<PK>>
      order_id : uuid <<FK>>
      customer_id : uuid? <<FK>>
      category : enum
      description : text
      evidence_url : varchar
      status : enum
      resolution : enum
      handled_by : uuid? <<FK>>
      resolved_at : timestamptz
  }
  entity "membership_tiers" as membership_tiers {
    * tier_id : uuid <<PK>>
      code : enum
      min_annual_spend : decimal
      point_multiplier : decimal
      active : bool
  }
  entity "customer_tier_history" as customer_tier_history {
    * tier_history_id : uuid <<PK>>
      customer_id : uuid <<FK>>
      tier_id : uuid <<FK>>
      qualified_spend : decimal
      evaluated_at : timestamptz
      valid_from : timestamptz
      valid_to : timestamptz
  }
  entity "loyalty_transactions" as loyalty_transactions {
    * point_txn_id : uuid <<PK>>
      customer_id : uuid <<FK>>
      order_id : uuid? <<FK>>
      txn_type : enum
      points_delta : int
      source_key : varchar
      expires_at : timestamptz
      approved_by : uuid? <<FK>>
      created_at : timestamptz
  }
  entity "promotions" as promotions {
    * promotion_id : uuid <<PK>>
      code : varchar
      name : varchar
      discount_type : enum
      discount_value : decimal
      min_order_quantity : int
      min_order_value : decimal
      stackable : bool
      budget_total : decimal
      starts_at : timestamptz
      ends_at : timestamptz
      status : enum
  }
  entity "promotion_branches" as promotion_branches {
    * promotion_id : uuid <<PK, FK>>
    * branch_id : uuid <<PK, FK>>
  }
  entity "vouchers" as vouchers {
    * voucher_id : uuid <<PK>>
      promotion_id : uuid <<FK>>
      customer_id : uuid? <<FK>>
      code : varchar
      max_redemptions : int
      redeemed_count : int
      expires_at : timestamptz
  }
  entity "order_promotions" as order_promotions {
    * applied_promo_id : uuid <<PK>>
      order_id : uuid <<FK>>
      promotion_id : uuid <<FK>>
      voucher_id : uuid? <<FK>>
      discount_amount : decimal
      applied_at : timestamptz
  }
}

package "SY — Gián đoạn vận hành & đồng bộ" {
  entity "sync_batches" as sync_batches {
    * sync_batch_id : uuid <<PK>>
      terminal_id : uuid <<FK>>
      submitted_by : uuid <<FK>>
      started_at : timestamptz
      finished_at : timestamptz
      status : enum
  }
  entity "sync_batch_orders" as sync_batch_orders {
    * sync_item_id : uuid <<PK>>
      sync_batch_id : uuid <<FK>>
      order_id : uuid? <<FK>>
      client_order_key : varchar
      payload_hash : varchar
      outcome : enum
      failure_reason : text
  }
}

package "RP — Báo cáo & đánh giá chi nhánh" {
  entity "branch_period_costs" as branch_period_costs {
    * cost_id : uuid <<PK>>
      branch_id : uuid <<FK>>
      period_start : date
      period_end : date
      rent_cost : decimal
      staff_cost : decimal
      utilities_cost : decimal
      other_cost : decimal
      recorded_by : uuid <<FK>>
  }
}

users ||--o{ user_sessions : user_id
users ||--o{ staff_assignments : user_id
branches ||--o{ staff_assignments : branch_id
users |o--o{ audit_logs : actor_id
branches |o--o{ audit_logs : branch_id
branches ||--o{ delivery_zones : branch_id
branches ||--o{ pos_terminals : branch_id
pos_terminals ||--o{ cash_shifts : terminal_id
users ||--o{ cash_shifts : cashier_id
branches ||--o{ branch_incidents : branch_id
users |o--o{ branch_incidents : reported_by
products ||--o{ product_variants : product_id
option_groups ||--o{ option_values : option_group_id
product_variants ||--o{ variant_option_groups : variant_id
option_groups ||--o{ variant_option_groups : option_group_id
branches |o--o{ price_lists : branch_id
users |o--o{ price_lists : approved_by
price_lists ||--o{ price_list_items : price_list_id
product_variants |o--o{ price_list_items : variant_id
option_values |o--o{ price_list_items : option_value_id
product_variants |o--o{ recipes : variant_id
option_values |o--o{ recipes : option_value_id
inventory_items |o--o{ recipes : prepared_item_id
users |o--o{ recipes : approved_by
recipes ||--o{ recipe_components : recipe_id
inventory_items ||--o{ recipe_components : item_id
branches ||--o{ branch_menu_availability : branch_id
product_variants ||--o{ branch_menu_availability : variant_id
branches ||--o{ branch_stock_policies : branch_id
inventory_items ||--o{ branch_stock_policies : item_id
branches ||--o{ purchase_orders : branch_id
suppliers ||--o{ purchase_orders : supplier_id
users ||--o{ purchase_orders : created_by
purchase_orders ||--o{ purchase_order_lines : purchase_id
inventory_items ||--o{ purchase_order_lines : item_id
branches ||--o{ stock_lots : branch_id
inventory_items ||--o{ stock_lots : item_id
suppliers |o--o{ stock_lots : supplier_id
stock_lots |o--o{ stock_lots : source_lot_id
stock_lots ||--o| stock_balances : lot_id
stock_lots ||--o{ stock_movements : lot_id
users ||--o{ stock_movements : actor_id
purchase_orders ||--o{ goods_receipts : purchase_id
users ||--o{ goods_receipts : received_by
goods_receipts ||--o{ goods_receipt_lines : receipt_id
purchase_order_lines ||--o{ goods_receipt_lines : purchase_line_id
stock_lots |o--o{ goods_receipt_lines : lot_id
order_items ||--o{ stock_reservations : order_item_id
stock_lots ||--o{ stock_reservations : lot_id
branches ||--o{ preparation_batches : branch_id
recipes ||--o{ preparation_batches : recipe_id
stock_lots |o--o{ preparation_batches : output_lot_id
preparation_batches ||--o{ preparation_inputs : prep_batch_id
stock_lots ||--o{ preparation_inputs : lot_id
branches ||--o{ stock_transfers : source_branch_id
branches ||--o{ stock_transfers : target_branch_id
users ||--o{ stock_transfers : requested_by
users |o--o{ stock_transfers : approved_by
stock_transfers ||--o{ stock_transfer_lines : transfer_id
inventory_items ||--o{ stock_transfer_lines : item_id
stock_lots |o--o{ stock_transfer_lines : source_lot_id
stock_lots |o--o{ stock_transfer_lines : target_lot_id
branches ||--o{ stock_counts : branch_id
users ||--o{ stock_counts : opened_by
stock_counts ||--o{ stock_count_lines : stock_count_id
stock_lots ||--o{ stock_count_lines : lot_id
users |o--o{ stock_count_lines : approved_by
branches ||--o{ orders : branch_id
customers |o--o{ orders : customer_id
cash_shifts |o--o{ orders : shift_id
orders ||--o{ order_items : order_id
product_variants ||--o{ order_items : variant_id
recipes ||--o{ order_items : recipe_id
order_items ||--o{ order_item_options : order_item_id
option_values ||--o{ order_item_options : option_value_id
recipes |o--o{ order_item_options : recipe_id
orders ||--o{ order_status_events : order_id
users |o--o{ order_status_events : actor_id
orders ||--o| kitchen_tickets : order_id
orders ||--o| order_deliveries : order_id
orders ||--o{ payments : order_id
cash_shifts |o--o{ payments : shift_id
payments ||--o{ refunds : payment_id
orders ||--o{ refunds : order_id
users |o--o{ refunds : approved_by
cash_shifts ||--o| cash_shift_closures : shift_id
users |o--o{ cash_shift_closures : approved_by
orders ||--o{ customer_complaints : order_id
customers |o--o{ customer_complaints : customer_id
users |o--o{ customer_complaints : handled_by
customers ||--o{ customer_tier_history : customer_id
membership_tiers ||--o{ customer_tier_history : tier_id
customers ||--o{ loyalty_transactions : customer_id
orders |o--o{ loyalty_transactions : order_id
users |o--o{ loyalty_transactions : approved_by
promotions ||--o{ promotion_branches : promotion_id
branches ||--o{ promotion_branches : branch_id
promotions ||--o{ vouchers : promotion_id
customers |o--o{ vouchers : customer_id
orders ||--o{ order_promotions : order_id
promotions ||--o{ order_promotions : promotion_id
vouchers |o--o{ order_promotions : voucher_id
pos_terminals ||--o{ sync_batches : terminal_id
users ||--o{ sync_batches : submitted_by
sync_batches ||--o{ sync_batch_orders : sync_batch_id
orders |o--o{ sync_batch_orders : order_id
branches ||--o{ branch_period_costs : branch_id
users ||--o{ branch_period_costs : recorded_by

legend right
  PK: Primary Key
  FK: Foreign Key
  || : một bản ghi cha
  o{ : 0..n bản ghi con
  o| : 0..1 bản ghi con
  ? trên kiểu dữ liệu biểu thị FK có thể NULL.
  Các cột còn lại NOT NULL trừ trường hợp có ghi chú riêng.
  Quan hệ sinh tự động từ khóa ngoại trong 03_DB_Du_lieu.md.
endlegend
@enduml
```
