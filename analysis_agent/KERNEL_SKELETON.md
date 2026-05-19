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

### P_invest_strategy

**Mục đích:** Sinh báo cáo **chiến lược đầu tư** VN theo 2 chu kỳ lồng nhau — báo cáo tháng (parent, đầu tháng, hình thành thesis) và báo cáo tuần (child, tracking trong tháng, đọc lại monthly để cập nhật). Horizon 1-3 tháng forward-looking. Khác `P_weekly_market` (đó là báo cáo tổng hợp thông tin); pack này là **định vị chiến lược** — thị trường VN đang ở đâu trong chu kỳ vĩ mô + chu kỳ thanh khoản + chu kỳ định giá, theme nào chi phối, sector nào ưu tiên, kịch bản nào dự phòng, mã nào đại diện theme.

Khung tư duy 6 trục cốt lõi: (1) môi trường vĩ mô & tài chính, (2) định vị thị trường VN, (3) themes & narratives, (4) sector allocation, (5) kịch bản & risk map, (6) high-conviction watchlist. **Structure flex** — sub-section trong từng trục, độ sâu, số theme/sector/mã linh hoạt theo phát hiện thực tế. Không ép số rigid như `P_weekly_market`.

**Chuẩn institutional output:** mỗi theme/sector/mã đều có **conviction level** (HIGH/MID/LOW) + **time horizon** (1m / 1-3m / 3-6m) + **disconfirming signals** ("what would change our mind" — reference field data cụ thể). Trục 4 có sector tilts consolidated table chuẩn buy-side. Trục 6 watchlist có ADV tháng cho liquidity awareness. Review N-1 có Best call / Worst call honesty attribution. Disclaimer có forward-looking statement chuẩn institutional.

**User overlay (PM input):** user có thể inject view ở 3 channel — pre-flight, mid-flow (interrupt session), checkpoint override. Agent xử lý theo matrix 5 trạng thái (Confirm / Partial / Conflict / Flag / Out of scope), không silently override agent finding bằng view user. Báo cáo cuối có badge inline + User overlay log table trong metadata làm audit trail.

**Stage 0 evaluation (đánh giá chiến lược cũ):** cả monthly và weekly mode có optional Stage 0 — agent đọc file báo cáo cũ (N-1 hoặc W-1) user upload, cross-check thesis với actual data từ `agent_db` (giá / dòng tiền / BCTC / vĩ mô), compose eval block 6 phần (monthly) hoặc 4 phần (weekly), present tại Checkpoint 0, user accept / refine / skip carry-forward trước khi build cycle mới. Lưu ý: DB không có collection storage cho báo cáo cũ — chỉ user upload file MD trong session.

**Monthly mode:** workflow 4 stage + 2 checkpoint (Checkpoint 0 sau Stage 0 eval, Checkpoint 1 sau Stage 1 regime/themes). Output 8-12 trang.

**Weekly update mode:** workflow 2 stage + HARD GATE pre-flight. HARD GATE bắt buộc: agent compute hôm nay → tuần thứ [N] của tháng [M/YYYY], hỏi user có monthly active đúng tháng chưa. **Không có monthly → REFUSE chạy weekly**, đề xuất 3 path (chạy monthly trước / dùng `P_weekly_market` / override với note decay). Header báo cáo bắt buộc ghi "Tuần [N] của tháng [M/YYYY]" + link file monthly tham chiếu. Mỗi trục có status Hold / Shift / Materialize; watchlist refresh 4 trạng thái (Hold / Watch closely / Out / Vào mới); 1-2 action item định tính. Output 3-5 trang.

Wording observation/luận điểm, không command (mua/bán/giảm tỷ trọng/stop loss). Watchlist không entry/stop/target/size — chỉ luận điểm theme + signal theo dõi + disconfirming signal + ADV. Kịch bản if-then trigger, không % xác suất. Branding & disclaimer optional, render branded khi user cung cấp ở pre-flight.

Pack độc lập với `P_invest_memo`, `P_weekly_market`, `P_stock_pitch` — không share state. Watchlist ở pack này là theme play observation, khác bản chất với portfolio deep-dive của `P_invest_memo`.

**Master:** `P_invest_strategy_00`

**Trigger:**
- Monthly: "báo cáo chiến lược tháng", "monthly strategy", "outlook tháng [N]", "chiến lược đầu tư tháng", "định vị thị trường tháng [N]"
- Weekly update: "update tuần [DD/MM] chiến lược", "weekly strategy update", "cập nhật tuần báo cáo tháng [N]", "weekly check chiến lược"

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

### O_invest_strategy

**Mục đích:** Render spec cho deliverable của pack `P_invest_strategy` — báo cáo chiến lược đầu tư VN, 2 mode (monthly parent + weekly update child). **Flex structure** thay vì rigid: 6 trục H2 cố định nhưng sub-section H3 và độ sâu flex theo phát hiện thực tế của tháng/tuần; trục Hold rút gọn 3-5 dòng, trục signal mạnh đào sâu không giới hạn. Quy định header (plain / branded), layout từng trục, format watchlist (observation only, không level giá), checkpoint block monthly, status badge weekly (Hold / Shift / Materialize), watchlist refresh 4 trạng thái, disclaimer 3 trường hợp (custom / default branded / plain), K hygiene, metadata mỗi mode.

**Master:** `O_invest_strategy_00`

**Trigger:** Activate cùng với `P_invest_strategy` khi user yêu cầu báo cáo chiến lược tháng hoặc weekly update.

**Depends:** `P_invest_strategy`, `K_agent_db`.

**Status:** Active.

## Render binary (pptx / docx / xlsx)

MD final là source of truth. Khi user yêu cầu render binary, agent chạy theo workflow ở `system_prompt.md` mục 4 "Render binary — workflow": xác định style qua (a) O pack render spec, (b) branding info pre-flight, (c) user explicit; nếu không rõ thì hỏi clarify. **Body font chốt: Roboto** (fallback Roboto → Open Sans → Arial). Binary derive từ MD final, không edit độc lập — sửa nội dung phải sửa MD trước rồi re-render.

## Naming convention tham chiếu

Pack tương lai đặt tên theo pattern (lặp lại từ system prompt mục 2 để agent tiện đối chiếu):

```
K_{domain}_{NN}              ví dụ K_realestate_00, K_macro_00
P_{flow_name}_{NN}           ví dụ P_quick_check_00, P_earnings_review_00
O_{format_or_style}_{NN}     ví dụ O_memo_docx_00, O_inline_chat_00
```

Số `NN` ý nghĩa nội bộ pack, quy định trong file `_00` của pack đó.

