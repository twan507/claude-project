# P_vbse_strategy_00 — Master Workflow

File này là điểm vào tổng thể của pack `P_vbse_strategy` — quy trình build báo cáo chiến lược đầu tư VN theo 2 chu kỳ lồng nhau (tháng parent + tuần child). Đọc xong 5-7 trang này sẽ hiểu kiến trúc, biết giai đoạn nào cần mở file con nào. Spec chi tiết nằm ở 9 file con của pack — không trùng lặp với master.

**Pack dependency:** `P_vbse_strategy` phụ thuộc `K_agent_db` (schema, query patterns, methodology diễn giải chỉ báo và tin tức). Khi chạy workflow này, agent phải đọc trước file `_00` của cả hai pack. Render spec ở `O_vbse_strategy_00`.

## 1. Mục đích & triết lý

**Mục đích:** sinh báo cáo **chiến lược đầu tư** (institutional buy-side style) cho thị trường cổ phiếu VN, horizon 1-3 tháng forward-looking, audience analyst nội bộ + có thể swap branding cho audience khách hàng. Mục tiêu cao nhất:

- Định vị thị trường VN trong chu kỳ vĩ mô + chu kỳ thanh khoản + chu kỳ định giá
- Chốt 2-5 themes & narratives chi phối tháng/quý tới
- Đề xuất sector allocation (ngành quan tâm / trung tính / cần thận trọng)
- Build risk framework đa kịch bản với trigger signal cụ thể
- Đưa watchlist **20 mã** đại diện theme (10 Priority Tier 1 + 10 Standby Tier 2) — kèm **bucket entry timing** (observation, không level giá)

**Scope cốt lõi:** pack này là báo cáo **định vị chiến lược** — VN đang ở đâu trong chu kỳ, theme nào chi phối, sector nào ưu tiên, kịch bản nào dự phòng, mã nào đại diện theme. Pack standalone — kích hoạt pack này thì dùng pack này, không cross-reference pack khác.

**Scope dừng ở watchlist quan sát chiến lược:** luận điểm theme, signals theo dõi, bucket entry timing. KHÔNG cover entry/stop/target/size — đó là phạm vi khác, không nằm trong pack này.

### 1.1. Triết lý fundamental-driven supremacy

**Pack này được thiết kế để fundamental-driven nhất có thể.** Mọi quyết định chiến lược — regime call, theme chốt, sector bias, risk scenario, watchlist universe — đều phải dẫn dắt bằng **vĩ mô / cơ bản / chính sách / catalyst**. Phân tích kỹ thuật (PTKT) **không có vai trò quyết định** trong bất kỳ trục nào.

**Vai trò duy nhất của PTKT trong pack này:** phân **bucket entry timing** cho mã đã được chọn vào watchlist bằng cơ bản. Cụ thể:

- **Phase 1 Screen** (chọn universe watchlist): chỉ cơ bản + catalyst + thanh khoản. **Cấm dùng PTKT làm filter**.
- **Phase 2 Bucket entry** (sau khi đã có watchlist): PTKT đa khung (technical_zone w/m/q/y) làm tiêu chí phân Bucket 1 / 2 / 3 (xem `P_vbse_strategy_06`).

**Hệ quả:** ngành/mã có cơ bản tốt + định giá rẻ + catalyst rõ nhưng technical xấu → **vẫn vào watchlist** với Bucket 2 hoặc 3 (chờ pullback hoặc đợi technical bật). Không bị loại.

Triết lý này đảo ngược pattern phổ biến của retail VN (technical-first, fundamental sau) — vốn là nguồn alpha leak lớn nhất khi horizon ≥ 1 tháng.

### 1.2. Triết lý thiết kế cấu trúc

Khung tư duy tổng quát, structure flex theo phát hiện thực tế của tháng/tuần. **Không ép 6 phần rigid.** Agent được khuyến khích đào sâu trục có signal mạnh, lướt nhanh trục không có gì đặc biệt. Số lượng theme/sector/mã linh hoạt theo bối cảnh, không ép con số cứng.

