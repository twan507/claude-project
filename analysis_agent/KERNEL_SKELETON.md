# KERNEL SKELETON

File index của project knowledge. Agent đọc file này đầu session để biết pack nào available và trigger nào activate pack nào. Chi tiết nội dung pack nằm trong file `_00` master của pack tương ứng, không duplicate vào đây.

## Cách dùng file này

1. Agent scan file này đầu session.
2. Khi query user đến, match với trigger của pack để quyết định activate pack nào.
3. Activate pack thì đọc `_00` master của pack đó trước (bắt buộc theo rule master-first reading trong system prompt mục 5.7).
4. Nếu không match pack nào, fall back về Default inline hoặc Default report (xem system prompt mục 6).
5. Pack không có trong file này = không tồn tại. Agent không được suy diễn pack ngoài danh sách.

## K — Knowledge packs

### K_agent_db

**Mục đích:** Knowledge base về dữ liệu chứng khoán Việt Nam trong MongoDB `agent_db`. Bao gồm schema 23 collection, query patterns, anti-patterns, methodology diễn giải chỉ báo (dòng tiền, technical zone, trend đa khung, PTCB theo 4 type doanh nghiệp), methodology phân tích tin tức 4 loại (doanh nghiệp, quốc tế, trong nước, thông cáo).

**Master:** `K_agent_db_00`

**Trigger:** Mọi query về cổ phiếu Việt Nam, thị trường VN, ticker, ngành, BCTC, dòng tiền, khối ngoại, technical, tin tức chứng khoán VN, hoặc khi cần số liệu định lượng từ `agent_db`.

**Depends:** Không có.

**Status:** Active.

## P — Process packs

### P_invest_memo

**Mục đích:** Workflow đầu tư cổ phiếu Việt Nam niêm yết, horizon 1-6 tháng, chỉ long, portfolio dưới 1 triệu USD. Pipeline 5 giai đoạn: gate vĩ mô → chọn 3-5 ngành → screen 6-10 mã/ngành → chấm điểm top 3/ngành → memo deep-dive 7 phần. Bổ sung song song: portfolio construction + monitoring & exit. Mỗi giai đoạn kết bằng checkpoint review 6 phần, chờ user confirm trước khi qua tier kế.

**Master:** `P_invest_memo_00`

**Trigger:** User mention tier hoặc giai đoạn cụ thể (ví dụ "tier 3", "giai đoạn 2", "gate vĩ mô"); yêu cầu "chạy quy trình đầu tư", "viết memo deep-dive nội bộ", "deep-dive mã X", "screen ngành", "shortlist", "portfolio construction", "monitoring danh mục", "rebalance".

**Lưu ý:** "memo" ở đây là conviction memo deep-dive nội bộ (tier 5C), KHÔNG phải báo cáo gửi khách hàng. Báo cáo gửi KH dùng `P_stock_pitch`.

**Depends:** `K_agent_db`.

### P_weekly_market

**Mục đích:** Sinh báo cáo thị trường tuần dạng deliverable MD 9-11 trang, dùng được cho cả họp nội bộ và gửi khách hàng. Báo cáo gồm 12 phần, mục tiêu (a) thống kê dữ liệu thị trường tuần đã qua và (b) đưa chiến lược giao dịch cho tuần tới. Workflow chia 2 stage, ngăn cách bằng 1 checkpoint duy nhất sau khi quyết regime + sector bias.

Wording dạng observation/luận điểm phân tích, không dùng từ command (mua/bán/giảm tỷ trọng). Sector bias diễn đạt "quan tâm/thận trọng". Watchlist đề xuất mã có cơ hội đầu tư qua luận điểm, không kèm level giá. Branding & disclaimer optional — user cung cấp ở pre-flight thì render branded version cho audience khách hàng.

Pack độc lập với `P_invest_memo` — không đọc state file invest cycle, không liên quan portfolio đang cầm. **Không sử dụng chỉ báo trend** (`market_snapshot.trend`, `industry_snapshot.trend`, `*_recent.recent_trend`) — đây là methodology nội bộ analyst, audience cuối (KH) không có nền để hiểu. Trend chỉ dùng trong `P_invest_memo`.

**Master:** `P_weekly_market_00`

**Trigger:** User yêu cầu "viết báo cáo tuần", "weekly market report", "báo cáo thị trường tuần", "tổng hợp thị trường tuần", "chiến lược tuần tới".

**Depends:** `K_agent_db`.

### P_stock_pitch

