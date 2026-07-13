# P_vbse_strategy_09 — User overlay + Self-audit + Edge cases + Output contract

File meta của pack `P_vbse_strategy` — quy tắc xử lý view user inject vào báo cáo, self-audit checklist trước render, edge cases cross-cutting, và output contract cho cả 2 mode. Dependency: `P_vbse_strategy_00` master.

## 1. User overlay handling — Ghép nối quan điểm user vào báo cáo

Báo cáo chiến lược là sản phẩm hợp tác analyst + PM. Pack cho phép user inject view ở **3 channel** + có methodology chuẩn để agent xử lý.

### 1.1. Three channel injection

1. **Pre-flight injection** — user nêu view từ đầu trong câu pre-flight (monthly câu 4, weekly câu 5; xem `P_vbse_strategy_07` mục 2 + `P_vbse_strategy_08` mục 2). Agent build báo cáo có tính đến view này.
2. **Mid-flow injection** — user có thể interrupt bất cứ lúc nào trong session để add view. Agent dừng turn hiện tại, tiếp nhận, xử lý qua matrix mục 1.2, rồi tiếp tục từ điểm dừng.
3. **Checkpoint injection** — user override / refine / bổ sung tại checkpoint (CP0 monthly Stage 0 eval, CP1 monthly Regime+themes, CP0 weekly Stage 0 eval).

### 1.2. Synthesis matrix — agent xử lý view user thế nào

Mỗi view user inject, agent chạy 3 bước:

**Bước 1 — Parse view:** xác định view thuộc trục nào (1-6) và bản chất gì:
- Factual claim (số liệu hoặc sự kiện cụ thể)
- Predictive thesis (dự đoán xu hướng)
- Risk concern (rủi ro user nhìn thấy)
- Theme suggestion (theme user đề xuất)
- Ticker idea (mã user nêu)

**Bước 2 — Cross-check với data agent_db + web:** agent query data hiện có để verify view.

**Bước 3 — Phân loại + xử lý theo matrix:**

| Trạng thái | Tình huống | Cách integrate vào báo cáo |
|---|---|---|
| **Confirm** | Data agent_db hoặc web search xác nhận view user | Integrate view vào trục liên quan như evidence chính. Render trong báo cáo kèm badge `[Synthesized from PM input + data confirm]` và cite data field. |
| **Partial confirm** | Một phần view có data ủng hộ, một phần không | Render cả 2 phần: phần data confirm → integrate; phần không có data → ghi rõ "PM view, data chưa confirm — cần monitor [signal cụ thể]". Badge `[Mixed: PM input + partial data]`. |
| **Conflict** | Data có sẵn ngược chiều view user | KHÔNG silently override. Present cả 2 view trong trục liên quan: "PM view: [X]. Agent finding from data: [Y]. Tension chưa resolve, đề xuất user quyết." Badge `[Conflict: PM view ↔ data]`. Đưa vào checkpoint nếu là monthly + chưa qua checkpoint. |
| **No data** | Không có data để verify (vd "tôi nghe tin nội bộ về M&A") | Ghi nhận như standalone "PM flag" — không treat như fact đã verify. Trong báo cáo: render trong trục liên quan với badge `[PM flag — chưa có data verify]`. Vẫn integrate vào risk map hoặc themes "watching" tier (LOW conviction). |
| **Out of scope** | View không thuộc 6 trục (vd "tôi muốn add panel mới về crypto") | Báo user view ngoài scope pack hiện tại, đề xuất ghi nhận như note phụ chân báo cáo, hoặc gợi ý pack/workflow khác phù hợp hơn. |

### 1.3. Audit trail trong báo cáo cuối

Pack bắt buộc render audit trail cho user contribution:

- **Trong nội dung báo cáo:** mỗi điểm tích hợp view user có inline badge (như mục 1.2)
- **Trong metadata cuối:** mục "User overlay log" liệt kê tất cả view user đã inject + trạng thái xử lý (Confirm / Partial / Conflict / Flag / Out of scope)
- **Trong executive summary:** nếu có view user ở conviction HIGH hoặc Conflict chưa resolve, mention 1 dòng để leadership đọc

### 1.4. Quy tắc bất di bất dịch

