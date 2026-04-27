# P_stock_pitch_00 — Master Workflow

Pack `P_stock_pitch` build báo cáo khuyến nghị mua mã đơn lẻ phục vụ gửi khách hàng (stock pitch). Chỉ khuyến nghị MUA (long only). Báo cáo dạng deliverable MD source, render pptx khi user yêu cầu. Pack độc lập với `P_invest_memo` — user gọi lúc nào cũng được, có hoặc không có memo cycle hiện tại.

Pack 1 file. Toàn bộ workflow + methodology nằm trong file này. Render spec ở `O_stock_pitch_00`.

## 1. Mục đích & scope

**Mục đích:** sinh báo cáo khuyến nghị mua mã cổ phiếu Việt Nam niêm yết, dùng gửi trực tiếp cho khách hàng (stock pitch). Báo cáo có cấu trúc **13-16 mục** flex theo số luận điểm (cover → tóm tắt → foundation → 4-7 luận điểm → execution → bear case → kịch bản → disclaimer). Convention "15 mục" trong file này dùng default 6 luận điểm; báo cáo thực tế: 4 luận điểm = 13 mục, 5 = 14, 6 = 15, 7 = 16.

**Mục tiêu cao nhất:** thuyết phục được KH có conviction để hành động, đồng thời bảo vệ được pháp lý và uy tín của broker khi recommendation không như kỳ vọng. Báo cáo phải:
- Có data hard, mỗi claim truy được nguồn
- Có variant perception explicit (khác consensus chỗ nào)
- Có steelman bear case honest (không "all win")
- Có 3 kịch bản if-then trigger objective (không gán xác suất)
- Có execution chi tiết nhưng kèm disclaimer "tham khảo, không bắt buộc"

**Input kỳ vọng:**
- Trigger từ user: "viết khuyến nghị [ticker]", "recommendation [ticker]", "báo cáo MUA [ticker]", "stock pitch [ticker]"
- Mã ticker cụ thể
- (Optional) memo P_invest_memo tier 5C của mã đó nếu user đã build trước đó
- (Optional) branding info — logo, tên công ty, hotline, website, phòng ban biên soạn

**Output kỳ vọng:** file MD theo structure rigid 15 mục `O_stock_pitch_00` quy định. Render pptx khi user yêu cầu.

**Negative scope:**
- Không khuyến nghị BÁN (rủi ro pháp lý — KH có thể đang nắm giữ, bị xem là gây hoang mang)
- Không khuyến nghị GIỮ (ít giá trị cho KH — đã nắm giữ rồi)
- Không khuyến nghị shortsell, derivative, options
- Không gán xác suất % cho 3 kịch bản — dùng if-then trigger objective (xem K_agent_db_00 mục 4.3)
- Không sử dụng chỉ báo trend (`market_snapshot.trend`, `industry_snapshot.trend`, `*_recent.recent_trend`) — pattern đã thiết lập từ P_weekly_market
- Không hứa hẹn lợi nhuận chắc chắn, không dùng từ "đảm bảo", "chắc chắn", "không thể giảm"

**Dependency:** `K_agent_db`. Pack đọc trước `K_agent_db_00` master, các file con khi cần (chủ yếu `_01` schema, `_02` query patterns, `_04` methodology PTCB + technical, `_05` news methodology).

## 2. Naming convention & lưu trữ

**Naming file output:** `stock_pitch_<TICKER>_<YYYYMMDD>.md` — ngày là ngày phát hành báo cáo.

**Lưu trữ:** agent KHÔNG lưu file qua các session. User tự archive.

## 3. Workflow tổng thể

Workflow 5 stage, 2 checkpoint:

```
─── Pre-flight ──────────────────────────────────────
  Hỏi user 3 câu: ticker, memo tier 5C có sẵn, branding info

─── Stage 1: Foundation (mục 3, 4) ──────────────────
  Mục 3  Hồ sơ doanh nghiệp
  Mục 4  Dữ liệu giao dịch phiên hiện tại + mức kỹ thuật

─── Stage 2: Thesis & Variant Perception (mục 5-10) ─
  4-7 luận điểm cốt lõi (flex theo mã, không cố định 6)
  Variant perception explicit: consensus nghĩ gì, thesis khác đâu

─── CHECKPOINT 1: Thesis review ─────────────────────
  Agent xuất draft thesis + variant perception
  User confirm / reject / yêu cầu thêm bớt luận điểm

─── Stage 3: Execution (mục 11, 12) ─────────────────
  Mục 11  Cấu trúc mua N lớp + cắt lỗ + khung thời gian
  Mục 12  N mốc chốt lời + R/R

─── Stage 4: Steelman Bear Case (mục 13) ────────────
  5-7 lo ngại bear case + phản biện
  BẮT BUỘC: 1-2 lo ngại còn yếu sau phản biện (honest)

─── CHECKPOINT 2: Bear case review ──────────────────
  Agent xuất bear case + lo ngại còn yếu
  User confirm có đủ conviction để recommend
  Nếu phản biện yếu → ABORT, không gửi KH

─── Stage 5: Closing (mục 1, 2, 14, 15) ─────────────
  Mục 1   Cover (sau cùng)
  Mục 2   Tóm tắt khuyến nghị (giá target, R/R)
  Mục 14  3 kịch bản if-then trigger
  Mục 15  Disclaimer (theo branding info)

─── Render & deliver ────────────────────────────────
  Compile MD theo O_stock_pitch_00 structure
  Save file stock_pitch_<TICKER>_<YYYYMMDD>.md
  Hỏi user có cần render pptx không
  Present file qua present_files tool
```

