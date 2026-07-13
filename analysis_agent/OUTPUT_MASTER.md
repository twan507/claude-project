# OUTPUT_MASTER — Glossary EN → VN cho deliverable cuối

File chốt cách dịch term tiếng Anh khi render output cuối cho user đọc. Áp tuyệt đối cho mọi deliverable (memo, weekly overview, stock report, strategy), bất kể O pack nào active.

Đọc đầu session 1 lần, mỗi session re-queryable khi compose deliverable.

## 1. Scope

**Áp khi:** compose content vào MD final (và binary derive từ MD) — file user sẽ đọc.

**Không áp khi:**
- File K pack (data schema, methodology nội bộ) — giữ EN cho consistency với code/query
- File P pack mô tả pipeline mechanic (Stage, checkpoint, workflow nội bộ) — giữ EN
- Code identifier, field name DB, YAML key, backtick `like_this`, code block
- Heading template đã thành quy ước section name của O pack (xem mục 4) — giữ EN cho consistency với spec
- Finance abbreviation (xem mục 5) — giữ EN

## 2. Glossary — 3 nhóm

### 2.1. Nhóm A — Dịch luôn, không kèm ngoặc EN

Mọi occurrence trong narrative dùng VN. (Polysemy → xem mục 3.)

| EN | VN |
|---|---|
| Ticker | mã CK |
| Stock | cổ phiếu |
| Portfolio | danh mục |
| Position (vị thế trading) | vị thế |
| Watchlist | danh mục theo dõi |
| Entry (vào lệnh) | điểm vào |
| Exit (thoát lệnh) | điểm thoát |
| Screening | sàng lọc |
| Long (vị thế trading) | mua / vị thế mua |
| Short (vị thế trading) | bán khống |
| Trend (prose) | xu hướng |
| Signal (prose) | tín hiệu |
| Volatility | biến động |
| Recovery | phục hồi |
| Consolidation | tích lũy |
| Zone (prose) | vùng |
| Take-profit (prose) | chốt lời |
| Stop-loss (prose) | cắt lỗ |
| Time-stop (prose) | chốt theo thời gian |
| Tighten stop | siết stop |
| Add / Trim (position) | thêm / cắt giảm vị thế |
| Rebalance | cân bằng lại danh mục |
| Allocation | phân bổ |
| Sizing | định cỡ vị thế |
| Margin of safety | biên an toàn |
| Cross-check | kiểm chứng chéo |
| Coverage (analyst cover mã) | bao phủ |
| Hard trigger | tín hiệu cứng |
| Soft trigger | tín hiệu mềm |
| Bull case (prose) | kịch bản tăng |
| Base case (prose) | kịch bản cơ sở |
| Bear case (prose) | kịch bản giảm |
| ADV | GTGD trung bình |

### 2.2. Nhóm B — Dịch + ngoặc EN lần đầu trong mỗi file

Pattern: `VN (EN)` ở lần đầu xuất hiện trong file output, các lần sau dùng VN-only.

| EN | VN | Ví dụ lần đầu | Lần sau |
|---|---|---|---|
| Thesis | luận điểm đầu tư | luận điểm đầu tư (Thesis) | luận điểm đầu tư |
| Conviction | độ tin cậy | độ tin cậy (Conviction) | độ tin cậy |
| Bucket | nhóm | nhóm (Bucket) | nhóm |
| **Bucket entry** (cụm) | nhóm ưu tiên | nhóm ưu tiên (Bucket entry) | nhóm ưu tiên |
| Regime | trạng thái thị trường | trạng thái thị trường (Regime) | trạng thái thị trường |
| Horizon | khung thời gian | khung thời gian (Horizon) | khung thời gian |
| Variant perception | nhận định khác biệt | nhận định khác biệt (Variant perception) | nhận định khác biệt |
| Disconfirming signal | tín hiệu phản chứng | tín hiệu phản chứng (Disconfirming signal) | tín hiệu phản chứng |
| Benchmark | mức tham chiếu | mức tham chiếu (Benchmark) | mức tham chiếu |
| Breadth | độ rộng thị trường | độ rộng thị trường (Breadth) | độ rộng thị trường |
| Pullback | nhịp điều chỉnh | nhịp điều chỉnh (Pullback) | nhịp điều chỉnh |
| Materialize / Materialized | hiện thực hoá | hiện thực hoá (Materialize) | hiện thực hoá |
| Drawdown | mức sụt giảm đỉnh-đáy | mức sụt giảm đỉnh-đáy (Drawdown) | mức sụt giảm |
| Forensic flag | dấu hiệu bất thường BCTC | dấu hiệu bất thường BCTC (Forensic flag) | dấu hiệu BCTC |
| Red flag | dấu hiệu cảnh báo | dấu hiệu cảnh báo (Red flag) | dấu hiệu cảnh báo |