### 1.3. Quan hệ tháng ↔ tuần

- **Báo cáo tháng = parent**, chạy 1 lần đầu tháng (tuần đầu tiên). Hình thành thesis tổng + 6 trục đầy đủ. Spec: `P_vbse_strategy_07`.
- **Báo cáo tuần = child**, chạy mỗi tuần trong tháng. **Đọc báo cáo tháng đang active** → track signals đã đặt → flag shift / hold → cập nhật watchlist & risk map. Không build thesis từ đầu. Spec: `P_vbse_strategy_08`.
- Sau 4-5 tuần update, đến đầu tháng mới chạy lại monthly cycle (refresh full thesis).

### 1.4. Input kỳ vọng

| Mode | Input bắt buộc | Input optional |
|---|---|---|
| Monthly | Trigger user "báo cáo chiến lược tháng" / "monthly strategy" / "outlook tháng N" + ngày kết thúc tháng N-1 hoặc đầu tháng N | Branding info, focus theme/sector user gợi ý, file báo cáo tháng N-1 (để review hit rate thesis cũ) |
| Weekly | Trigger user "update tuần [DD/MM]" / "weekly strategy update" + **file báo cáo tháng đang active** (user upload) | File báo cáo tuần W-1, sự kiện tuần qua user nhấn mạnh |

**Output kỳ vọng:** file MD theo structure flex của `O_vbse_strategy_00`. Monthly target 8-12 trang, weekly update target 3-5 trang (gọn hơn vì là tracking).

### 1.5. Negative scope

- Không phải báo cáo tổng quan thị trường tuần.
- Không phải pipeline screen + deep-dive mã cụ thể cho portfolio. Watchlist ở pack này dừng ở mức luận điểm theme + bucket entry (20 mã observation, chia 2 tier), không có entry/stop/target/size.
- Không khuyến nghị giao dịch ngắn hạn (intraday, T+) — horizon 1-3 tháng forward.
- Không gán % xác suất kịch bản (tuân `K_agent_db_00` mục 4.3).
- Không dùng từ command (mua/bán/giảm tỷ trọng/stop loss) — diễn đạt qua observation/luận điểm.
- **Không sử dụng chỉ báo trend nội bộ** (`*.trend`, `series[].trend` trong `*_recent`) nếu báo cáo render branded cho audience KH. Khi render plain (nội bộ) thì được dùng, nhưng phải dịch ra ngôn ngữ tự nhiên.

## 2. Naming & lưu trữ

**Naming file output:**
- Monthly: `vbse_strategy_monthly_<YYYYMM>.md` — YYYYMM là tháng báo cáo (vd `vbse_strategy_monthly_202605.md` cho tháng 5/2026)
- Weekly update: `vbse_strategy_weekly_<YYYYMMDD>.md` — YYYYMMDD là ngày kết thúc tuần update

**Lưu trữ:** agent KHÔNG lưu file qua session. User tự archive. Mode weekly bắt buộc user upload file monthly đang active — pack không tự nhớ thesis tháng.

## 3. Từ điển thuật ngữ cốt lõi

