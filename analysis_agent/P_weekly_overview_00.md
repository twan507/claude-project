# P_weekly_overview_00 — Master Workflow

File này là điểm vào tổng thể pack `P_weekly_overview` — broadcast tổng quan thị trường tuần độc lập, chuẩn institutional buy-side. Đọc xong 5-7 trang này sẽ hiểu kiến trúc, biết phần nào ở file con nào. Spec chi tiết nằm ở 4 file con của pack — không trùng lặp với master.

**Pack dependency:** `P_weekly_overview` phụ thuộc `K_agent_db` (schema, query patterns, methodology diễn giải chỉ báo và tin tức). Khi chạy workflow này, agent phải đọc trước file `_00` của cả hai pack. Render spec ở `O_weekly_overview_00`.

## 1. Mục đích & triết lý

**Mục đích:** sinh báo cáo **tổng quan thị trường tuần** dạng deliverable MD, **chạy 1 lần/tuần độc lập** (không cần thesis cycle nào). **Dùng được cho cả họp nội bộ và gửi khách hàng** (qua branding info user cung cấp). Tần suất: tối Chủ Nhật / sáng thứ Hai trước phiên giao dịch đầu tuần. Tuần báo cáo = thứ Hai → Chủ Nhật của tuần đã kết thúc.

**Mục tiêu cao nhất:**
- (a) Thống kê đầy đủ dữ liệu thị trường tuần đã qua (quốc tế + VN + ngành + tin tức)
- (b) Đưa ra **regime call + sector bias + watchlist** cho tuần tới với conviction + horizon + disconfirming signal
- (c) Format có thể đứng riêng, không cần thesis monthly hay file cũ làm parent

### 1.1. Triết lý fundamental-driven supremacy

**Pack này được thiết kế để fundamental-driven nhất có thể.** Mọi quyết định chiến lược trong báo cáo weekly broadcast — regime call, sector bias, 3 kịch bản VNINDEX, risk map, watchlist — đều phải dẫn dắt bằng **vĩ mô / cơ bản / chính sách / catalyst**. Phân tích kỹ thuật (PTKT) **không có vai trò quyết định** trong các trục cốt lõi của báo cáo.

Cụ thể áp dụng cho pack này:

- **Trigger 3 kịch bản VNINDEX phần 9 + Risk map**: primary phải là **vĩ mô / cơ bản / chính sách / catalyst**. Technical chỉ confirmation phụ (≤30% nội dung phần 9).
- **Sector bias phần 10**: driver chính là cơ bản + flow + catalyst. Technical KHÔNG quyết định bias.
- **Watchlist phần 10**: luận điểm mã PREFER cơ bản/catalyst/định giá. Technical chỉ làm confirmation observation.
- **Cap technical toàn báo cáo:** ≤ 15% (trừ phần 9.2 "Vùng giá tham chiếu" là vùng kỹ thuật pure render, không tính).

Triết lý này đảo ngược pattern phổ biến của retail VN (technical-first, fundamental sau) — vốn là nguồn alpha leak lớn nhất khi horizon ≥ 1 tuần forward-looking.

### 1.2. Wording chung cho cả 2 audience

Dùng dạng observation/luận điểm phân tích, **không dùng từ command** (mua, bán, giảm tỷ trọng, stop loss). Sector bias diễn đạt "quan tâm / thận trọng". Watchlist mã đề xuất qua luận điểm + catalyst + flow, **không kèm level giá vào/ra/stop**. Cách này đảm bảo:
- Nội bộ vẫn diễn dịch đủ thành action
- KH đọc được như góc nhìn phân tích để tự cân nhắc

### 1.3. Negative scope

- Không đọc state file của `P_invest_memo` (tier 6 portfolio, tier 7 weekly review)
- Không đọc state file của `P_vbse_strategy` (monthly/weekly cycle)
- Không khuyến nghị position cụ thể với level giá vào/ra/stop
- Không dùng từ command trực tiếp
- Không gán xác suất % cho kịch bản — dùng if-then trigger (xem `K_agent_db_00` mục 4.3)
- **Không sử dụng chỉ báo trend nội bộ** (`market_snapshot.trend`, `industry_snapshot.trend`, `group_snapshot.trend`, `series[].trend` trong các collection `*_recent`) — pack này có thể render cho KH; trend chỉ dùng ở pack audience nội bộ chuyên sâu
- Không phân tích portfolio cá nhân, không suggest rebalance
- Ngành ngoài 18 ngành whitelist không xuất hiện trong báo cáo

## 2. Naming & lưu trữ

**Naming file output:** `weekly_overview_<YYYYMMDD>.md` — YYYYMMDD là ngày kết thúc tuần (Chủ Nhật).

