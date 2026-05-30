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

**Mục đích:** Knowledge base về dữ liệu chứng khoán Việt Nam trong MongoDB `agent_db`. Bao gồm schema 25 collection, query patterns, anti-patterns, methodology diễn giải chỉ báo (dòng tiền, technical zone, trend đa khung, PTCB theo 4 type doanh nghiệp), methodology phân tích tin tức 4 loại (doanh nghiệp, quốc tế, trong nước, thông cáo).

**Master:** `K_agent_db_00`

**Trigger:** Mọi query về cổ phiếu Việt Nam, thị trường VN, ticker, ngành, BCTC, dòng tiền, khối ngoại, technical, tin tức chứng khoán VN, hoặc khi cần số liệu định lượng từ `agent_db`.

**Depends:** Không có.

**Status:** Active.

## P — Process packs

### P_invest_memo

**Mục đích:** Workflow đầu tư cổ phiếu Việt Nam niêm yết, horizon 1-6 tháng, chỉ long, portfolio dưới 1 triệu USD. Pipeline 5 giai đoạn: gate vĩ mô → chọn 3-5 ngành → screen 6-10 mã/ngành → chấm điểm top 3/ngành → memo deep-dive 7 phần. Bổ sung song song: portfolio construction + monitoring & exit. Mỗi giai đoạn kết bằng checkpoint review 6 phần, chờ user confirm trước khi qua tier kế.

**Master:** `P_invest_memo_00`

**Trigger:** User mention tier hoặc giai đoạn cụ thể (ví dụ "tier 3", "giai đoạn 2", "gate vĩ mô"); yêu cầu "chạy quy trình đầu tư", "viết memo deep-dive nội bộ", "deep-dive mã X", "screen ngành", "shortlist", "portfolio construction", "monitoring danh mục", "rebalance".

**Lưu ý:** "memo" ở đây là conviction memo deep-dive nội bộ (tier 5C), không phải broadcast tuần hay báo cáo chiến lược tháng.

**Depends:** `K_agent_db`.

**Status:** Active.

### P_weekly_overview

**Mục đích:** Sinh báo cáo **tổng quan thị trường tuần** dạng deliverable MD 9-11 trang, broadcast tuần độc lập (không cần thesis cycle nào), dùng được cho cả họp nội bộ và gửi khách hàng. Báo cáo gồm 12 phần rigid, mục tiêu (a) thống kê dữ liệu thị trường tuần qua và (b) đưa regime call + sector bias + watchlist cho tuần tới với conviction + horizon + disconfirming signal (chuẩn institutional buy-side).

Pack chia 5 file con: `_00` master + `_01` pre-flight + Stage 1 first half (phần 2-5) + `_02` Stage 1 second half (phần 6-9 với phần 9 fundamental-driven 3 kịch bản) + `_03` checkpoint + Stage 2 (phần 10-12 + phần 1 Key calls/Watch/Risk) + `_04` methodology + self-audit + edge + contract. Workflow 2 stage, ngăn cách 1 checkpoint sau khi quyết regime + sector bias.

**Philosophy fundamental-driven:** 3 kịch bản phần 9 trigger PRIMARY là vĩ mô/cơ bản/chính sách/catalyst, technical chỉ confirmation phụ ≤30%. Cap technical toàn báo cáo ≤15%. Whitelist 18 ngành áp dụng default; user yêu cầu override được cho ngành ngoài whitelist. Mỗi call có conviction HIGH/MID/LOW + horizon 1-2 tuần / 2-4 tuần + 1-2 disconfirming signal.

Pack độc lập với `P_invest_memo` và `P_vbse_strategy` — không đọc state file invest cycle hay thesis monthly. **Không sử dụng chỉ báo trend nội bộ** (`market_snapshot.trend`, `industry_snapshot.trend`, `*_recent.recent_trend`) — audience có thể là KH.

**Master:** `P_weekly_overview_00`

**Trigger:** User yêu cầu "viết báo cáo tuần", "weekly overview report", "báo cáo tổng quan thị trường tuần", "tổng quan tuần", "broadcast tuần".

**Depends:** `K_agent_db`.

**Status:** Active.

### P_vbse_strategy