| Thuật ngữ | Giải thích |
|---|---|
| **Horizon 1-3 tháng forward** | Khoảng thời gian báo cáo định hướng. Pack thiết kế cho trung hạn — không trading T+, không holding ≥ 1 năm. |
| **Regime vĩ mô** | Trạng thái tổng quát chu kỳ vĩ mô + tài chính: early cycle nới lỏng / mid cycle ổn định / late cycle thắt chặt / shock. Không cần phân loại chính thức, dùng ngôn ngữ định tính. |
| **Theme** | Câu chuyện lớn (chính sách, sự kiện vĩ mô, mùa BCTC, M&A, commodity cycle) chi phối dòng tiền 1-3 tháng tới. Mỗi theme 5 thành phần: cơ chế / conviction / horizon / catalyst / disconfirming signals. |
| **Conviction** | HIGH / MID / LOW — mức tự tin về theme/sector/mã. HIGH = cơ chế rõ + catalyst materialize + cross-check ≥2 trục đồng thuận. |
| **Sector tilt** | Phân tầng định tính ngành: quan tâm / trung tính / cần thận trọng. Không dùng "overweight/underweight" để tương thích cả audience nội bộ và KH. |
| **Whitelist 18 ngành** | Pack chỉ phân tích 18 ngành trong scope (xem `K_agent_db_00` mục 4.5 + `K_agent_db_01` Section B). Các ngành ngoài whitelist không xuất hiện trong báo cáo. |
| **Watchlist 20 mã (2 tier)** | Output Trục 6 — mã đại diện theme & sector bias. Chia Tier 1 (10 Priority) + Tier 2 (10 Standby). Observation only, không entry/stop/target. |
| **Tier 1 — Priority picks** | 10 mã ưu tiên cao. Cơ bản strong + catalyst rõ với ngày + ADV ≥ 5 tỷ. Conviction HIGH/MID. Bucket 1/2/3 áp dụng đầy đủ. |
| **Tier 2 — Standby picks** | 10 mã chờ thời cơ. Cơ bản clean (no red flag) + KHÔNG catalyst tiêu cực + ADV ≥ 3 tỷ + technical setup bottom-fishing. Conviction MID/LOW (cap MID). KHÔNG có Bucket 1. |
| **Phase 1 Screen** | Bước chọn universe watchlist — Tier 1: cơ bản strong + catalyst + thanh khoản. Tier 2: cơ bản clean + no negative catalyst + technical bottom-fishing setup. |
| **Phase 2 Bucket entry** | Bước phân entry timing cho mã đã chọn — **PTKT đa khung làm tiêu chí** (Bucket 1 vào ngay / Bucket 2 chờ pullback / Bucket 3 watchlist). Định nghĩa pack-internal ở `P_vbse_strategy_06` mục 4.1. |
| **Bucket 1** | Technical_zone w/m ∈ (A, AA, AAA) + dòng tiền tuần tích cực → vào ngay khi catalyst materialize. **Chỉ Tier 1.** |
| **Bucket 2** | Technical_zone q/y tốt nhưng w/m đang B/C → chờ pullback xác nhận. |
| **Bucket 3** | Chỉ q/y tích cực, w/m yếu → watchlist, chưa vào, đợi technical bật. |
| **Bear regime mode** | Mode đặc biệt Trục 6 khi Trục 1 macro negative + Trục 2 định vị "phân phối/suy yếu" (chỉ dùng vĩ mô, không PTKT). Tier 1 giảm 8 mã (≥40% defensive), Tier 2 mở 12 mã. Conviction cap MID toàn pack. Bear case mandatory mỗi Tier 1 mã. |
| **Negative catalyst gate** | HARD reject (loại hẳn): audit qualified/adverse/disclaimer, suspended, lãnh đạo sai phạm, BCTC restate material > 30%, regulatory action ban kinh doanh. SOFT reject (cap LOW + flag + chỉ Tier 2): BCTC miss 1 quý < 20%, chính sách siết một phần, regulatory observation, commodity tiêu cực ngắn hạn, kế toán bất thường 1 lần. |
| **Conviction CAP rules** | Bộ rule cấu trúc chống HIGH conviction inflation: contradict regime → LOW; không catalyst ngày → MID; evidence < 2 trục → MID; consensus crowded → MID; penny stock → LOW + chỉ Tier 2; newly listed → MID; Bear regime → MID toàn pack. |
| **Variant Perception** | Mandatory chỉ Tier 1: mỗi mã 1 dòng "consensus đang nghĩ X" + "thesis này khác ở Y". Consensus trùng → flag "consensus crowded" + cap MID. Tier 2 không bắt buộc. |
| **6 trục cốt lõi** | Spine báo cáo: Vĩ mô / Định vị / Themes / Sector / Risk / Watchlist (chi tiết `P_vbse_strategy_01` đến `06`). |
| **Weight balance** | Rule trọng số signal theo tầng PRIMARY (vĩ mô/cơ bản/chính sách/catalyst) ≥ SECONDARY (định vị/flow) > TERTIARY (technical). Mục 4 dưới đây. |
| **Checkpoint (CP)** | Điểm dừng bắt buộc — agent xuất block ngắn, user confirm/override trước khi qua stage kế. Monthly có CP0 + CP1; Weekly có CP0. |
| **HARD GATE** | Pre-flight weekly bắt buộc — không có monthly active đúng tháng → refuse weekly mode (xem `P_vbse_strategy_08`). |
| **Stage 0 eval** | Stage optional đánh giá thesis chu kỳ trước (monthly N-1 hoặc weekly W-1) trước khi build mới. |
| **Disconfirming signal** | "What would change our mind" — 2-3 chỉ báo cụ thể sẽ invalidate theme/thesis. Chuẩn institutional research. |
| **User overlay** | Cơ chế ghép view user vào báo cáo qua 3 channel (pre-flight / mid-flow / checkpoint) + synthesis matrix 5 trạng thái (xem `P_vbse_strategy_09`). |