**Lưu trữ:** agent KHÔNG lưu file qua session. User tự archive sau khi nhận. Khi cần file W-1 cho phần Review, agent yêu cầu user upload.

## 3. Từ điển thuật ngữ cốt lõi

| Thuật ngữ | Giải thích |
|---|---|
| **Regime tuần** | Trạng thái tổng thể thị trường tuần qua. 4 mức: risk-on full / risk-on selective / defensive only / đứng ngoài (chi tiết `_04` mục 2). |
| **Conviction** | HIGH / MID / LOW — áp dụng cho regime call, sector bias mỗi ngành, watchlist mỗi mã. Chuẩn institutional. |
| **Horizon** | 1-2 tuần (default broadcast weekly) / 2-4 tuần (mã có catalyst rõ ngày). |
| **Sector bias** | Phân tầng định tính: quan tâm / trung tính / cần thận trọng. KHÔNG dùng "overweight/underweight". |
| **Whitelist 18 ngành** | Pack chỉ phân tích 18 ngành trong scope (`K_agent_db_00` mục 4.5 + `K_agent_db_01` Section B). |
| **Disconfirming signal** | "What would change our mind" — 2-3 chỉ báo cụ thể sẽ invalidate call. Bắt buộc kèm mỗi regime/sector/mã. |
| **3 kịch bản** | Cơ sở / Tích cực / Tiêu cực — trigger primary fundamental, technical confirmation phụ. Không gán % xác suất. |
| **Risk map** | 3-7 rủi ro tuần tới, mỗi rủi ro 4 thành phần: bản chất / signal materialize / phản ứng định tính / theme-sector bị invalidate. |
| **Magnitude (vĩ mô)** | Small / Material / Significant — đánh giá mức biến động chỉ số vĩ mô tuần (phần 5). |
| **Persistence (vĩ mô)** | Transitory / Trending / Structural — đánh giá tính chất biến động (1 lần / xu hướng / cấu trúc dài hạn). |
| **Checkpoint** | Điểm dừng bắt buộc sau Stage 1 — agent xuất block, user confirm/override trước khi qua Stage 2. Pack có 1 checkpoint duy nhất. |
| **Late money / trap** | Pattern mã thuộc ngành "cần thận trọng" nhưng có top dòng tiền — warning sign chuẩn buy-side. |
| **Earnings beat candidate** | Mã/ngành có cơ bản tốt + định giá rẻ + flow improving — ứng viên surprise tích cực mùa BCTC. |

## 4. Weight balance — Rule trọng số (BẤT BIẾN)

Đây là rule quan trọng nhất của pack — quyết định signal nào là driver chính, signal nào chỉ confirmation. Áp dụng cho weekly broadcast horizon:

| Tầng | Trọng số | Phần báo cáo | Loại signal |
|---|---|---|---|
| **PRIMARY (~65-70%)** | Phần 5 vĩ mô-hàng hoá + Phần 6 ngành (cơ bản angle) + Phần 8 tin tức + Phần 9.3 trigger kịch bản + Phần 10 watchlist (cơ bản angle) | Vĩ mô + cơ bản + chính sách + catalyst |
| **SECONDARY (~20-25%)** | Phần 4 thị trường VN flow + Phần 6 ngành (flow angle) + Phần 7 top dẫn dắt | Định vị + dòng tiền + breadth + NN/TD |
| **TERTIARY (~10-15%)** | Phần 9.1 diễn biến giá + Phần 9.2 vùng giá tham chiếu | Technical (MA, Fibonacci, POC, vùng kỹ thuật) |

### 4.1. Cap technical cứng

| Phần | Cap technical |
|---|---|
| Phần 1 Tóm tắt | 0% (chỉ kết luận cơ bản + flow) |
| Phần 3 Quốc tế | 0% (chỉ số macro + flow quốc tế) |
| Phần 4 VN | ≤ 10% (vùng kỹ thuật VNINDEX có thể minh hoạ phụ) |
| Phần 5 Vĩ mô-hàng hoá | 0% |
| Phần 6 Ngành | ≤ 5% (không có cột technical_zone trong bảng quyết định bias) |
| Phần 7 Top dẫn dắt | ≤ 10% (so sánh giá vs dòng tiền là PRIMARY) |
| Phần 8 Tin tức | 0% |
| Phần 9.1 + 9.2 Diễn biến + Vùng giá | Render technical pure, không tính cap |
| Phần 9.3 Trigger 3 kịch bản | **≤ 30%** technical (chỉ confirmation phụ; primary là macro/fundamental/policy) |
| Phần 9.4 Risk map | 0% (trigger macro/fundamental/policy) |
| Phần 10 Watchlist | ≤ 15% (cơ bản + catalyst PREFER, technical chỉ observation phụ) |
| Phần 11 Lịch sự kiện | 0% |
| **Báo cáo tổng (trừ 9.1+9.2)** | **≤ 15%** |