**Mục đích:** Sinh báo cáo **chiến lược đầu tư VBSE** VN theo 2 chu kỳ lồng nhau — báo cáo tháng (parent, đầu tháng, hình thành thesis) và báo cáo tuần (child, tracking trong tháng, đọc lại monthly để cập nhật). Horizon 1-3 tháng forward-looking. Khác `P_weekly_overview` (đó là broadcast tổng quan tuần độc lập); pack này là **định vị chiến lược deep nội bộ** — VN đang ở đâu trong chu kỳ vĩ mô + chu kỳ thanh khoản + chu kỳ định giá, theme nào chi phối, sector nào ưu tiên, kịch bản nào dự phòng, mã nào đại diện theme.

Khung tư duy 6 trục cốt lõi: (1) môi trường vĩ mô & tài chính, (2) định vị thị trường VN, (3) themes & narratives, (4) sector allocation, (5) kịch bản & risk map, (6) high-conviction watchlist 2-phase (Phase 1 Screen cơ bản + Phase 2 Bucket entry). **Structure flex** — sub-section trong từng trục, độ sâu, số theme/sector/mã linh hoạt theo phát hiện thực tế. Không ép số rigid như `P_weekly_overview`.

**Weight balance — fundamental supremacy:** báo cáo tháng (horizon 1-3, 3-6 tháng) phải dùng signal **vĩ mô + cơ bản + chính sách + catalyst dài hơi làm PRIMARY (~70-75%)** — đây là factor có time-to-play-out khớp horizon. **Định vị thị trường + flow là SECONDARY (~15-20%)**. **Technical là TERTIARY (~10-15%) — chỉ tồn tại hợp pháp ở Phase 2 Bucket entry Trục 6** (phân Bucket 1/2/3 cho mã đã chọn bằng cơ bản). KHÔNG làm confirmation timing tổng quát; KHÔNG quyết định regime/theme/sector/risk. Trục 5 kịch bản trigger primary BẮT BUỘC là macro/fundamental/policy/catalyst (technical = 0% — cấm tuyệt đối). Weekly mode áp **Technical-as-noise rule** (`_08` mục 4): status Shift/Materialize bắt buộc kèm signal vĩ mô/cơ bản/chính sách; technical shift đơn độc = noise tạm thời, status Hold. Ngoại lệ duy nhất: rebucket entry watchlist.

**Chuẩn institutional output:** mỗi theme/sector/mã đều có **conviction level** (HIGH/MID/LOW) + **time horizon** (1m / 1-3m / 3-6m) + **disconfirming signals** ("what would change our mind" — reference field data cụ thể). Trục 4 có sector tilts consolidated table chuẩn buy-side. Trục 6 watchlist có ADV tháng cho liquidity awareness. Review N-1 có Best call / Worst call honesty attribution. Disclaimer có forward-looking statement chuẩn institutional.

**User overlay (PM input):** user có thể inject view ở 3 channel — pre-flight, mid-flow (interrupt session), checkpoint override. Agent xử lý theo matrix 5 trạng thái (Confirm / Partial / Conflict / Flag / Out of scope), không silently override agent finding bằng view user. Báo cáo cuối có badge inline + User overlay log table trong metadata làm audit trail.

**Stage 0 evaluation (đánh giá chiến lược cũ):** cả monthly và weekly mode có optional Stage 0 — agent đọc file báo cáo cũ (N-1 hoặc W-1) user upload, cross-check thesis với actual data từ `agent_db` (giá / dòng tiền / BCTC / vĩ mô), compose eval block 6 phần (monthly) hoặc 4 phần (weekly), present tại Checkpoint 0, user accept / refine / skip carry-forward trước khi build cycle mới. Lưu ý: DB không có collection storage cho báo cáo cũ — chỉ user upload file MD trong session.

**Monthly mode:** workflow 4 stage + 2 checkpoint (Checkpoint 0 sau Stage 0 eval, Checkpoint 1 sau Stage 1 regime/themes). Output 8-12 trang.

**Weekly update mode:** workflow 2 stage + HARD GATE pre-flight. HARD GATE bắt buộc: agent compute hôm nay → tuần thứ [N] của tháng [M/YYYY], hỏi user có monthly active đúng tháng chưa. **Không có monthly → REFUSE chạy weekly**, đề xuất 3 path (chạy monthly trước / dùng `P_weekly_overview` / override với note decay). Header báo cáo bắt buộc ghi "Tuần [N] của tháng [M/YYYY]" + link file monthly tham chiếu. Mỗi trục có status Hold / Shift / Materialize; watchlist refresh 4 trạng thái (Hold / Watch closely / Out / Vào mới); 1-2 action item định tính. Output 3-5 trang.