## 4. Weight balance — Rule trọng số (BẤT BIẾN)

Đây là **rule quan trọng nhất của pack** — quyết định signal nào driver chính, signal nào chỉ confirmation.

### 4.1. Monthly horizon (1-3 tháng / 3-6 tháng) — driver chính phải dài hơi

| Tầng | Trọng số chỉ định | Loại signal | Lý do |
|---|---|---|---|
| **PRIMARY (~70-75%)** | Trục 1 + 3 + phần lớn Trục 4 + Phase 1 Screen Trục 6 | **Vĩ mô** (lãi suất / tỷ giá / FII flow / chu kỳ vĩ mô) + **Cơ bản** (BCTC, tăng trưởng EPS, ROE, biên, định giá vs lịch sử) + **Chính sách** (Nghị quyết / luật / quyết định bộ ngành / Fed-ECB-PBOC) + **Catalyst dài hơi** (M&A, niêm yết mới, upgrade FTSE/MSCI, mùa BCTC, dividend cycle, capacity online) | Factor có time-to-play-out khớp horizon 1-3 tháng. Re-rating equity, sector rotation lớn, theme dominant đều dẫn dắt bởi nhóm này. |
| **SECONDARY (~15-20%)** | Trục 2 (chủ yếu) + phần dòng tiền Trục 4 | **Định vị thị trường** (P/E phân vị lịch sử, dòng tiền tuần aggregate 4 tuần, NN tháng/quý, breadth tổng thể, sentiment proxy) | Chu kỳ tâm lý + định vị flow định hướng dòng tiền chung. Trung gian giữa fundamental dài hơi và technical ngắn hạn. |
| **TERTIARY (~10-15%, chỉ ở Phase 2 Bucket entry Trục 6)** | Phase 2 Bucket entry Trục 6 | **Technical** (technical_zone đa khung, MA, Fibonacci, POC) | **Chỉ làm bucket entry timing cho mã đã chọn bằng cơ bản. KHÔNG được dùng làm primary driver ở Trục 1, 3, 4, 5 hoặc Phase 1 Screen Trục 6.** |

### 4.2. Cap technical cứng theo trục