- **Không silently override agent finding bằng user view.** Cả 2 phải xuất hiện trong báo cáo có thể truy được.
- **Không treat "PM flag" (no data) như fact verified.** Phải có badge phân biệt.
- **Không bỏ qua user view vì agent thấy không hợp lý.** Nếu disagree, present tension, để user quyết.
- **Không edit retroactively báo cáo đã render** khi user inject view muộn — render version mới với log thay đổi, hoặc note "addendum" cuối file.

## 2. Self-audit trước khi xuất file — Monthly mode

Chạy checklist trước khi render. Vi phạm câu nào sửa rồi mới render.

### 2.1. Cấu trúc & checkpoint

1. 6 trục đầy đủ (trục lướt nhanh vẫn có 3-5 dòng kết luận)?
2. Nếu user chọn Stage 0 (eval N-1): có checkpoint 0 đã user phản hồi? Eval block đủ 6 phần (Regime/Themes/Sectors/Watchlist/Risk/Calibration)? Carry-forward đã feed vào Stage 1?
3. Checkpoint 1 regime + themes đã có user phản hồi (confirm/override)?

### 2.2. Weight balance & technical caps (BẤT BIẾN — theo `_00` mục 4)

4. **Cap technical toàn báo cáo (trừ Phase 2 Bucket entry Trục 6) ≤ 15%?**
5. **Cap technical Trục 2 ≤ 20%?** Câu chốt định vị mở đầu bằng PRIMARY (định giá phân vị + dòng tiền aggregate + FII + breadth), KHÔNG bằng technical?
6. **Cap technical Trục 4 ≤ 5%?** Bảng sector tilts KHÔNG có cột technical_zone làm decision factor?
7. **Cap technical Trục 5 = 0%?** Mọi trigger 3 kịch bản + risk map là macro/fundamental/policy/catalyst? Technical chỉ confirmation phụ (nếu có)?
8. **Cap technical Phase 1 Screen Trục 6 = 0%?** Không có mã nào bị loại universe vì technical yếu?
9. **Phase 2 Bucket entry Trục 6 dùng đúng định nghĩa pack-internal ở `_06` mục 4.1?** Bucket KHÔNG nâng/giảm conviction theme? Tier 2 KHÔNG có mã ở Bucket 1?

### 2.3. Whitelist 18 ngành (Nguyên tắc 3 master)

10. Mọi query / aggregate ngành dùng 18 ngành whitelist (xem `K_agent_db_01` Section B)?
11. Bảng sector tilts (Trục 4) render đủ 18 ngành, không có ngành ngoài whitelist?
12. Aggregate dòng tiền cấp thị trường (Trục 2) tính trên 18 ngành, không 24?
13. Rank ngành đã **tự tổng hợp** theo `week_score` qua 18 ngành whitelist (DB không lưu `industry_rank` tĩnh — xem `K_agent_db_01` mục "Xếp hạng ngành")?
14. Watchlist Trục 6 chỉ có mã thuộc 18 ngành whitelist?

### 2.4. Nội dung từng trục

15. Trục 3 — mỗi theme có đủ 5 thành phần (cơ chế / conviction / horizon / catalyst / disconfirming signals)?
16. Trục 4 — bảng sector tilts tổng hợp với conviction + driver + disconfirming signal mỗi ngành?
17. Trục 5 — 3 kịch bản dùng if-then trigger, **không có % xác suất**? Risk map mỗi rủi ro đủ 4 thành phần (bản chất / signal materialize / phản ứng định tính / theme bị invalidate)?
18. Trục 6 — watchlist 20 mã (10 Tier 1 + 10 Tier 2 mặc định; điều chỉnh theo regime), mỗi mã đủ 6 thành phần + Tier marker + ADV tháng + Bucket entry? Không có level giá vào/ra/stop?
18a. **Tier 1** (10 mã, hoặc 8 Bear regime / 6 quá mua): cơ bản strong + catalyst rõ với ngày + ADV ≥ 5 tỷ + Variant Perception statement mỗi mã?
18b. **Tier 2** (10 mã, hoặc 12 Bear regime / 14 quá mua): cơ bản clean + technical bottom-fishing setup + ADV ≥ 3 tỷ? KHÔNG có Bucket 1?
18c. **Negative catalyst gates** áp đúng — HARD reject (audit qualified/adverse, suspended, lãnh đạo sai phạm, BCTC restate material, regulatory action ban kinh doanh) loại hẳn? SOFT reject (BCTC miss 1 quý, chính sách siết một phần, regulatory observation) cap LOW + flag + chỉ Tier 2?
18d. **Conviction CAP rules** áp đúng — contradict regime → LOW; không catalyst ngày → MID; evidence < 2 trục → MID; consensus crowded → MID; penny stock → LOW + chỉ Tier 2; newly listed → MID?
18e. **Bear regime mode** (nếu Trục 1 macro negative + Trục 2 định vị "phân phối/suy yếu") — Tier 1 = 8 mã, ≥40% defensive sectors, ADV Tier 1 ≥ 8 tỷ, conviction CAP MID toàn pack, bear case mandatory mỗi Tier 1 mã?
18f. **Sector diversification** — không > 30% mã/sector ở mỗi tier; Tier 2 đa dạng ≥5 sectors?
18g. **Risk materialize auto-action** (`_05` mục 4.1): nếu ≥2 risks materialize trong cycle → conviction toàn pack downgrade 1 bậc + flag Executive Summary?