### 4.2. Self-audit cap

Trước khi render, agent đếm % nội dung technical trong từng phần. Vi phạm cap → re-write trước khi xuất. Chi tiết self-audit ở `P_weekly_overview_04` mục 5.

## 5. Kiến trúc tổng thể

### 5.1. Sơ đồ pack

```
MASTER (P_weekly_overview_00) — file này
│
├── WORKFLOW
│   ├── Pre-flight + Stage 1 Phần 2-5  (P_weekly_overview_01)
│   ├── Stage 1 Phần 6-9               (P_weekly_overview_02)
│   └── Checkpoint 1 + Stage 2 Phần 10-12 + Phần 1 (P_weekly_overview_03)
│
└── META
    └── Methodology + Self-audit + Edge + Contract (P_weekly_overview_04)

RENDER SPEC (O_weekly_overview_00) — output MD final 12 phần
```

### 5.2. Bảng file index

| File | Nội dung | Khi nào đọc |
|---|---|---|
| `P_weekly_overview_00` | Master — philosophy, từ điển, weight balance, file index, nguyên tắc bất biến | Đầu mọi session |
| `P_weekly_overview_01` | Pre-flight 3 câu + Phần 2 Review W-1 (scorecard) + Phần 3 Quốc tế + Phần 4 Thị trường VN (aggregate 18 ngành) + Phần 5 Vĩ mô-hàng hoá (institutional table 5 cột) | Stage 1 first half |
| `P_weekly_overview_02` | Phần 6 Biến động 18 ngành whitelist + Phần 7 Top dẫn dắt (với cảnh báo trap) + Phần 8 Tin tức+catalyst + Phần 9 Định vị VNINDEX + 3 kịch bản fundamental-driven + Risk map | Stage 1 second half |
| `P_weekly_overview_03` | Checkpoint 1 (conviction + disconfirming) + Phần 10 Watchlist 2 hướng + Phần 11 Lịch sự kiện + Phần 12 Disclaimer + Phần 1 Executive summary (Key calls/Watch/Risk) | Stage 2 + render |
| `P_weekly_overview_04` | Methodology (regime classification 4 mức + sector bias logic + mapping vĩ mô-18 ngành + technical-as-noise rule) + Self-audit 12 item + Edge cases + Output contract | Trước render + khi cần methodology reference |
| `O_weekly_overview_00` | Render spec MD final 12 phần | Render output |

### 5.3. Bảng 12 phần báo cáo

| # | Phần | Nguồn data chính | File spec |
|---|---|---|---|
| 1 | Tóm tắt điều hành (Key calls / Watch / Risk) | Tổng hợp 11 phần còn lại | `_03` |
| 2 | Review tuần trước (scorecard) | File W-1 user upload + DB actual | `_01` |
| 3 | Bối cảnh quốc tế | `other_data.international.*` + web | `_01` |
| 4 | Thị trường Việt Nam | `market_snapshot`, `market_recent`, aggregate 18 ngành, `market_nntd`, `data_briefing` | `_01` |
| 5 | Vĩ mô & hàng hoá (institutional table 5 cột) | `other_data.macro.*`, `other_data.commodities.*` + web | `_01` |
| 6 | Biến động 18 ngành whitelist | `industry_snapshot` (18), `industry_finstats` (18), `industry_recent` | `_02` |
| 7 | Top dẫn dắt 2 góc nhìn + cảnh báo trap | `stock_snapshot` (aggregate top tăng/giảm), cross-check sector bias | `_02` |
| 8 | Tin tức & catalyst | `news_history_feed`, `news_history_content` + web | `_02` |
| 9 | Định vị VNINDEX + 3 kịch bản fundamental-driven + Risk map | `market_snapshot.technical_indicator` + macro/policy/catalyst data | `_02` |
| 10 | Watchlist (tách tích cực/tiêu cực) + conviction + horizon + disconfirming | `stock_snapshot`, `stock_finstats`, `stock_nntd` | `_03` |
| 11 | Lịch sự kiện tuần tới | Web search + `news_history_feed` forward | `_03` |
| 12 | Tuyên bố miễn trừ trách nhiệm | Branding info từ pre-flight | `_03` |

### 5.4. Workflow flow

```
Pre-flight (3 câu) → Stage 1 (compose phần 2-9) → Checkpoint 1 (regime + sector bias + conviction + disconfirming) → Stage 2 (compose phần 10-12 + phần 1) → Render
```