| Phần báo cáo | Cap technical |
|---|---|
| Trục 1 — Vĩ mô | **0%** (không liên quan) |
| Trục 2 — Định vị thị trường | **≤ 20%** (chỉ minh hoạ phụ, không driver) |
| Trục 3 — Themes | **≤ 5%** (chỉ trong disconfirming signal nếu thực sự cần) |
| Trục 4 — Sector allocation | **≤ 5%** (gần như 0 — sector bias không bị technical quyết) |
| Trục 5 — Risk scenarios | **0% (CẤM TUYỆT ĐỐI)** — trigger phải macro/fundamental/policy |
| Trục 6 Phase 1 Screen | **0% (CẤM TUYỆT ĐỐI)** — universe filter chỉ cơ bản + catalyst + thanh khoản |
| Trục 6 Phase 2 Bucket entry | **~80-100%** (đây là nơi PTKT có chỗ đứng hợp pháp) |
| Báo cáo tổng (trừ Phase 2 Bucket) | **≤ 15%** |

**Self-audit bắt buộc:** trước khi xuất báo cáo, agent đếm % nội dung technical trong từng trục. Vi phạm cap → re-weight, đào thêm fundamental, đồng thời cắt bớt technical. Chi tiết self-audit ở `P_vbse_strategy_09`.

### 4.3. Weekly horizon — Technical-as-noise rule

Weekly tracking horizon ngắn → cám dỗ lean technical lớn hơn monthly. Pack siết:

- **Status trục Hold/Shift/Materialize bắt buộc kèm signal vĩ mô/cơ bản/chính sách**. Technical shift đơn độc (vd "VNINDEX tụt khỏi MA20", "industry rank tụt") **không đủ** để gọi Shift — phải gọi "noise tạm thời, status Hold".
- **Ngoại lệ duy nhất:** rebucket entry trong watchlist (Bucket 3 → Bucket 1 khi w/m bật A có thể gọi shift bucket). Đây là tracking entry timing, không phải shift thesis.

Detail rule ở `P_vbse_strategy_08`.

## 5. Kiến trúc tổng thể

### 5.1. Sơ đồ pack

```
MASTER (P_vbse_strategy_00) — file này
│
├── 6 TRỤC CỐT LÕI (spine báo cáo)
│   ├── Trục 1 — Vĩ mô & tài chính           (P_vbse_strategy_01)
│   ├── Trục 2 — Định vị thị trường VN        (P_vbse_strategy_02)
│   ├── Trục 3 — Themes & narratives          (P_vbse_strategy_03)
│   ├── Trục 4 — Sector allocation            (P_vbse_strategy_04)
│   ├── Trục 5 — Risk scenarios               (P_vbse_strategy_05)
│   └── Trục 6 — Watchlist + Bucket entry     (P_vbse_strategy_06)
│
├── 2 WORKFLOW MODES
│   ├── Workflow Monthly                      (P_vbse_strategy_07)
│   └── Workflow Weekly Update + HARD GATE    (P_vbse_strategy_08)
│
└── META (overlay + audit + edge + contract)  (P_vbse_strategy_09)

RENDER SPEC (O_vbse_strategy_00) — output MD final
```

### 5.2. Bảng file index