## 4. Pre-flight — hỏi user trước khi vào Stage 1

Agent hỏi 1 turn 3 câu, user trả lời rồi mới chạy:

```
Trước khi tôi bắt đầu compose báo cáo khuyến nghị, xác nhận 3 điểm:

1. Ticker mã cần recommend:
   [user nêu ticker cụ thể, ví dụ GEL, VNM, FPT]

2. Có memo P_invest_memo tier 5C của mã này không?
   (a) Có, tôi gửi đính kèm — agent sẽ đọc và base trên thesis sẵn có
   (b) Không có — agent build fresh từ DB

3. Branding & disclaimer info (báo cáo gửi KH cần đầy đủ):
   (a) Có — vui lòng cung cấp: tên công ty, logo (optional), hotline, website, phòng ban biên soạn, nội dung disclaimer mong muốn
   (b) Không cần — render bản plain (không khuyến khích cho báo cáo gửi KH)
```

User trả lời → agent ghi nhận + chạy Stage 1. Nếu user chọn 3(b), agent cảnh báo: "Báo cáo gửi KH cần có disclaimer rõ ràng để bảo vệ pháp lý. Nếu không cung cấp branding/disclaimer, báo cáo render dạng plain chỉ phù hợp dùng nội bộ. Khuyến nghị cung cấp ở câu 3."

## 5. Stage 1 — Foundation (mục 3, 4)

### 5.1. Mục 3 — Hồ sơ doanh nghiệp

**Query DB:**

1. `stock_info` filter ticker:
   - Tên công ty, mã sàn (HSX/HNX/UPCOM), ngày niêm yết
   - Vốn điều lệ, số CP lưu hành, vốn hoá thị trường
   - Tỷ lệ free float, cổ đông lớn nhất + % sở hữu
   - Tỷ lệ sở hữu nước ngoài hiện tại / trần
   - Ngành theo phân loại 24 ngành VBSE

2. `stock_finstats` slice quý + năm gần nhất:
   - Doanh thu, LNTT, LNST quý gần nhất + cùng kỳ
   - Tổng tài sản, vốn chủ sở hữu, nợ vay
   - ROE, ROA, biên LN gộp 3 quý gần nhất
   - Tỷ lệ nợ/vốn chủ, ICR (Interest Coverage Ratio)

3. `stock_finstats` deep dive cơ cấu kinh doanh:
   - Mảng kinh doanh chính (parent) + tỷ trọng doanh thu/LN
   - Công ty con/liên kết quan trọng + tỷ lệ sở hữu
   - Tài sản cốt lõi (BĐS, KCN, nhà máy, license, dòng tiền độc quyền)

**Output structured:**

Sub-section 3.1 Thông tin cơ bản: bảng 9-11 hàng (mã, ngày niêm yết, vốn điều lệ, CP lưu hành, vốn hoá, free float, cổ đông lớn nhất, sở hữu NN, ngành)

Sub-section 3.2 Cơ cấu kinh doanh & tài sản cốt lõi: liệt kê 3-6 mảng/công ty con quan trọng, mỗi mảng 1 đoạn 2-3 dòng (tên + tỷ lệ sở hữu + đặc điểm dòng tiền + tình trạng hiện tại)

### 5.2. Mục 4 — Dữ liệu giao dịch phiên hiện tại + mức kỹ thuật

**Query DB:**

1. `stock_snapshot` filter ticker (1 doc):
   - `price.open, high, low, close, volume, trading_value, volume_strength_index, diff, pct_change`
   - `change.w_pct, m_pct, q_pct, y_pct`
   - `money_flow_score.day_score, week_score, industry_rank_pct, market_rank_pct`
   - `technical_indicator.ma`: ma5, ma20, ma60, ma120, ma240
   - `technical_indicator.fibonacci.{w|m|q|y}`: `f382, f500, f618` (3 mức Fibonacci 38.2/50/61.8% — K_01 chỉ có 3 mức này)
   - `technical_indicator.volume_profile.{w|m|q|y}`: poc, val, vah
   - `technical_indicator.pivot.{w|m|q|y}`: pivot, r1, s1 (Classical Pivot — K_01 chỉ có pivot/r1/s1)
   - `technical_zone.overall.{w|m|q|y}` (zone đa khung — chỉ dùng nội bộ, dịch khi output)

   **Lưu ý:** `stock_snapshot` không có field `range_position`. Vị thế giá trong biên độ tuần/tháng/quý tự tính từ `price.close` và `technical_indicator.ohl.{w|m|q|y}.prev_high/prev_low`.

2. `stock_nntd` filter ticker:
   - `nn.latest.net_value` (phiên hiện tại), `nn.week.net_value` (5 phiên), `nn.month.net_value` (20 phiên)
   - `td.latest/week/month.net_value` cho tự doanh

3. `market_snapshot` (1 doc) tham chiếu — VNINDEX hôm đó để có context thị trường (`price.close`, `change.w_pct`, `breadth`)

**Output structured:**

Sub-section 4.1 Dữ liệu giao dịch phiên: 4 stat callout — đóng cửa + biến động phiên / cao nhất / thấp nhất / GTGD + cường độ thanh khoản (đã dịch từ `volume_strength_index` theo K hygiene)

Sub-section 4.2 Dòng tiền tổ chức phiên: NN + tự doanh, mỗi loại stat callout giá trị net phiên