### 2.5. Review + user overlay

19. Review N-1 (nếu có) đã có best call / worst call honesty?
20. **User overlay** (nếu có view inject): mỗi view có trạng thái xử lý rõ (Confirm / Partial / Conflict / Flag / Out of scope)? Audit trail trong metadata đầy đủ?

### 2.6. K hygiene + nguồn + branding

21. K hygiene: ký hiệu DB raw đã dịch, taxonomy nội bộ không lộ ra?
22. Số liệu đã quy đổi đơn vị theo `K_agent_db_00` mục 6 (BCTC: tỷ đồng; `*_pct` ĐÃ là điểm % — KHÔNG nhân 100; ngoại lệ nhân 100: `*_trend`, ratio finstats)?
23. Mỗi claim định lượng có nguồn (collection + field / URL web search)?
24. Tin có dẫn link finext.vn hoặc URL gốc?
25. Branding info đã render đúng (custom / default branded / plain)?
26. Disclaimer có forward-looking statement language?
27. Executive summary 3-6 bullet đứng đầu, đọc 30 giây hiểu toàn báo cáo? Mention user overlay HIGH-conviction hoặc Conflict (nếu có)?

## 3. Self-audit Weekly update mode

### 3.1. HARD GATE + cấu trúc

1. Pre-flight HARD GATE đã pass: agent compute đúng tuần [N] tháng [M/YYYY]? User đã confirm có monthly active đúng tháng (hoặc đã chọn override path với note)?
2. Header báo cáo có ghi rõ "tuần [N] của tháng [M/YYYY]" và link tham chiếu monthly?
3. Nếu user chọn Stage 0 (eval W-1): checkpoint 0 đã user phản hồi? Eval block đủ 4 phần (Status carry-over / Watchlist W-1 tracking / Action item W-1 / Carry-forward)?
4. Đã đọc file monthly active user upload? Extract đủ 6 trục?

### 3.2. Technical-as-noise rule (BẤT BIẾN — theo `_00` mục 4.3 + `_08` mục 4)

5. **Mỗi trục có 1 trong 3 status rõ (Hold / Shift / Materialize)?**
6. **Status Shift / Materialize bắt buộc kèm signal vĩ mô / cơ bản / chính sách?** Technical shift đơn độc (vd VNINDEX MA tụt, industry rank tụt) → status Hold + ghi "noise tạm thời", KHÔNG gọi Shift?
7. Ngoại lệ Rebucket entry (Bucket 3 → Bucket 1 khi w/m bật A): được PTKT-driven, KHÔNG là Shift thesis?

### 3.3. Watchlist refresh + nội dung

8. Watchlist refresh có phân loại 4 trạng thái (Hold / Watch closely / Out / Vào mới)? Mã "Vào mới" có đủ 6 thành phần Phase 1 + ADV + Bucket?
9. Mã Bucket 2 ≥ 4 tuần chưa confirm → flag timeout?
10. Action item tuần tới định tính, không có command/level giá?

### 3.4. User overlay + K hygiene

11. **User overlay** (nếu có view inject tuần): mỗi view có trạng thái xử lý + audit trail?
12. K hygiene + nguồn đầy đủ?
13. Độ dài 3-5 trang — không phình ra mode monthly?

## 4. Edge cases cross-cutting

### 4.1. Monthly mode

