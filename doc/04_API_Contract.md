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