Sub-section 4.3 Các mức kỹ thuật: bảng các vùng giá quan trọng theo thứ tự từ cao xuống thấp:
- Kháng cự khung tuần/tháng (Fibonacci, Pivot)
- Biên trên vùng giao dịch tuần (VAH)
- Vùng tập trung giao dịch tuần (POC)
- Giá hiện tại
- Biên dưới vùng giao dịch tuần (VAL)
- Hỗ trợ khung tuần/tháng
- MA60 / MA240 (nếu cần làm hỗ trợ sâu)

Mỗi mức kèm cột "+/- so giá hiện tại" để KH thấy rõ khoảng cách.

## 6. Stage 2 — Thesis & Variant Perception (mục 5-10)

**Quy tắc số luận điểm:** flex 4-7 tuỳ mã. Không ép 6.

### 6.1. Khung 7 luận điểm tiềm năng — chọn lọc tuỳ mã

| Luận điểm | Khi nào dùng | Khi nào skip |
|---|---|---|
| L1 — Định giá hấp dẫn | Mã có P/E hoặc P/B thấp so peer hoặc median 3Y, hoặc có variant perception về cách đọc đúng (P/E vs P/B vs EV/EBITDA) | Mã đã trade ở định giá cao hơn peer, không có variant perception |
| L2 — Dòng tiền tổ chức | NN + tự doanh có pattern tích luỹ rõ ràng tuần/tháng | NN/tự doanh trung tính hoặc bán ròng |
| L3 — Kỹ thuật vùng mua tối ưu | Giá đang test vùng hỗ trợ kỹ thuật quan trọng + lực bán cạn | Giá đang ở đỉnh hoặc giữa vùng |
| L4 — Catalyst ngắn hạn | Có 2-4 tin/sự kiện trong 1-3 tháng tới có thể đẩy giá | Không có catalyst rõ ràng |
| L5 — Catalyst trung hạn | Có dự án/chính sách/M&A quy mô lớn (≥ 6 tháng) | Mã chỉ có thesis kỹ thuật ngắn hạn |
| L6 — Vị thế cạnh tranh | Mã có moat rõ (độc quyền, license, tài sản khó thay thế, brand) | Mã không có lợi thế cạnh tranh đặc biệt — không nên bịa |
| L7 — Catalyst sự kiện corporate đặc biệt | Sắp có ĐHCĐ, divestment, niêm yết bổ sung, chia tách | Không có sự kiện corporate đặc biệt |

**Tối thiểu 4 luận điểm.** Dưới 4 = thesis yếu, không nên recommend, abort sang yêu cầu user chọn mã khác.

**Tối đa 7 luận điểm.** Trên 7 = báo cáo dài, KH khó scan, ép luận điểm yếu vào.

### 6.2. Compose từng luận điểm

**Format chung mỗi luận điểm:**

- **Tiêu đề luận điểm:** ngắn gọn 1 câu đứng riêng
- **Headline data point:** 1 con số/fact lớn nhất (stat callout dạng 60-72pt khi render pptx)
- **3-4 sub-points:** mỗi sub-point có claim + data + nguồn
- **Bảng tham chiếu (nếu có):** số liệu so sánh peer hoặc lịch sử

**Source query theo loại luận điểm:**

| Luận điểm | Collection chính | Field/aggregate |
|---|---|---|
| L1 Định giá | `stock_finstats.valuation_ratios[]` + `industry_finstats.valuation_ratios[]` (peer comparison) | P/E, P/B, EV/EBITDA, BVPS — đọc từ `valuation_ratios[]` array với `vi_name` lookup. Định giá ngành KHÔNG nằm trong `industry_snapshot` (xem K_01 mục B) |
| L2 Dòng tiền | `stock_nntd` (`nn.week/month.net_value`), `stock_snapshot.money_flow_score` | Net_value tuần/tháng, week_score, day_score; aggregate 5/20 phiên đã sẵn trong doc |
| L3 Kỹ thuật | `stock_snapshot.technical_zone, technical_indicator.{fibonacci, volume_profile, pivot}` | Vùng giá, `price.volume_strength_index`, vị thế tự tính từ `ohl` |
| L4-L5 Catalyst | `news_history_feed` filter ticker + ngành, web search | Tin tuần/tháng/quý gần nhất, dẫn link finext.vn |
| L6 Vị thế | `stock_info` + web search peer comparison | Market share, license, partnership |
| L7 Sự kiện | `news_history_feed` filter type thông cáo | Sự kiện corporate sắp tới |

### 6.3. Variant Perception — bắt buộc explicit

**Vị trí:** lồng vào luận điểm phù hợp nhất (thường L1 định giá hoặc L4 catalyst). Không phải mục riêng nhưng phải có và dễ identify.

**Format 3 câu:**

1. **Consensus sell-side / báo phổ thông nghĩ gì về mã này?** (1-2 dòng)
2. **Consensus retail / room nghĩ gì?** (1-2 dòng — nếu khác sell-side)
3. **Thesis của báo cáo này khác consensus chỗ nào?** (2-3 dòng giải thích insight chính)

**Ví dụ chuẩn (mẫu GEL):**

> Consensus sell-side hiện đọc GEL theo P/E 54x — kết luận "quá đắt". Retail cũng nhìn P/E và bỏ qua. **Thesis này khác:** GEL là doanh nghiệp hạ tầng thâm dụng tài sản với tổng tài sản 46.925 tỷ — phải đọc theo P/B (1,37x) hoặc EV/EBITDA (9,94x) chứ không phải P/E. P/E 54x phản ánh chi phí lãi vay từ khoản hợp vốn 200 triệu USD và chi phí chuẩn bị niêm yết — tạm thời, không phải yếu kém kinh doanh.