- **Thiếu dữ liệu trục 1 (vĩ mô):** chỉ số vĩ mô tháng có thể release chậm 2-3 tuần. Dùng số gần nhất, ghi rõ ngày cập nhật. Web search bổ sung nếu DB lỗi thời.
- **Tháng "không có gì đặc biệt" (không có theme rõ):** ghi explicit "Tháng N không có theme dominant. Chiến lược: chờ tín hiệu mới, sector bias trung tính, watchlist defensive." Không bịa theme.
- **Conflict regime vĩ mô vs định vị thị trường:** vd vĩ mô tích cực nhưng định vị thị trường quá mua → ghi rõ tension, cho cả 2 view, để checkpoint user quyết.
- **Tháng đầu cycle (không có file N-1):** skip Stage 0 eval + skip Review section, ghi 1 dòng "Lần đầu chạy monthly cycle, chưa có dữ liệu review tháng trước".
- **Stage 0 eval phát hiện thesis cũ sai rõ:** không cố biện minh. Trong eval, ghi worst call honest + pitfall identified. Carry-forward learning: tháng N tránh lặp pitfall đó, có thể nâng/giảm conviction methodology tương ứng.
- **File N-1 user upload không parse được (corrupt MD / sai format):** báo user file lỗi, hỏi có muốn skip Stage 0 không. Không tự đoán nội dung file.
- **Toàn bộ 18 ngành whitelist đều "trung tính" hoặc "thận trọng":** ghi rõ "tháng N không có ngành quan tâm — môi trường khó, watchlist defensive 2-3 mã hoặc bỏ trống". Không ép vào ngành.

### 4.2. Weekly update mode

- **HARD GATE — không có monthly active:** REFUSE chạy weekly, đề xuất 2 path (monthly trước / override). Tuyệt đối không silently chạy weekly độc lập — pack không build weekly không có parent thesis.
- **HARD GATE — monthly active của tháng khác:** flag rõ, present 2 path. Nếu user chọn override với monthly cũ, header báo cáo cuối có note "Thesis carry-over từ tháng M-1, đã decay [X] tuần, conviction tổng thể giảm 1 bậc so monthly gốc".
- **Tuần 1 của tháng (vừa chạy monthly):** không có W-1 → tự động skip Stage 0 eval. Weekly chạy nhẹ vì thesis tươi.
- **Stage 0 eval phát hiện trục có shift lớn ở W-1 nhưng tuần [N] không tiếp diễn:** ghi rõ "shift W-1 đã reverse, không carry-forward; quay lại baseline monthly".
- **Tuần "không có shift":** vẫn render report ngắn 2-3 trang, ghi rõ tất cả trục Hold, watchlist Hold, action item "tiếp tục theo dõi signals đã đặt". Không bịa shift.
- **Materialize risk lớn (≥2 rủi ro materialize trong cùng tuần):** flag rõ ở đầu báo cáo + Tóm tắt tuần, đề xuất user xem xét chạy lại monthly cycle giữa kỳ (mid-month refresh).
- **Mâu thuẫn weekly update vs monthly (≥3 trục materialize/shift mạnh):** không tự "viết lại" monthly. Báo user: "Nhiều shift lớn tuần này, đề xuất chạy lại monthly cycle thay vì tiếp tục weekly update".
- **File W-1 user upload không parse được:** báo user, hỏi có skip Stage 0 không.

### 4.3. Cross-cutting (cả 2 mode)

- **Whitelist ngành thay đổi giữa cycle** (vd user thêm/bỏ ngành): áp dụng từ cycle kế tiếp, không retroactive trong báo cáo hiện tại. Ghi rõ trong metadata "whitelist update [DD/MM]".
- **Mã trong watchlist bị huỷ niêm yết / split / merge:** flag riêng + loại khỏi watchlist; nếu là target theme đại diện → xem xét thay mã khác cùng theme.
- **Vi phạm cap technical phát hiện ở self-audit:** re-write trục vi phạm, đào sâu fundamental, KHÔNG render với note "technical hơi cao chấp nhận được". Cap là cap.
- **User inject view ở tier ngoài 6 trục:** xử lý theo matrix "Out of scope" mục 1.2, đề xuất pack/workflow khác phù hợp.

### 4.4. Standalone

- Pack standalone — kích hoạt pack này thì dùng pack này thôi, không cross-reference pack khác. Watchlist của pack này dừng ở theme play + bucket entry observation, không cover entry/stop/target/size.