| File | Nội dung chính | Khi nào đọc |
|---|---|---|
| `P_vbse_strategy_00` | Master — philosophy, naming, từ điển, weight balance, file index | Đầu mọi session |
| `P_vbse_strategy_01` | Trục 1 Vĩ mô — lăng kính macro/policy/commodity, data source, output spec | Build Trục 1 |
| `P_vbse_strategy_02` | Trục 2 Định vị thị trường — định giá phân vị + dòng tiền aggregate + FII + breadth (fundamental-first), technical chỉ minh hoạ ≤20% | Build Trục 2 |
| `P_vbse_strategy_03` | Trục 3 Themes — 5 thành phần mỗi theme, web search heavy | Build Trục 3 |
| `P_vbse_strategy_04` | Trục 4 Sector — PRIMARY cross 3 trục + BCTC + định giá + catalyst, SECONDARY flow, **bỏ TERTIARY technical**. Whitelist 18 ngành integration. | Build Trục 4 |
| `P_vbse_strategy_05` | Trục 5 Risk — 3 kịch bản if-then (trigger macro/fundamental/policy ONLY) + risk map 3-7 rủi ro | Build Trục 5 |
| `P_vbse_strategy_06` | Trục 6 Watchlist 2-tier 20 mã — Tier 1 Priority (10) + Tier 2 Standby (10). Phase 1 Screen (cơ bản + negative catalyst gate) + Phase 2 Bucket entry (PTKT-driven). Bear regime mode khi market xấu. Bucket 1/2/3 định nghĩa pack-internal. | Build Trục 6 |
| `P_vbse_strategy_07` | Workflow Monthly — Pre-flight, Stage 0 eval N-1, CP0, Stage 1-3, CP1, render | Chạy monthly cycle |
| `P_vbse_strategy_08` | Workflow Weekly Update — HARD GATE pre-flight, Stage 0 eval W-1, Stage 1 tracking 6 trục với technical-as-noise rule, render | Chạy weekly cycle |
| `P_vbse_strategy_09` | User overlay (3 channel + matrix 5 trạng thái) + Self-audit (16 item monthly / 8 item weekly với cap technical siết) + Edge cases + Output contract | Trước render + khi user inject view |
| `O_vbse_strategy_00` | Render spec MD final cho cả 2 mode | Render output |

### 5.3. Bảng tóm tắt 6 trục (cho cả monthly + weekly)

| Trục | Câu hỏi cốt lõi | Driver chính | Nguồn data chính |
|---|---|---|---|
| 1 — Vĩ mô | Môi trường lãi suất / tỷ giá / dòng vốn đang hỗ trợ hay siết equity VN 1-3 tháng? | Macro + policy quốc tế + trong nước | `other_data` + web search bắt buộc |
| 2 — Định vị | VNINDEX ở đâu trong chu kỳ định giá / dòng tiền / breadth? | **Định giá phân vị + dòng tiền aggregate 18 ngành + FII + breadth** | `market_snapshot`, `market_recent`, `market_nntd`, `industry_snapshot` aggregate, `data_briefing` |
| 3 — Themes | 2-5 câu chuyện lớn nào chi phối 1-3 tháng tới? | Chính sách + sự kiện vĩ mô + mùa BCTC + M&A + commodity cycle | `news_history_feed` + web search heavy |
| 4 — Sector | Ngành nào quan tâm / trung tính / thận trọng (whitelist 18)? | Cross 3 trục + BCTC ngành + định giá phân vị + catalyst | `industry_finstats`, `industry_snapshot`, `industry_recent`, `news_history_feed` |
| 5 — Risk | Kịch bản nào đảo ngược thesis? Trigger nào báo hiệu? | Macro / fundamental / policy trigger **only** | `other_data` + `industry_finstats` + `news_history_feed` + web |
| 6 — Watchlist | 20 mã đại diện theme (10 Tier 1 + 10 Tier 2). Phase 1 Screen 2-tier + Phase 2 Bucket (PTKT timing) + Bear regime mode | Tier 1: cơ bản strong + catalyst + ADV ≥5 tỷ. Tier 2: cơ bản clean + technical bottom-fishing + ADV ≥3 tỷ. Phase 2: technical_zone đa khung | `stock_snapshot`, `stock_finstats`, `stock_recent`, `stock_nntd`, `news_history_feed` |

### 5.4. Bảng tóm tắt 2 workflow

| Workflow | Stage | Output | Tần suất | Checkpoint |
|---|---|---|---|---|
| Monthly | Pre-flight → [Stage 0 eval N-1] → CP0 → Stage 1 Trục 1-3 → CP1 → Stage 2 Trục 4-5 → Stage 3 Trục 6 + Exec sum → Render | File `vbse_strategy_monthly_<YYYYMM>.md` 8-12 trang | 1 lần đầu tháng | CP0 + CP1 |
| Weekly | Pre-flight HARD GATE → [Stage 0 eval W-1] → CP0 → Stage 1 tracking 6 trục → Render | File `vbse_strategy_weekly_<YYYYMMDD>.md` 3-5 trang | Mỗi tuần trong tháng | CP0 (nếu chạy Stage 0) |