**Variant perception yếu = thesis yếu.** Nếu không tìm được điểm khác consensus rõ ràng (mã đang được consensus đánh giá đúng) → recommend rủi ro cao vì không có thị trường mismatch để khai thác. Trong trường hợp này, agent flag CAUTION + downgrade self-conviction theo 4 dấu hiệu ở mục 7.1, user quyết tại checkpoint 1 (consistent triết lý flex+user-quyết, không auto-reject).

### 6.4. Output Stage 2

Compose 4-7 mục (đánh số mục 5 đến mục `5+N-1` với N = số luận điểm). Cụ thể:
- 4 luận điểm → mục 5, 6, 7, 8 (tổng báo cáo 13 mục)
- 5 luận điểm → mục 5-9 (14 mục)
- 6 luận điểm → mục 5-10 (15 mục, default)
- 7 luận điểm → mục 5-11 (16 mục, các mục sau dịch lên 1: cấu trúc mua → mục 12, chốt lãi → 13...)

Mỗi mục theo format luận điểm. Variant perception lồng vào 1 luận điểm. Numbering các mục sau (cấu trúc mua, chốt lãi, bear, kịch bản, disclaimer) shift theo số luận điểm — O pack render adjust số section tương ứng.

Sau Stage 2, **không in ra báo cáo final**. Agent compose draft thesis + variant perception, xuất ra checkpoint 1.

## 7. Checkpoint 1 — Thesis Review

Đặt giữa Stage 2 và Stage 3.

### 7.1. Block xuất tại checkpoint 1

```
─── THESIS REVIEW — Trước khi build execution ───

**Mã:** [TICKER]
**Số luận điểm đã compose:** [N] (tối thiểu 4, tối đa 7)

**Tóm tắt từng luận điểm:**
- L1 [Tiêu đề]: [headline data point + 1 dòng tóm tắt]
- L2 [...]: [...]
- ...

**Variant Perception:**
- Consensus sell-side: [1-2 dòng]
- Consensus retail: [1-2 dòng nếu khác]
- Thesis khác consensus: [2-3 dòng insight chính]

**Self-assessment variant perception** (theo 4 dấu hiệu objective, consistent P_invest_memo_07 Gate 1):

| Dấu hiệu | Action đề xuất |
|---|---|
| Có differentiation rõ + evidence cụ thể + catalyst + timing | **PASS**, conviction đầy đủ |
| Differentiation về giá (undervalued) nhưng không có catalyst để re-rate | **CAUTION** — flag "value trap risk" + downgrade conviction 1 bậc |
| Có differentiation nhưng evidence chỉ là wishful thinking, không data hard | **CAUTION** — flag "wishful thinking" + downgrade conviction 1 bậc |
| Variant view ≈ consensus (không differentiation rõ) | **CAUTION** — flag "buy vào consensus, thiếu alpha" + downgrade conviction 1 bậc |

Agent không tự reject — đưa thông tin đầy đủ, user quyết proceed (size nhỏ + audit log) hoặc loại mã.

Confirm hay điều chỉnh trước khi build strategy execution?
- (a) Confirm thesis như trên, tiếp Stage 3
- (b) Cần thêm luận điểm → [user nêu loại luận điểm cần thêm]
- (c) Cần bớt luận điểm → [user nêu loại bỏ]
- (d) Variant perception yếu hoặc không thuyết phục → review lại
- (e) Thesis tổng thể yếu → ABORT, không tiếp tục
```

### 7.2. Xử lý phản hồi user

| User chọn | Action |
|---|---|
| (a) Confirm | Stage 3 chạy thẳng |
| (b) Thêm luận điểm | Agent query bổ sung theo yêu cầu, compose luận điểm mới, xuất lại checkpoint 1 |
| (c) Bớt luận điểm | Agent loại bỏ luận điểm chỉ định, xuất lại checkpoint 1 |
| (d) Refine variant perception | Agent đào sâu data, refine 3 câu variant perception, xuất lại |
| (e) Abort | Workflow dừng. Agent gợi ý: "Mã này thesis hiện chưa đủ mạnh để recommend. Có thể chờ thêm tín hiệu (catalyst, dòng tiền cải thiện) hoặc chọn mã khác." Không tiếp Stage 3. |

## 8. Stage 3 — Execution (mục 11, 12)

### 8.1. Mục 11 — Cấu trúc mua N lớp

**Methodology:**

Cấu trúc 3 lớp chuẩn (có thể flex 2-4 lớp tuỳ tình huống mã):

| Lớp | Vai trò | Vùng giá | Tỷ trọng % | Trigger |
|---|---|---|---|---|
| Lớp 1 — Mua thăm dò | Vào trước khi confirm rõ | Biên dưới vùng giao dịch tuần / hỗ trợ gần | 30-40% | Lực bán cạn (vsi thấp), giá test hỗ trợ và bật |
| Lớp 2 — Mua khi xác nhận | Confirm tín hiệu hấp thụ | Test lại đáy phiên 1 với volume thấp hơn / Fibonacci 38.2% | 40-50% | Volume test thấp hơn lần đầu = lực bán đã cạn |
| Lớp 3 — Mua gia tăng | Bứt phá xác nhận đảo chiều | Vượt biên trên vùng giao dịch tuần / kháng cự gần | 20-30% | Volume break-out > 1.2x trung bình + đóng cửa trên kháng cự |