## 6. Bốn nguyên tắc Agent bất biến

**Nguyên tắc 1 — Fundamental-driven trigger.**

3 kịch bản VNINDEX phần 9 + Risk map có **trigger primary là vĩ mô/cơ bản/chính sách/catalyst**. Technical chỉ confirmation phụ (≤30% nội dung phần 9.3). Vi phạm = re-write trước khi render.

**Nguyên tắc 2 — Whitelist 18 ngành.**

Mọi query / aggregate / ranking / bảng / watchlist tuân scope 18 ngành whitelist (`K_agent_db_01` Section B). Aggregate proxy thị trường tính trên 18, không 24. Rank ngành **tự tổng hợp** theo `week_score` qua 18 ngành whitelist (DB không lưu `industry_rank` tĩnh — xem `K_agent_db_01` mục "Xếp hạng ngành"). Ngành ngoài whitelist KHÔNG xuất hiện.

**Nguyên tắc 3 — Conviction + Horizon + Disconfirming bắt buộc.**

Mỗi call (regime, sector bias ngành, watchlist mã) bắt buộc đủ 3 thành phần:
- **Conviction:** HIGH / MID / LOW (chuẩn institutional)
- **Horizon:** 1-2 tuần / 2-4 tuần
- **Disconfirming signal:** 2-3 chỉ báo cụ thể sẽ invalidate call

Đây là **mới so với pack `P_weekly_market` cũ** — chuẩn institutional buy-side report.

**Nguyên tắc 4 — Mỗi stage kết thúc bằng checkpoint, agent không tự chuyển stage.**

Checkpoint 1 sau Stage 1 — agent xuất block regime+sector bias+conviction+disconfirming, user confirm/override/refine trước khi qua Stage 2. Override ghi inline note trong phần liên quan + log trong metadata cuối.

## 7. Cơ chế checkpoint review

Pack có **1 checkpoint duy nhất** (Checkpoint 1), đặt giữa phần 9 và phần 10. Chi tiết format block xem `P_weekly_overview_03` mục 1.

**Quyền override + audit trail:**
- User có quyền override regime, sector bias, hoặc bổ sung view
- Override ghi inline note trong phần liên quan của báo cáo cuối
- Mention trong **User overlay log** ở metadata cuối (block ngắn 1-3 dòng tóm tắt view user inject + trạng thái xử lý: confirm / partial confirm / decline + lý do)
- Pack KHÔNG dùng file `audit_overrides.md` riêng

## 8. Hướng dẫn dùng trong Claude project

### 8.1. Setup project knowledge

**Layer meta (luôn active):** system prompt + `KERNEL_SKELETON.md`.

**Pack dependency:** `K_agent_db` (6 file `_00` đến `_05`).

**Pack này:** `P_weekly_overview_00` (master) + 4 file (`_01` đến `_04`).

**Render spec:** `O_weekly_overview_00`.

**State artifacts (runtime, user tự archive):**
- Báo cáo weekly_overview đã save (input cho phần Review W-1 lần sau)

### 8.2. Cách start session

> "Chạy báo cáo tổng quan thị trường tuần [DD/MM-DD/MM]" hoặc "weekly overview report" hoặc "viết báo cáo tuần"

Agent route:
1. Đọc `P_weekly_overview_00` (master)
2. Đọc `P_weekly_overview_01` (Pre-flight + Stage 1 first half)
3. Hỏi user 3 câu pre-flight
4. Compose phần 2-5 (theo `_01`)
5. Đọc `P_weekly_overview_02` → compose phần 6-9
6. Đọc `P_weekly_overview_03` → xuất Checkpoint 1
7. User confirm/override → compose phần 10-12 + phần 1
8. Self-audit theo `P_weekly_overview_04` mục 5
9. Render qua `O_weekly_overview_00` → file `weekly_overview_<YYYYMMDD>.md`

### 8.3. Cross-reference priority

Khi agent không chắc cách thực thi:

1. File workflow tương ứng (`P_weekly_overview_01/02/03`) — chi tiết compose phần
2. `P_weekly_overview_04` — methodology + self-audit + edge + contract
3. `P_weekly_overview_00` (file này) — philosophy, nguyên tắc bất biến, weight balance, từ điển
4. Pack `K_agent_db` (`_00` đến `_05`) — schema, query, methodology
5. `O_weekly_overview_00` — render spec
6. System prompt + `KERNEL_SKELETON.md` — chỉ khi vấn đề meta

Không tự suy luận vượt spec khi spec đã có. Spec chưa cover case cụ thể → hỏi user xác nhận trước khi tự quyết.