**Quan trọng — Compound term (cụm 2+ từ):**

Cụm như `Bucket entry`, `Conviction tier`, `Hard trigger`, `Soft trigger`, `Forensic flag`, `Red flag`, `Variant perception`, `Disconfirming signal` — treat **nguyên cụm như 1 đơn vị**. Lookup glossary theo cụm đầy đủ, KHÔNG tách từng từ ra dịch riêng.

❌ Sai: `Bucket entry: 2` → "Nhóm (Bucket) entry: 2" (tách "Bucket" ra dịch, "entry" để nguyên — vô nghĩa)
✅ Đúng: `Bucket entry: 2` → "nhóm ưu tiên (Bucket entry): 2" lần đầu → "nhóm ưu tiên: 2" lần sau

Lưu ý ngữ nghĩa: "Bucket entry" số 1/2/3 là **nhóm ưu tiên vào lệnh** (1 = ưu tiên cao nhất, vào ngay; 2 = chờ nhịp điều chỉnh; 3 = danh mục theo dõi). Dịch "nhóm ưu tiên" khớp ngữ với số đi sau — "nhóm ưu tiên: 1" hiểu được ngay.

Quy tắc match: greedy longest match — agent ưu tiên match cụm dài nhất trong glossary trước khi fallback về từng từ riêng. Nếu cụm không có trong glossary mà các từ riêng có → giữ cả cụm EN, KHÔNG dịch lẻ tẻ.

### 2.3. Nhóm C — Giữ nguyên EN

Không dịch. Lý do: đã quen với giới đầu tư VN hoặc dịch sẽ làm mất nghĩa kỹ thuật.

| EN | Ghi chú |
|---|---|
| Memo | tên loại deliverable, quá quen |
| Catalyst | term buy-side chuẩn |
| Exhaustion | term PTKT chuẩn |
| Momentum | trader VN nói "momentum" hơn "đà" |
| Rally | phổ thông, gọn |
| Bounce | gọn, quen tai |
| Steelmanned (bear case) | term buy-side chuẩn, dịch awkward |
| Framework | đã thành quy ước học thuật |
| TP1 / TP2 / SL | viết tắt label trong table/callout |
| Hold / Shift | status keyword chuẩn hoá ở P_vbse_strategy weekly (Materialize đã có ở Nhóm B với dịch "hiện thực hoá" cho narrative; ở weekly status badge giữ EN) |
| intact / partial / deteriorating / fail | status keyword Thesis verification ở quarterly review |
| HIGH / MID / LOW | conviction tier label trong table |
| Buy / Pass / Watch / Avoid | recommendation label nội bộ (audience KH → dịch, xem mục 6) |

## 3. Polysemy & compound — exception khi áp glossary

### 3.1. Compound term — không tách

Cụm 2+ từ phải lookup nguyên cụm (greedy longest match). Cụm có trong glossary → dùng dịch nguyên cụm. Cụm KHÔNG có trong glossary → giữ EN cả cụm, KHÔNG dịch từng từ.