Chi tiết workflow ở `P_vbse_strategy_07` (Monthly) + `P_vbse_strategy_08` (Weekly).

## 6. Sáu nguyên tắc Agent bất biến

Các nguyên tắc này áp dụng ở mọi trục, mọi stage, mọi mode. Agent không được skip hay tự ý nới lỏng.

**Nguyên tắc 1 — Fundamental supremacy.**

Mọi quyết định chiến lược (regime / theme / sector bias / risk / watchlist Phase 1) phải được driver bởi vĩ mô / cơ bản / chính sách / catalyst. PTKT **không có vai trò quyết định** trong các phần này. Vi phạm cap technical → re-weight trước khi render.

**Nguyên tắc 2 — Technical chỉ làm bucket entry timing.**

PTKT có chỗ đứng hợp pháp **duy nhất** ở Phase 2 Bucket entry của Trục 6 (phân Bucket 1/2/3 cho mã đã chọn bằng cơ bản). Không bao giờ làm filter loại universe, không bao giờ làm trigger chính cho thesis shift weekly.

**Nguyên tắc 3 — Whitelist 18 ngành bắt buộc tuân thủ.**

Mọi query / aggregate / ranking cấp ngành filter theo whitelist 18 ngành (xem `K_agent_db_00` mục 4.5 + `K_agent_db_01` Section B). Aggregate proxy thị trường (dòng tiền, breadth) tính trên 18 ngành whitelist, không 24. Mã thuộc ngành ngoài whitelist không vào watchlist theme / sector tilts.

**Nguyên tắc 4 — Không gán % xác suất kịch bản, không command từ.**

Kịch bản dùng if-then trigger định tính, không % xác suất. Watchlist diễn đạt observation, không "mua/bán/giảm tỷ trọng/stop loss". Tương thích cả audience nội bộ và KH.

**Nguyên tắc 5 — Disconfirming signal bắt buộc mỗi theme/sector/mã.**

Mỗi theme (Trục 3), mỗi sector tilt (Trục 4), mỗi mã watchlist (Trục 6) bắt buộc kèm 2-3 disconfirming signal cụ thể (reference data field hoặc số cụ thể). "What would change our mind" — không có disconfirming = không phải thesis chuẩn institutional.

**Nguyên tắc 6 — Mỗi stage kết thúc bằng checkpoint, agent không tự chuyển stage.**

Monthly: CP0 sau Stage 0 eval (nếu chạy), CP1 sau Stage 1 build thesis. Weekly: CP0 sau Stage 0 eval (nếu chạy). User confirm/override/refine trước khi qua stage kế. Override ghi inline note trong trục liên quan của báo cáo cuối.

**Nguyên tắc 7 — Caution mechanisms khi market xấu (bắt buộc).**

Pack có 4 cơ chế caution structural để chống bias buy/HIGH conviction khi thị trường xấu:

1. **Negative catalyst gate (Trục 6 mục 2.2.e + 2.3.d-e):** HARD reject (audit qualified/adverse, suspended, lãnh đạo sai phạm, BCTC restate material, regulatory action) → loại hẳn. SOFT reject (BCTC miss 1 quý, chính sách siết một phần, regulatory observation) → cap LOW + flag + chỉ Tier 2.
2. **Conviction CAP rules (Trục 3 mục 3 + Trục 4 mục 4.1 + Trục 6 mục 3):** contradict regime → LOW; không catalyst ngày → MID; evidence < 2 trục → MID; consensus crowded → MID + flag; penny stock → LOW; newly listed → MID.
3. **Risk materialize auto-action (Trục 5 mục 4.1):** ≥2 risks materialize trong cycle → auto downgrade conviction toàn pack 1 bậc. ≥3 risks → recommend chạy lại monthly cycle mid-month.
4. **Bear regime mode (Trục 6 mục 5):** trigger khi Trục 1 macro negative + Trục 2 định vị "phân phối/suy yếu" (chỉ dùng vĩ mô, không PTKT). Tier 1 giảm 8 mã (≥40% defensive sectors), Tier 2 mở 12 mã, conviction CAP MID toàn pack, ADV Tier 1 ≥8 tỷ, bear case mandatory mỗi Tier 1 mã.

