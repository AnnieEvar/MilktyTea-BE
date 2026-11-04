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