Wording observation/luận điểm, không command (mua/bán/giảm tỷ trọng/stop loss). Watchlist không entry/stop/target/size — chỉ luận điểm theme + signal theo dõi + disconfirming signal + ADV + Bucket entry (1/2/3). Kịch bản if-then trigger, không % xác suất. Branding & disclaimer optional, render branded khi user cung cấp ở pre-flight.

**Triết lý fundamental supremacy + 2-phase watchlist:** Phase 1 Screen cơ bản-only (cấm PTKT), Phase 2 Bucket entry PTKT-driven. Cap technical toàn báo cáo ≤15%; Trục 2 ≤20%; Trục 4, 5, Phase 1 Trục 6 ≈ 0% (cấm tuyệt đối ở Risk + Screen).

Pack chia 10 file con: `_00` master + `_01..06` mỗi trục một file + `_07` Workflow Monthly + `_08` Workflow Weekly + `_09` Overlay + Self-audit + Edge + Contract. Pack độc lập với `P_invest_memo` và `P_weekly_overview` — không share state. Watchlist ở pack này là theme play observation + bucket entry timing, khác bản chất với portfolio deep-dive của `P_invest_memo`.

**Master:** `P_vbse_strategy_00`

**Trigger:**
- Monthly: "báo cáo chiến lược tháng", "monthly strategy", "outlook tháng [N]", "chiến lược đầu tư tháng", "định vị thị trường tháng [N]", "vbse strategy monthly"
- Weekly update: "update tuần [DD/MM] chiến lược", "weekly strategy update", "cập nhật tuần báo cáo tháng [N]", "weekly check chiến lược", "vbse strategy weekly"

**Depends:** `K_agent_db`.

**Status:** Active.

## O — Output packs

### O_invest_memo

**Mục đích:** Render spec cho deliverable của pack `P_invest_memo` — báo cáo checkpoint tier 0-7, memo deep-dive 7 phần, portfolio plan, weekly review portfolio. Quy định structure rigid theo loại output, format MD/docx/pptx, K hygiene, citation, chart annotation.

**Master:** `O_invest_memo_00`

**Trigger:** Activate cùng với `P_invest_memo` khi user yêu cầu deliverable file (memo, report, pitch deck) hoặc render output theo style chuẩn của workflow đầu tư.

**Depends:** `P_invest_memo`, `K_agent_db`.

**Status:** Active.

### O_weekly_overview

**Mục đích:** Render spec cho deliverable của pack `P_weekly_overview` — báo cáo tổng quan thị trường tuần 12 phần rigid, heading exact. Quy định format MD (source of truth), docx (archive formal), pptx (meeting trình bày), 3 mode branding (custom / default branded / plain), K hygiene, mapping bảng/chart cho từng phần, conviction marker + horizon + disconfirming bắt buộc cho mỗi call (regime / sector bias / watchlist mã).

**Master:** `O_weekly_overview_00`

**Trigger:** Activate cùng với `P_weekly_overview` khi user yêu cầu báo cáo tổng quan thị trường tuần.

**Depends:** `P_weekly_overview`, `K_agent_db`.

**Status:** Active.

### O_vbse_strategy

**Mục đích:** Render spec cho deliverable của pack `P_vbse_strategy` — báo cáo chiến lược đầu tư VBSE VN, 2 mode (monthly parent + weekly update child). **Flex structure** thay vì rigid: 6 trục H2 cố định nhưng sub-section H3 và độ sâu flex theo phát hiện thực tế; trục Hold rút gọn 3-5 dòng, trục signal mạnh đào sâu không giới hạn. Quy định header (plain / branded), layout từng trục với fundamental-first vocabulary, format watchlist (Phase 1 Screen cơ bản + Phase 2 Bucket entry observation, không level giá), checkpoint block monthly (CP0 + CP1), status badge weekly (Hold / Shift / Materialize) với technical-as-noise rule, watchlist refresh 4 trạng thái + rebucket section, disclaimer 3 trường hợp, K hygiene, metadata mỗi mode + User overlay log table.

**Master:** `O_vbse_strategy_00`

**Trigger:** Activate cùng với `P_vbse_strategy` khi user yêu cầu báo cáo chiến lược tháng hoặc weekly update.

**Depends:** `P_vbse_strategy`, `K_agent_db`.

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