Các cơ chế này áp tự động, không cần user enable. Self-audit `P_vbse_strategy_09` check đầy đủ.

## 7. Cơ chế checkpoint review

Mỗi checkpoint agent xuất block 0.5-1 trang trong message (không render full MD), chờ user phản hồi. Khung tùy stage:

### CP0 (Monthly Stage 0 eval N-1) — chi tiết `P_vbse_strategy_07`

Eval block 6 phần: Regime / Themes / Sectors / Watchlist / Risk / Calibration learning → user accept / refine / skip carry-forward.

### CP1 (Monthly sau Stage 1 Trục 1-3) — chi tiết `P_vbse_strategy_07`

Block: regime + định vị + 2-5 themes proposed → user confirm / override regime / override themes / đào thêm.

### CP0 (Weekly Stage 0 eval W-1) — chi tiết `P_vbse_strategy_08`

Eval block 4 phần: Status carry-over / Watchlist W-1 tracking / Action item W-1 / Carry-forward → user accept / refine / skip.

**Quyền override + audit trail:** user có quyền override mọi checkpoint. Override ghi inline note trong trục liên quan của báo cáo cuối + mention trong user overlay log (xem `P_vbse_strategy_09`). Pack KHÔNG dùng file `audit_overrides.md` riêng — log nằm trong báo cáo cuối.

## 8. Hướng dẫn dùng trong Claude project

### 8.1. Setup project knowledge

**Layer meta (luôn active):** system prompt + `KERNEL_SKELETON.md`.

**Pack dependency:** `K_agent_db` (6 file `_00` đến `_05`).

**Pack này:** `P_vbse_strategy_00` (master) + 9 file tier (`_01` đến `_09`).

**Render spec:** `O_vbse_strategy_00`.

**State artifacts (runtime, user tự archive):**
- Báo cáo monthly đã save (input cho weekly + Stage 0 eval N-1)
- Báo cáo weekly đã save (input cho Stage 0 eval W-1)

Pack không dùng audit log file riêng.

### 8.2. Cách start session

**Monthly session:**
> "Chạy báo cáo chiến lược tháng [N/YYYY]"

Agent route: đọc `P_vbse_strategy_00` → `_07` (Monthly workflow) → mở các file trục con khi vào stage tương ứng → gọi `K_agent_db` cho query → render qua `O_vbse_strategy_00`.

**Weekly session:**
> "Update tuần [DD/MM]" + upload file monthly active

Agent route: đọc `P_vbse_strategy_00` → `_08` (Weekly workflow) → check HARD GATE → tracking → render.

### 8.3. Pattern 1 session/cycle

Mỗi session tương ứng 1 cycle (1 monthly hoặc 1 weekly update). Trong cycle có nhiều checkpoint — user review từng checkpoint trong cùng session, không cần end session giữa checkpoint.

### 8.4. Cross-reference priority

Khi agent không chắc cách thực thi:

1. File tier tương ứng (`P_vbse_strategy_01` đến `_09`) — chi tiết thực thi
2. `P_vbse_strategy_00` (file này) — philosophy, nguyên tắc bất biến, weight balance, từ điển
3. Pack `K_agent_db` (`_00` đến `_05`) — schema, query, methodology diễn giải chỉ báo và tin tức
4. `O_vbse_strategy_00` — render spec
5. System prompt + `KERNEL_SKELETON.md` — chỉ khi vấn đề meta

Không tự suy luận vượt spec khi spec đã có. Spec chưa cover case cụ thể → hỏi user xác nhận trước khi tự quyết.