**Mục đích:** Sinh báo cáo khuyến nghị mua mã đơn lẻ phục vụ gửi trực tiếp cho khách hàng. Chỉ khuyến nghị MUA (long only), horizon 1-6 tháng. Báo cáo có 15 mục (cover → tóm tắt → foundation → 4-7 luận điểm flex → execution → bear case → kịch bản → disclaimer). MD là source of truth, render pptx khi user yêu cầu.

Workflow 5 stage, 2 checkpoint: checkpoint 1 sau Stage 2 review thesis + variant perception, checkpoint 2 sau Stage 4 review bear case + lo ngại còn yếu. Pack có thể ABORT ở checkpoint 1 hoặc 2 nếu thesis yếu hoặc conviction không đủ — không gửi KH sản phẩm half-baked.

Variant Perception bắt buộc explicit (3 câu: consensus sell-side / retail / thesis khác consensus). Bear case có "honest steelman" — bắt buộc 1-2 lo ngại còn yếu sau phản biện, không "all win". Ba kịch bản if-then trigger, không gán xác suất %.

Pack độc lập với `P_invest_memo` — user gọi lúc nào cũng được. Nếu user có memo tier 5C của mã, agent đọc và base trên thesis sẵn có. Không có thì build fresh từ DB. **Không khuyến nghị BÁN/GIỮ/short/derivative.** Không dùng chỉ báo trend (audience là KH, không có nền methodology để hiểu chỉ báo trend nội bộ).

**Master:** `P_stock_pitch_00`

**Trigger:** User yêu cầu "viết khuyến nghị [TICKER]", "recommendation [TICKER]", "báo cáo MUA [TICKER]", "báo cáo phân tích [TICKER] gửi khách hàng", "khuyến nghị mã [TICKER]", "stock pitch [TICKER]".

**Depends:** `K_agent_db`.

**Status:** Active.

## O — Output packs

### O_invest_memo

**Mục đích:** Render spec cho deliverable của pack `P_invest_memo` — báo cáo checkpoint tier 0-7, memo deep-dive 7 phần, portfolio plan, weekly review portfolio. Quy định structure rigid theo loại output, format MD/docx/pptx, K hygiene, citation, chart annotation.

**Master:** `O_invest_memo_00`

**Trigger:** Activate cùng với `P_invest_memo` khi user yêu cầu deliverable file (memo, report, pitch deck) hoặc render output theo style chuẩn của workflow đầu tư.

**Depends:** `P_invest_memo`, `K_agent_db`.

**Status:** Active.

### O_weekly_market

**Mục đích:** Render spec cho deliverable của pack `P_weekly_market` — báo cáo thị trường tuần 12 phần với structure rigid, heading exact. Quy định format MD (source of truth), docx (archive formal), pptx (meeting trình bày), K hygiene cho weekly market, mapping bảng/chart cho từng phần.

**Master:** `O_weekly_market_00`

**Trigger:** Activate cùng với `P_weekly_market` khi user yêu cầu báo cáo tuần thị trường.

**Depends:** `P_weekly_market`, `K_agent_db`.

**Status:** Active.

### O_stock_pitch

**Mục đích:** Render spec cho deliverable của pack `P_stock_pitch` — báo cáo khuyến nghị mã đơn lẻ 15 mục với structure rigid. MD là source of truth, render pptx 15 slide khi user yêu cầu (format chính cho gửi KH), docx khi user muốn archive formal. Quy định layout MD/pptx mỗi section với visual element bắt buộc (stat callout, bảng), K hygiene, mapping disclaimer footer.

Khi P pack abort ở checkpoint 1 hoặc 2, O pack KHÔNG render file — tránh sản phẩm half-baked có thể bị gửi nhầm cho KH.

**Master:** `O_stock_pitch_00`

**Trigger:** Activate cùng với `P_stock_pitch` khi user yêu cầu báo cáo khuyến nghị mã.

**Depends:** `P_stock_pitch`, `K_agent_db`.

**Status:** Active.

## Render binary — out of scope

Pack này dừng ở MD final. Render binary (pptx/docx/xlsx) là concern downstream, không thuộc scope. MD final do O pack xuất ra đã đủ structured (heading hierarchy + chart annotation YAML + citation + locale vi-VN) để consume bằng tool render bên ngoài.

## Naming convention tham chiếu

Pack tương lai đặt tên theo pattern (lặp lại từ system prompt mục 2 để agent tiện đối chiếu):

```
K_{domain}_{NN}              ví dụ K_realestate_00, K_macro_00
P_{flow_name}_{NN}           ví dụ P_quick_check_00, P_earnings_review_00
O_{format_or_style}_{NN}     ví dụ O_memo_docx_00, O_inline_chat_00
```

Số `NN` ý nghĩa nội bộ pack, quy định trong file `_00` của pack đó.