**Quy tắc disclaim tỷ trọng %:**

Ngay sau bảng cấu trúc 3 lớp, BẮT BUỘC ghi:
> *Tỷ trọng % phân bổ là gợi ý tham khảo, không bắt buộc. Khách hàng tự cân nhắc theo size portfolio, mức chịu đựng rủi ro và position sizing cá nhân.*

**Quản trị rủi ro — phải có 3 thông số:**

- **Cắt lỗ cứng:** mức giá cụ thể, tính khoảng -X% từ giá vào trung bình (thường -3% đến -5%). Dựa trên hỗ trợ kỹ thuật rõ rệt (S1 tuần, biên dưới VAL, MA20)
- **Cắt lỗ dự phòng:** mức giá thấp hơn cắt lỗ cứng, dùng khi giá gãy support cấu trúc lớn hơn (đáy tháng, MA60)
- **Khung thời gian:** ngắn hạn N tuần (4-8 tuần thường) + core position N tháng (6-12 tháng cho catalyst lớn)

### 8.2. Mục 12 — Mục tiêu chốt lời theo N mốc

**Methodology:**

N mốc chốt lời (flex 3-5 tuỳ mã):

| Mốc | Vai trò | Vùng giá | Tỷ trọng chốt | Cơ sở kỹ thuật |
|---|---|---|---|---|
| M1 — Chốt lãi đầu | Đảm bảo lợi nhuận đầu tiên | Fibonacci 38.2% khung tuần / kháng cự gần | 20-30% | Vùng kháng cự đầu tiên |
| M2 — Chốt lãi giữa | Khi đạt mức kháng cự rõ rệt | R1 tuần / Fibonacci 50% khung tháng | 20-30% | Kháng cự kỹ thuật mạnh |
| M3 — Chốt lãi mục tiêu chính | Đạt target trung hạn | Biên trên vùng giao dịch tháng / Fibonacci 61.8% | 20-30% | Mục tiêu kỹ thuật chính |
| M4 — Mục tiêu trung hạn cao | Khi catalyst lớn materialize | Đỉnh quý / VAH khung quý / vùng tâm lý | Phần còn lại | Mục tiêu trung hạn |

**Tóm tắt R/R:**

- Lợi nhuận bình quân các mốc đầu (trừ M cuối cùng): +X%
- Rủi ro tối đa đến điểm cắt lỗ: -Y%
- Tỷ lệ R/R = X/Y (ví dụ 1:2.1)
- Mục tiêu trung hạn cao nhất: +Z%

**Đánh giá R/R:**

| R/R | Nhận xét | Action |
|---|---|---|
| 1:1 hoặc thấp hơn | Không hấp dẫn | Flag cảnh báo mạnh ở CP2 + downgrade self-conviction, user quyết định |
| 1:1.5 đến 1:2 | Tạm chấp nhận | Flag note ở CP2, user xác nhận |
| 1:2 đến 1:3 | Hấp dẫn | Pass |
| > 1:3 | Rất hấp dẫn | Pass nhưng kiểm tra kỳ vọng phi thực tế (target overshoot?) |

Agent không tự reject mã do R/R thấp — đưa thông tin đầy đủ ở checkpoint 2, user quyết. Triết lý flex+user-quyết consistent với P_invest_memo gate logic.

## 9. Stage 4 — Steelman Bear Case (mục 13)

**Đây là gate quan trọng nhất.** Khác với Stage 1-3 là build long thesis, Stage 4 đứng vào vai trò bear, tìm điểm yếu.

### 9.1. Methodology

**Liệt kê 5-7 lo ngại bear case mạnh nhất:**

Lấy từ 4 nguồn:
1. **Định giá:** P/E cao, P/B đắt so peer, EV/EBITDA cao, không có discount cho rủi ro
2. **Dòng tiền:** NN bán ròng tuần, tự doanh trung tính, dòng tiền cốt lõi yếu
3. **Tài chính:** đòn bẩy cao (D/E > 1), ICR thấp (< 2), biên LN suy giảm, KQKD không đạt kế hoạch
4. **Sự kiện:** mới niêm yết, free float thấp, catalyst dài hạn không chắc chắn, ngành áp lực vĩ mô

**Phản biện cho mỗi lo ngại:**

Format mỗi lo ngại 2 phần:
- **Lo ngại:** 1-2 dòng phát biểu rõ
- **Phản biện:** 2-3 dòng — số liệu cụ thể giải thích tại sao lo ngại này không phải deal-breaker

### 9.2. Honest steelman — bắt buộc

**Sau khi phản biện 5-7 lo ngại, BẮT BUỘC ghi rõ 1-2 lo ngại còn yếu sau phản biện.**

Đây là điểm khác biệt quan trọng với mẫu VBSE GEL (phản biện all win, không thừa nhận lo ngại nào còn yếu). Real bear case phải có ít nhất 1-2 điểm phản biện chỉ "tạm chấp nhận" chứ không "hoàn toàn không phải vấn đề".

**Format honest steelman:**

```
**Lo ngại còn yếu sau phản biện:**

[Tên lo ngại 1]: phản biện chỉ giải quyết được [phần X]. Điểm yếu còn lại: [Y]. Tác động nếu materialize: [Z — mức độ ảnh hưởng đến thesis tổng thể].

[Tên lo ngại 2 nếu có]: ...
```

**Ví dụ honest steelman:**