## 5. Output contract — Monthly mode

Pack sinh structured content cho `O_vbse_strategy_00` render. Ràng buộc bắt buộc:

- **6 trục đầy đủ** (kể cả trục Hold không có gì đặc biệt vẫn có 3-5 dòng kết luận)
- **Weight balance:** mọi trục tuân cap technical theo `_00` mục 4.2. Trục 2 ≤ 20%, Trục 3 ≤ 5%, Trục 4 ≤ 5%, Trục 5 = 0%, Phase 1 Trục 6 = 0%, Phase 2 Trục 6 ~ 80-100% (đây là vùng hợp pháp cho PTKT). Báo cáo tổng (trừ Phase 2 Bucket) ≤ 15% technical. Vi phạm = re-write trước render
- **Whitelist 18 ngành:** mọi query / aggregate / ranking / bảng tilts / watchlist tuân Nguyên tắc 3 master
- **Stage 0 evaluation** (nếu user chọn chạy): có checkpoint 0 user phản hồi, eval block đủ 6 phần, carry-forward feed vào Stage 1
- **Checkpoint 1 regime + themes** có user phản hồi trước khi vào Stage 2
- **Trục 3 themes** — mỗi theme đủ 5 thành phần (cơ chế / conviction HIGH-MID-LOW / horizon 1m-1to3m-3to6m / catalyst trigger / disconfirming signals)
- **Trục 4 sector** — bảng tilts tổng hợp 18 ngành với conviction + driver + disconfirming signal mỗi ngành
- **Trục 5** — 3 kịch bản if-then trigger (macro/fundamental/policy ONLY), không % xác suất; risk map 3-7 rủi ro với theme invalidation cross-link
- **Trục 6 watchlist 20 mã (10 Tier 1 + 10 Tier 2, điều chỉnh theo regime: Bear 8/12, quá mua 6/14)** — mỗi mã 6 thành phần Phase 1 (ticker-ngành-theme / conviction / horizon / luận điểm / signal theo dõi / disconfirming) + Tier marker + ADV tháng + Bucket entry (1/2/3, Tier 2 không Bucket 1); Tier 1 mandatory Variant Perception; observation only, không entry/stop/target/size
- **Caution mechanisms áp tự động:** negative catalyst gate (HARD/SOFT), conviction CAP rules, risk materialize auto-action (`_05` mục 4.1), Bear regime mode (`_06` mục 5) khi Trục 1 + Trục 2 đồng thuận bearish
- **Review N-1** (nếu có) có best call / worst call
- **User overlay** (nếu có): mỗi view có badge trạng thái xử lý + integrate đúng matrix; metadata có User overlay log
- **Executive summary** viết cuối, 3-6 bullet, mention user overlay HIGH-conviction hoặc Conflict nếu có
- **Số liệu đã quy đổi**, ký hiệu đã dịch (K hygiene)
- **Mỗi claim định lượng có nguồn**
- **Disclaimer** có forward-looking statement language

## 6. Output contract — Weekly update mode

- **HARD GATE pre-flight:** agent compute tuần [N] tháng [M/YYYY] + check user có monthly active đúng tháng. Không có monthly → REFUSE, không silently chạy weekly độc lập
- **Header báo cáo** ghi rõ "tuần [N] của tháng [M/YYYY]" + link tham chiếu file monthly
- **File monthly active user upload** là input bắt buộc (trừ override path đã được agreed với note rõ)
- **Stage 0 evaluation W-1** (nếu user chọn chạy): checkpoint 0 user phản hồi, eval block đủ 4 phần
- **6 trục mỗi trục có status Hold/Shift/Materialize** với Technical-as-noise rule áp dụng (Shift bắt buộc kèm signal vĩ mô/cơ bản/chính sách)
- **Watchlist refresh 4 trạng thái** (Hold / Watch closely / Out / Vào mới); mã Vào mới có đủ 6 thành phần Phase 1 + ADV + Bucket. Rebucket entry là ngoại lệ duy nhất PTKT-driven trong weekly
- **1-2 action item định tính**
- **User overlay** (nếu có) tương tự monthly
- **Độ dài 3-5 trang**
- **K hygiene + nguồn đầy đủ**

Pack KHÔNG tự quyết heading style / xưng hô / tone / length cuối — `O_vbse_strategy_00` quyết.