| Cụm | Xử lý |
|---|---|
| Bucket entry | Nhóm B — dịch "nhóm ưu tiên (Bucket entry)" |
| Conviction tier | Nhóm B — dịch "độ tin cậy (Conviction)" (drop "tier") |
| Variant perception | Nhóm B (đã có) |
| Disconfirming signal | Nhóm B (đã có) |
| Forensic flag | Nhóm B (đã có) |
| Red flag | Nhóm B (đã có) |
| Hard trigger / Soft trigger | Nhóm A — dịch "tín hiệu cứng / mềm" |
| Time-stop / Take-profit / Stop-loss | Nhóm A (đã có) |
| Cycle Top / Value Play / Value Trap / Growth at Premium | Nhóm B — dịch theo audience-aware mapping (xem mục 6) |
| Position sizing | Nhóm A — dịch "định cỡ vị thế" (dùng dịch của Sizing, KHÔNG lặp "vị thế" thêm lần) |
| Cross-check | Nhóm A (đã có) |
| Bear case steelmanned | heading section spec — giữ EN (mục 4) |

### 3.2. Polysemy — giữ EN khi không phải nghĩa trading/finance

Các term ở Nhóm A có thể có nghĩa khác trong context khác. Khi nghi ngờ → giữ EN.

| Term | Dịch khi | Giữ EN khi |
|---|---|---|
| Long / Short | vị thế trading ("Long position") | tính từ thời gian ("long-term", "short-term") |
| Position | vị thế trading ("size position") | competitive position, value chain position (dịch là "vị thế cạnh tranh / vị thế trong chuỗi giá trị" — KHÔNG dịch chữ "position" gốc) |
| Entry / Exit | vào/thoát lệnh trading | entry point pipeline, exit code workflow (giữ EN) |
| Trend | prose nói xu hướng giá/ngành | field name DB (`w_trend`, `series[].trend`) |
| Signal | prose nói tín hiệu thị trường | placeholder template `[Signal X]`, field name |
| Zone | prose nói vùng giá | field name (`technical_zone.overall`), label `Zone A/B/C` |
| Phase | giai đoạn (prose chung) | Phase 1/2/3 portfolio construction (giữ EN, code-like) |
| Tier | tầng/cấp (prose chung) | Tier 5A/5B/5C state file reference (giữ EN, code-like) |
| Target | giá mục tiêu, tỷ trọng mục tiêu | target system/field name |
| Cover | analyst bao phủ mã | covered call options (giữ EN — term PTKT khác) |
| Bull / Bear case | kịch bản tăng/giảm (prose) | `## Bull case` / `## Bear case` heading template (giữ EN, xem mục 4) |

## 4. Heading section spec — giữ EN

Heading template đã chuẩn hoá trong O pack giữ EN cho consistency với spec. KHÔNG dịch heading dù underlying term thuộc Nhóm A/B.

List heading chuẩn hoá:

```
## Recommendation
## Thesis core
## Variant Perception
## Catalysts
## Disconfirming signals
## Bear case / Bear case steelmanned
## Bull case
## Base case
## Exit triggers
## Take-profit triggers
## Stop-loss triggers
## Forensic flags
## Business Overview
## Monitoring + Exit Triggers
## Thesis verification
## Valuation update
## Rebalance proposal
## Portfolio Construction
```

**Khi term trong heading xuất hiện trong narrative cùng file:** áp glossary Nhóm A/B/C bình thường. Vd heading `## 4. Thesis verification` giữ EN, nhưng trong prose dưới heading "thesis fail" → tuỳ context (status keyword giữ EN per Nhóm C; prose narrative dịch).

## 5. Finance abbreviation — giữ EN

Viết tắt ratio/metric tài chính + market indicator + central bank: **giữ EN toàn bộ**, không gloss, không dịch.

**Ratio & metric:**
`P/E`, `P/B`, `EV/EBITDA`, `ROE`, `ROA`, `ROIC`, `NIM`, `NPL`, `CASA`, `LDR`, `LCR`, `CAR`, `WACC`, `DCF`, `FCF`, `EBIT`, `EBITDA`, `EPS`, `DSO`, `LLR`, `APE`, `NBV`