> Lo ngại còn yếu sau phản biện: Mới niêm yết HOSE 2.5 tháng + free float 28%. Phản biện "lợi thế chưa được nhiều quỹ theo dõi" chỉ đúng nếu mã được vào VN30/FTSE. Nếu chưa được vào, mã có thể bị thiếu thanh khoản kéo dài, hấp dẫn tổ chức không bù đắp được rủi ro thanh khoản với KH retail. Tác động: kịch bản trung hạn (+18%) phụ thuộc một phần vào catalyst index inclusion — nếu không xảy ra, target trung hạn có thể chỉ +10-12%.

### 9.3. Output Stage 4

Compose mục 13 đầy đủ: 5-7 lo ngại + phản biện + 1-2 lo ngại còn yếu.

Sau Stage 4, không in ra báo cáo final. Agent compose summary bear case + lo ngại còn yếu, xuất ra checkpoint 2.

## 10. Checkpoint 2 — Bear Case Review

Đặt giữa Stage 4 và Stage 5. Đây là **gate cuối cùng trước khi commit recommendation**.

### 10.1. Block xuất tại checkpoint 2

```
─── BEAR CASE REVIEW — Gate cuối trước recommend ───

**Mã:** [TICKER]
**Số lo ngại đã phản biện:** [N] (5-7)

**Tóm tắt phản biện:**
- Lo ngại 1: [tóm tắt 1 dòng] → phản biện: [Mạnh / Vừa / Yếu]
- Lo ngại 2: [...] → [...]
- ...

**Lo ngại CÒN YẾU sau phản biện:**

[Tên lo ngại A]: 
- Điểm yếu còn lại: [...]
- Tác động nếu materialize: [...]

[Tên lo ngại B nếu có]: ...

**Self-assessment conviction:**
- R/R: 1:[X] — [Hấp dẫn / Tạm chấp nhận / Không hấp dẫn]
- Conviction tổng thể: [Cao / Vừa / Thấp]
- Lý do: [1-2 dòng]

Confirm có đủ conviction để recommend KH?
- (a) Confirm — phản biện đủ mạnh, lo ngại còn yếu trong giới hạn chấp nhận → tiếp Stage 5
- (b) Lo ngại còn yếu là deal-breaker → ABORT, không gửi KH
- (c) Cần phản biện thêm cho lo ngại còn yếu → review
- (d) Cần bổ sung lo ngại bear case còn miss → review
```

### 10.2. Xử lý phản hồi user

| User chọn | Action |
|---|---|
| (a) Confirm | Stage 5 chạy thẳng |
| (b) Abort | Workflow dừng. Ghi note: "Conviction không đủ để recommend KH. Pack abort tại checkpoint 2." Agent gợi ý: "Có thể chờ thêm tín hiệu giải quyết được lo ngại còn yếu (catalyst đến, định giá điều chỉnh) hoặc chọn mã có thesis sạch hơn." |
| (c) Phản biện thêm | Agent đào sâu data, refine phản biện cho lo ngại còn yếu, xuất lại checkpoint 2 |
| (d) Bổ sung lo ngại | Agent thêm 1-2 lo ngại theo yêu cầu, phản biện, xuất lại |

**Quan trọng:** checkpoint 2 chính là điểm differentiate giữa báo cáo recommendation chuyên nghiệp và báo cáo "all-bullish" thiếu objectivity. Nếu user thường xuyên chọn (a) Confirm cho mọi mã, agent có thể flag pattern này ở turn sau ("Đã 5 báo cáo gần nhất đều confirm Stage 4 không abort. Có thể conviction đang bị bias bullish."). Nhưng không enforce — user là người quyết.

## 11. Stage 5 — Closing (mục 1, 2, 14, 15)

Sau khi qua checkpoint 2, agent compose nốt 4 mục cuối.

### 11.1. Mục 1 — Cover (compose sau)

**Output structured:**
- Tên báo cáo: BÁO CÁO PHÂN TÍCH [TICKER]
- Tên đầy đủ công ty
- Khuyến nghị: MUA
- Giá hiện tại
- Mục tiêu trung hạn (= mục tiêu cao nhất từ mục 12, ví dụ M4)
- Tiềm năng tăng giá: +X%
- Tiêu đề thesis chính (1 câu — captured insight chính của báo cáo)
- Ngày phát hành
- Branding info nếu user cung cấp (logo, tên công ty, hotline, website, phòng ban)

### 11.2. Mục 2 — Tóm tắt khuyến nghị

**Format:**

```
Luận điểm MUA [TICKER] quanh vùng giá [X] đồng

**Giá mục tiêu ngắn hạn:** [Y] đồng (+A%)
**Giá mục tiêu trung hạn:** [Z] đồng (+B%)
**Điểm cắt lỗ:** [W] đồng (-C%)
**Tỷ lệ R/R:** 1:[N] [Hấp dẫn / Tạm chấp nhận]

**LUẬN ĐIỂM CỐT LÕI:**

01 [Tiêu đề L1]
[2-3 dòng tóm tắt + headline data point]

02 [Tiêu đề L2]
[...]

03 [Tiêu đề L3]
[...]

04 [Tiêu đề L4 nếu có, max 4 luận điểm in tóm tắt — nếu báo cáo có 5-7 luận điểm thì gộp vào 4 tóm tắt mạnh nhất]
[...]
```

### 11.3. Mục 14 — Tóm tắt hành động + 3 kịch bản if-then

**Hành động đề xuất** (đầu mục):

