# P_vbse_strategy_03 — Trục 3: Themes & narratives chính

File chi tiết Trục 3 của khung 6 trục pack `P_vbse_strategy`. Phụ thuộc master `P_vbse_strategy_00` — đọc trước để nắm triết lý thiết kế, scope monthly/weekly, và **Weight balance** (mục 4 của master). Theo Weight balance: **Trục 3 thuộc tầng PRIMARY** (cùng nhóm với Trục 1 và một phần Trục 4), **technical cap ≤ 5%** — themes phải xây trên cơ chế vĩ mô / chính sách / cơ bản / catalyst, không phải break-out kỹ thuật.

## 1. Mục tiêu & câu hỏi cốt lõi

Trong 1-3 tháng tới, **2-5 câu chuyện lớn** nào sẽ chi phối dòng tiền thị trường VN?

## 2. Lăng kính phân tích

- **Chính sách trong nước:** dự thảo luật/nghị quyết/quyết định mới (vd cải cách thuế, sửa luật chứng khoán, gói hỗ trợ tài khoá), tiến độ đầu tư công, cải cách doanh nghiệp nhà nước
- **Sự kiện vĩ mô lớn sắp đến:** họp NHNN, FOMC, ECB; release CPI/GDP/PMI; bầu cử; geopolitics
- **Mùa BCTC:** đang trong/sắp vào mùa quý nào, sector nào kỳ vọng tăng trưởng nổi bật, sector nào áp lực
- **Tái cấu trúc ngành:** M&A, divestment, niêm yết mới, upgrade thị trường (FTSE/MSCI)
- **Catalyst commodity:** chu kỳ giá hàng hoá đang chuyển pha (vd dầu vào downcycle, thép vào upcycle), tác động chuỗi giá trị
- **Catalyst quốc tế:** chính sách Trump/Tập/EU tác động VN, supply chain shift, China stimulus

## 3. Spec mỗi theme — 5 thành phần bắt buộc

Mỗi theme có tên gọi rõ ràng (vd "Sóng đầu tư công Q2", "Margin ngân hàng cải thiện cuối chu kỳ hạ lãi suất", "Phục hồi xuất khẩu thuỷ sản theo USD/VND"). Bắt buộc đủ 5 thành phần:

1. **Cơ chế tác động** — mạch logic nguyên nhân → hệ quả (3-5 dòng)
2. **Conviction level — HIGH / MID / LOW:**
   - **HIGH** = cơ chế rõ + catalyst đã hoặc đang materialize + cross-check ≥2 trục khác đồng thuận
   - **MID** = cơ chế rõ + catalyst chưa rõ thời điểm, hoặc 1 trục khác chưa đồng thuận
   - **LOW** = early thesis, signals còn yếu, mang tính "watch list" theme

   **Conviction CAP rules — áp tuyệt đối (chống bias HIGH conviction inflation):**
   - Theme contradicting Trục 1 macro regime → cap LOW
   - Theme không có catalyst với ngày cụ thể trong horizon → cap MID (HIGH yêu cầu ngày)
   - Theme evidence chỉ từ < 2 trục → cap MID
   - Theme = consensus crowded (sell-side đã call rộng + media coverage cao) → cap MID + flag "alpha limited, consensus crowded"
   - Bear regime mode active (xem `_06` mục 5: Trục 1 macro negative + Trục 2 định vị "phân phối/suy yếu") → cap MID toàn pack, không HIGH bất kể trục đồng thuận
3. **Time horizon** — 1 tháng / 1-3 tháng / 3-6 tháng (theo timing catalyst materialize)
4. **Catalyst trigger** — sự kiện / mức số / chính sách cụ thể đã hoặc sắp xảy ra; nếu có ngày dự kiến, ghi ngày
5. **Disconfirming signals** — 2-3 chỉ báo cụ thể (reference data field) mà nếu xuất hiện sẽ invalidate theme. Vd "Industry rank ngành thép tụt khỏi top 8 trong 2 tuần liên tiếp", "USD/VND vượt 26500 + can thiệp NHNN", "Brent về dưới 65 USD/thùng kéo 1 tháng". Đây là "what would change our mind" — chuẩn institutional research.

## 4. Hướng tìm dữ liệu

| Nguồn | Mục đích | Ghi chú |
|---|---|---|
| `news_history_feed` filter `news_type` | Chính sách trong nước + tin lớn — rolling 30 ngày | PRIMARY cho lăng kính "chính sách" và "tái cấu trúc ngành" |
| `news_history_content` | Đọc sâu tin quan trọng — verify cơ chế tác động | Khi 1 tin có khả năng chi phối thesis tháng |
| `other_data` group `commodities.*` | Commodity cycle — xu hướng giá hàng hoá | PRIMARY cho lăng kính "catalyst commodity" |
| Web search **HEAVY** | Forward catalyst, sự kiện vĩ mô sắp đến, lịch FOMC/ECB, M&A pipeline, niêm yết mới, FTSE/MSCI upgrade timeline, chính sách quốc tế (Trump/Tập/EU), supply chain shift, China stimulus | Forward-looking phần lớn không có trong DB |
| File user upload (optional) | Theme user gợi ý / brief mảng quan tâm | Nếu user upload |

**Tỷ trọng nguồn (per master index):** ~35% DB + ~60% web + ~5% file user upload. Themes vốn forward-looking nên web search là kênh chính cho catalyst chưa materialize.

## 5. Output diễn giải

2-5 themes với tên gọi rõ ràng, mỗi theme bắt buộc đủ 5 thành phần (Cơ chế / Conviction / Horizon / Catalyst trigger / Disconfirming signals) như mục 3. Output feed sang Trục 4 (mapping theme → ngành hưởng lợi, tham chiếu `K_agent_db_01 Section B` whitelist 18 ngành), Trục 5 (theme-specific risk), và Trục 6 (chọn mã đại diện theme cho watchlist).

## 6. Edge cases

- **Tháng "không có theme dominant":** ghi explicit "tháng N không có theme chi phối rõ, thị trường trong giai đoạn waiting/transitional, ưu tiên defensive sector". **KHÔNG bịa theme** chỉ để lấp section.
- **Theme bị priced-in sớm:** nếu giá đã chạy trước catalyst materialize → giảm conviction từ HIGH xuống MID/LOW, ghi rõ "thesis correct nhưng entry timing đã muộn".
- **Catalyst delay sang tháng N+1:** không xoá theme khỏi báo cáo — carry-forward sang tháng sau với note "catalyst dự kiến tháng N nhưng đã delay, tracking lại trong báo cáo tháng N+1". Cập nhật signal theo dõi trong báo cáo weekly child.