(Lưu ý: `ADV` đã chuyển sang Nhóm A — dịch hẳn "GTGD trung bình" (Giá trị giao dịch trung bình), không giữ viết tắt EN vì KH không quen.)

**Time/period:**
`TTM`, `YoY`, `QoQ`, `MoM`, `YTD`, `CAGR`

**Market/macro:**
`FII`, `DXY`, `VND`, `USD`, `bps` (basis point)

**Central bank/institution:**
`FOMC`, `Fed`, `ECB`, `PBOC`, `SBV`, `OPEC`

**Doc reference VN:**
`BCTC`, `BCTN`, `AGM`, `EGM`

## 6. Audience-aware override — O pack quản

Khi audience cuối là KH (client-facing), một số term có dịch MỀM HƠN. Mapping default dưới đây áp khi O pack không có K hygiene table riêng. O pack có table riêng (vd `O_stock_report_00` mục 5) → table đó override mapping này trong scope O pack.

| Term nội bộ (analyst) | Audience KH |
|---|---|
| Long (recommendation) | Quan điểm tích cực |
| Watch (recommendation) | Tiếp tục theo dõi |
| Avoid (recommendation) | Cẩn trọng / Chưa khuyến nghị tham gia |
| Conviction HIGH | Quan điểm tích cực mạnh |
| Conviction MID | Quan điểm tích cực trung bình |
| Conviction LOW | Quan điểm thận trọng |
| Variant Perception | Góc nhìn khác với quan điểm chung thị trường (hoặc ẩn) |
| Disconfirming signal | Tín hiệu cần theo dõi để xem xét lại |
| Forensic flag | Vấn đề cần lưu ý từ BCTC |
| Value Play / Value Trap / Growth at Premium / Cycle Top | Định giá hấp dẫn có cơ sở / Định giá hấp dẫn nhưng có rủi ro / Tăng trưởng cao có cơ sở / Định giá đỉnh chu kỳ |
| TP1 / TP2 / SL (giá số cụ thể) | KHÔNG render — chỉ "Tín hiệu xem xét lại quan điểm" |
| Bucket 1/2/3 entry | KHÔNG render — ẩn bucket reference |
| Bear case (và Bear case steelmanned) | Kịch bản tiêu cực (wording mềm hơn, không "steelmanned") |

**Rule:**
- O pack có K hygiene table riêng cho audience KH → **áp table riêng đó** (override OUTPUT_MASTER trong scope O pack).
- O pack KHÔNG có table riêng + audience KH → **fall back bảng trên** làm default.
- Audience nội bộ analyst (default mọi O pack) → **không áp** audience-aware mapping; dùng thẳng glossary Nhóm A/B/C.
- Khi không rõ audience cuối → hỏi user clarify trước khi render (không tự đoán).

## 7. Conflict resolution

Thứ tự ưu tiên khi nhiều rule áp cùng 1 term:

1. **O pack K hygiene table riêng** (nếu có) — audience-aware override OUTPUT_MASTER trong scope O pack đó
2. **Heading section spec** (mục 4) — giữ EN dù term thuộc Nhóm A/B
3. **Compound term — longest match** (mục 3.1) — lookup nguyên cụm trước, KHÔNG tách từng từ ra dịch riêng
4. **Polysemy exception** (mục 3.2) — giữ EN khi context không phải trading/finance
5. **Finance abbreviation** (mục 5) — giữ EN
6. **Glossary Nhóm A/B/C** (mục 2) — apply rule tương ứng
7. **Term/cụm không match nhóm nào** — giữ EN, không tự đoán dịch

## 8. Maintenance

Thêm/sửa term: edit trực tiếp file này. Không duplicate glossary vào pack khác.

Term mới khi có polysemy phải ghi rõ scope (trading-only / cross-context) ở mục 3.

Nếu O pack phát sinh term audience-aware mới → ghi vào K hygiene table của O pack đó, không phải vào OUTPUT_MASTER (file này chỉ list pattern phổ biến).