```
**HÀNH ĐỘNG ĐỀ XUẤT:**

MUA [TICKER] vùng [X] - [Y] đồng. Mua thăm dò [N1]% vị thế [thời điểm/điều kiện 1], chờ test lại [Z]-[W] để mua tiếp [N2]%, giữ [N3]% cho bứt phá vượt [V] đồng.

*Tỷ trọng % là gợi ý tham khảo, không bắt buộc. Khách hàng tự cân nhắc theo size portfolio và position sizing cá nhân.*
```

**3 kịch bản if-then trigger** (BỎ xác suất):

```
**KỊCH BẢN CƠ SỞ**

**Trigger duy trì:** [điều kiện cụ thể — ví dụ: giá giữ trên POC tuần X + dòng tiền tổ chức tháng vẫn dương + không có shock vĩ mô]

**Khung thời gian:** [N tuần / tháng]
**Giá mục tiêu:** [vùng X-Y đồng]
**Tỷ suất kỳ vọng:** [+A-B%]
**Hành vi kỳ vọng:** [tóm tắt diễn biến + action gợi ý — ví dụ: hồi phục về vùng trước điều chỉnh khi catalyst N materialize. Đóng 50% lãi tại M2, giữ phần còn lại cho trung hạn.]

**KỊCH BẢN TÍCH CỰC**

**Trigger break-out:** [điều kiện cụ thể — ví dụ: tin chính thức về catalyst lớn (Sân bay, M&A) + breakout vượt kháng cự X với volume > 1.2x trung bình + NN mua ròng tăng cường]

**Khung thời gian:** [N tháng]
**Giá mục tiêu:** [vùng X-Y đồng — thường là target trung hạn cao nhất]
**Tỷ suất kỳ vọng:** [+A-B%]
**Hành vi kỳ vọng:** [vượt đỉnh cũ, định giá lại theo thesis variant perception]

**KỊCH BẢN TIÊU CỰC**

**Trigger break-down:** [điều kiện cụ thể — ví dụ: gãy hỗ trợ X (đáy tháng / MA60) + dòng tiền tổ chức đảo chiều bán ròng + lo ngại còn yếu materialize]

**Khung thời gian:** [N tuần]
**Giá mục tiêu:** [điểm cắt lỗ — ví dụ X đồng]
**Tỷ suất kỳ vọng:** [-Y%]
**Hành vi kỳ vọng:** thực thi điểm cắt lỗ kỷ luật, không ảnh hưởng vốn dài hạn của KH.
```

**Note kỹ thuật bắt buộc cuối mục 14:**
> *Các kịch bản là hệ thống điều kiện kỹ thuật, không phải dự báo chắc chắn. Diễn biến thực tế có thể lệch khỏi cả 3 nếu xuất hiện sự kiện ngoài kỳ vọng.*

### 11.4. Mục 15 — Disclaimer

**Output structured:**

Render đầy đủ disclaimer theo branding info user cung cấp ở pre-flight. Nếu user không cung cấp (chọn 3b ở pre-flight), dùng template default theo `O_stock_pitch_00`.

## 12. Render & deliver

Sau khi có đủ 15 mục structured content, agent gọi `O_stock_pitch_00` để render thành file MD final.

File output: `stock_pitch_<TICKER>_<YYYYMMDD>.md`, save vào `/mnt/user-data/outputs/`, gọi `present_files`.

Sau khi present MD, agent hỏi user:
> "MD đã sẵn sàng. Có cần render thêm pptx (15 slide branded) không?"

Nếu có → render pptx theo `O_stock_pitch_00` mục pptx (T pack runtime quyết theo brand audience).

## 13. Edge cases & xử lý

### 13.1. Mã không có thesis đủ mạnh

Sau Stage 2, nếu chỉ tìm được < 4 luận điểm có data hard, hoặc variant perception yếu → ghi note vào checkpoint 1, đề xuất user chọn mã khác hoặc chờ thêm tín hiệu.

### 13.2. R/R không hấp dẫn

Sau Stage 3, nếu R/R < 1:1.5 → flag rõ ở checkpoint 2 + downgrade self-conviction tổng thể, user quyết. Mã có R/R thấp thường không nên gửi KH dù conviction về fundamentals cao, nhưng quyết định cuối thuộc user (consistent triết lý flex). Audit log nếu user proceed.

### 13.3. Conviction sau bear case yếu

User chọn (b) abort ở checkpoint 2 → workflow dừng, agent không tự tiếp tục. File MD chưa render, không lưu.

### 13.4. User yêu cầu nhiều mã trong 1 session

Pack chạy 1 mã/lần. Nếu user yêu cầu 3 mã, agent confirm: "Sẽ chạy lần lượt từng mã. Bắt đầu với [mã 1]?"

### 13.5. Memo P_invest_memo tier 5C có sẵn

User cung cấp file tier 5C ở pre-flight 2(a) → agent đọc file và absorption nội dung theo mapping:

| Memo tier 5C (P_invest_memo_07) | Stock pitch (pack này) | Lưu ý transform |
|---|---|---|
| Phần 1 Recommendation + Thesis | Mục 2 Tóm tắt khuyến nghị + tiêu đề thesis chính | Memo có Buy/Pass/Watch — stock pitch chỉ Buy. Nếu memo decision Pass/Watch → ABORT, không build pitch |
| Phần 2 Variant Perception | Variant perception lồng vào mục 5 (luận điểm L1) | Memo 4 câu (consensus / khác / evidence / khi nào) → stock pitch 3 câu (consensus sell-side / retail / thesis khác). Gộp evidence + timing vào câu 3 |
| Phần 3 Business Overview | Mục 3 Hồ sơ doanh nghiệp + thông tin định tính các luận điểm | Memo độ dài 0.5-1 trang → stock pitch ngắn hơn, focus mảng kinh doanh chính |
| Phần 4 Financial + Valuation + Target Base/Bull/Bear | Mục 4 (technical) + L1 định giá + mục 11 + mục 12 | Target memo (Base/Bull/Bear) → mục 11 entry levels (margin of safety) + mục 12 chốt lời mốc + mục 14 kịch bản giá |
| Phần 5 Catalysts | Luận điểm L4-L5-L7 (catalyst) trong mục 5-10 | Memo 2-4 catalyst với 3 đặc tính → stock pitch luận điểm catalyst với headline data point + sub-point |
| Phần 6 Bear Case Steelmanned | Mục 13 Bear case + lo ngại còn yếu | **Format khác:** memo 3-5 arguments + probability-weighted target (audience analyst nội bộ) → stock pitch 5-7 lo ngại + phản biện + 1-2 còn yếu honest (audience KH). KHÔNG dùng probability-weighted target trong stock pitch (KH không hiểu methodology). Honest steelman 1-2 còn yếu lấy từ rebuttal yếu nhất của memo |
| Phần 7 Monitoring + Exit Triggers | Mục 11 cắt lỗ + mục 12 chốt lãi + mục 14 trigger break-down | Hard trigger memo (price < Bear × 0.9, BCTC fail) → stock pitch trigger break-down kịch bản thận trọng. Soft trigger memo → mục 14 trigger duy trì kịch bản cơ sở |

**Vẫn phải qua đủ 2 checkpoint.** Memo tier 5C giảm thời gian compose nhưng không bypass review — wording, audience, format khác (KH vs analyst nội bộ).

**Constraint quan trọng:**
- Stock pitch KHÔNG dùng probability-weighted bear target từ memo (audience KH).
- Stock pitch KHÔNG dùng chỉ báo trend/zone code raw từ memo (K hygiene cho KH).
- Memo decision Pass/Watch → stock pitch ABORT, agent thông báo "Memo tier 5C không recommend Buy, không build pitch".

### 13.6. Tin xấu mới về mã giữa workflow

Trong khi compose, nếu agent phát hiện tin tiêu cực mới trong news_history_feed (ví dụ thông cáo tiêu cực, KQKD gây sốc, lãnh đạo từ chức) → ngay lập tức flag ở checkpoint gần nhất, không silent.

### 13.7. Mã nằm trong ngành sector bias "thận trọng" của P_weekly_market

Pack độc lập, không đọc state P_weekly_market. Nhưng user trong session có thể đã chạy P_weekly_market và biết ngành đang trong mode thận trọng. Đây là decision của user — pack không tự enforce. Tuy nhiên, nếu user trong session cùng đã call regime defensive only và mã thuộc ngành tránh, agent có thể flag soft: "Mã thuộc ngành đang trong sector bias thận trọng theo báo cáo tuần. Vẫn tiếp tục recommendation?"

## 14. Self-audit trước khi xuất file

Chạy 12 câu trước khi present file:

1. Mã ticker + ngày phát hành đã chính xác?
2. Số luận điểm trong khoảng 4-7?
3. **Variant perception có explicit và rõ ràng không** (3 câu chuẩn: consensus sell-side, retail, thesis khác)?
4. Stage 1 (foundation) có data hard không bị nguỵ tạo?
5. Stage 3 (execution) đã có disclaimer "tham khảo, không bắt buộc"?
6. Stage 4 (bear case) có 5-7 lo ngại + 1-2 lo ngại CÒN YẾU sau phản biện không?
7. Mục 14 ba kịch bản dùng if-then trigger, **không có % xác suất**?
8. Mục 14 có note "kịch bản là hệ thống điều kiện kỹ thuật, không phải dự báo chắc chắn"?
9. Cả 2 checkpoint đã được user confirm trước khi compose tiếp?
10. Mục 15 disclaimer đầy đủ theo branding info user cung cấp?
11. **Đã không dùng bất kỳ chỉ báo trend nào** (`market_snapshot.trend`, `industry_snapshot.trend`, `*_recent.recent_trend`)?
12. K hygiene: ký hiệu DB raw đã dịch hết theo bảng `K_agent_db_00` mục 5.2 (không lộ field name, score raw, zone code, period code, money_flow_score raw vào output)?

Vi phạm câu nào sửa rồi mới render file final.

## 15. Output contract

Pack sinh ra structured content cho `O_stock_pitch_00` render. Ràng buộc:

- 15 mục đầy đủ theo thứ tự, không skip
- Số liệu định lượng quy đổi đơn vị theo K_agent_db_00 mục 6
- Ký hiệu DB raw dịch theo K_agent_db_00 mục 5.2
- Mỗi claim quan trọng có nguồn truy được: collection + field, hoặc URL web search
- Tin có dẫn link finext.vn (`https://finext.vn/news/<slug>`) hoặc URL gốc
- Cả 2 checkpoint có user phản hồi trước khi tiếp stage kế
- Mục 1 cover viết sau cùng
- Mục 14 ba kịch bản if-then không có % xác suất
- Mục 13 bear case có honest steelman 1-2 lo ngại còn yếu

Pack KHÔNG tự quyết heading style / xưng hô / tone / template pptx — `O_stock_pitch_00` quyết.
