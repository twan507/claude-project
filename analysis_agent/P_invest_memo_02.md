# P_invest_memo_02 — Tier 1: Chọn 3-5 ngành

Giai đoạn 2 của quy trình. Từ 24 ngành trong DB, chọn 3-5 ngành có đủ điều kiện để screen mã ở tier 2.

Reference: `P_invest_memo_00` phần Flow chi tiết (overview), `P_invest_memo_00` phần Cơ chế checkpoint review, `P_invest_memo_01` (tier 0 output làm input), pack `K_agent_db` (`K_agent_db_01` schema, `K_agent_db_02` query patterns, `K_agent_db_04` interpretation).

---

## 1. Mục tiêu & output expected

**Mục tiêu:** sàng lọc 24 ngành VN xuống 3-5 ngành, theo kiến trúc 2 tầng:
- **Universe filter:** loại thẳng ngành không đạt fundamental + catalyst
- **Ranking filter:** xếp hạng top 3-5 trong universe dùng flow + kỹ thuật

**Input:**

1. File output tier 0 (`tier0_YYYYMMDD_confirmed.md`) — regime + bảng catalyst + quota + flags bối cảnh
2. Quota ngành × mã từ regime:
   - Risk-on full: 5 ngành
   - Risk-on selective: 3 ngành
   - Defensive only: 2 ngành (chỉ phòng thủ)
   - Đứng ngoài: dừng, không chạy tier này

**Output chính:**

1. Shortlist 3-5 ngành với lý do và thứ tự ưu tiên
2. Flags bối cảnh PTKT cho mỗi ngành (thông tin chuyển sang tier 2)
3. Ngành sát nút (cho user review + override)
4. Report checkpoint 2 theo khung 6 phần (`P_invest_memo_00` phần Cơ chế checkpoint review)

**Tần suất:** hàng tuần sau khi tier 0 được confirm. Tier 1 không chạy độc lập — luôn follow sau tier 0.

**Thời gian session:** ước 20-40 phút Agent work. User review checkpoint 15-30 phút.

---

## 2. Kiến trúc 2 tầng filter

### Tầng 1 — Universe filter (loại thẳng)

Ngành phải lọt **cả Funnel B và Funnel C** mới vào universe. Đây là 2 tầng fundamental và catalyst — không liên quan flow/kỹ thuật.

- **Funnel B — Fundamental ngành:** ngành có trạng thái kinh doanh cơ bản ủng hộ đầu tư
- **Funnel C — Catalyst ngành:** ngành có catalyst active từ bảng tier 0

Exception: **Catalyst override** — tối đa 1 ngành/chu kỳ lọt shortlist dù fail Funnel B, nếu catalyst cực mạnh và xu hướng ngành bật từ đáy.

### Tầng 2 — Ranking filter (xếp hạng, không loại)

Sau khi có universe, dùng **Funnel A (Flow + kỹ thuật)** để xếp hạng, chọn top 3-5 mạnh nhất.

Ngành không lọt Funnel A vẫn ở trong universe — chỉ xếp hạng thấp. Nếu universe nhỏ (< 3 ngành), các ngành "yếu về flow" vẫn có thể vào shortlist.

### Tầng 3 — Thông tin bối cảnh (không filter, không ranking)

Trend và technical zone tuần+tháng của ngành chỉ dùng làm **context** chuyển sang tier 2:
- Ngành có trend tuần rơi mạnh → cảnh báo tier 2 cân nhắc size nhỏ hơn
- Ngành có technical zone AAA → ngành đang trong xu hướng mạnh, tier 2 có thể vào mạnh hơn

---

## 3. Funnel B — Fundamental ngành

### Query DB

```
collection: industry_finstats
filter: {}
projection: { "_id": 0, "industry_name": 1, "type": 1,
              "valuation_ratios": 1,
              "financial_statements.quarterly": { "$slice": -8 },
              "financial_statements.yearly": { "$slice": -3 } }
```

**Lấy:**
- 8 kỳ quarterly gần nhất (đủ để tính YoY + QoQ + đánh giá trend biên 3-4 quý)
- 3 kỳ yearly gần nhất (đủ để đánh giá P/E so với median 3 năm)
- Toàn bộ valuation_ratios hiện tại

**Lưu ý đơn vị:**
- BCTC absolute values: VND (chia 10⁹ → tỷ đồng khi trình bày)
- Vốn hoá: đã là tỷ đồng
- Tỷ lệ (ROE, ROA, biên, growth): thập phân (nhân 100 → %)

### Tiêu chí đạt Funnel B

Ngành phải thoả ít nhất **3/4 tiêu chí**:

**B1. Tăng trưởng doanh thu ngành YoY**
- Đạt: Revenue Growth YoY quý gần nhất ≥ 0 (cải thiện hoặc ổn định)
- Không đạt: âm trong 2 quý liên tiếp

**B2. Biên lợi nhuận ngành**
- Đạt: Gross Margin hoặc EBIT Margin ổn định/mở rộng qua 4 quý (dao động ± 2 điểm % không xấu đi rõ)
- Không đạt: co hẹp ≥ 3 điểm % qua 4 quý

**B3. Định giá ngành không quá đắt**
- Đạt: P/E ngành không vượt 150% median 3 năm của chính ngành đó
- Không đạt: P/E vượt 150% median 3Y (ngành đã vào vùng đắt lịch sử)
- **Lưu ý benchmark intentional:** ở tier 1 (ngành) dùng **historical median 3 năm của chính ngành** (so sánh ngành đang đắt hơn quá khứ chính nó — overvaluation toàn ngành). Ở tier 2 (mã, `P_invest_memo_03`) dùng **cross-sectional median ngành hiện tại** (so sánh mã đắt hơn peer). Hai mục đích khác nhau, intentional khác benchmark.

**B4. Sức khoẻ tài chính ngành**
- Đạt: ICR (Interest Coverage Ratio — chỉ số khả năng trả lãi vay, EBIT / chi phí lãi vay) ngành ≥ 3 (SXKD) hoặc NPL (Non-Performing Loans — tỷ lệ nợ xấu) ngành ≤ 2.5% (ngân hàng) hoặc tương đương cho các type khác
- Không đạt: ICR < 2 hoặc NPL > 3%

### Xử lý theo type ngành

**Ngành type SXKD (21/24 ngành):** áp đầy đủ B1-B4 như trên.

**Ngành type NGANHANG (1 ngành "Tài chính ngân hàng"):**
- B1: dùng NII Growth YoY (NII = Net Interest Income, thu nhập lãi thuần) + Loan Growth YoY thay vì Revenue Growth
- B2: dùng NIM (Net Interest Margin, biên lãi ròng = NII / tài sản sinh lãi) trend thay vì Gross Margin
- B3: dùng P/B so với median 3Y (bank không dùng P/E làm chính)
- B4: NPL + LLCR (Loan Loss Coverage Ratio, tỷ lệ bao phủ nợ xấu = dự phòng rủi ro tín dụng / dư nợ xấu) (xem ngưỡng ở `K_agent_db_04` D2)

**Ngành type CHUNGKHOAN (1 ngành):**
- B1: Revenue Growth YoY (biến động rất lớn theo thị trường, cần đọc theo chu kỳ)
- B2: gross margin thường 60%+, check trend
- B3: P/B hoặc P/E so với median 3Y
- B4: Margin Loans / Equity không vượt 200% (exposure rủi ro)

**Ngành type BAOHIEM (1 ngành):**
- B1: Revenue Growth (phí bảo hiểm) ≥ 0
- B2: Gross Margin có thể âm bình thường, check trend
- B3: P/B so với median 3Y
- B4: Financial Leverage trong range 7-11x (đặc thù ngành)

### Tính P/E median 3 năm

**Bước 1 — Query yearly ngành (3 năm gần nhất):**
```
// Đã có trong query trên, lấy "financial_statements.yearly": { "$slice": -3 }
```

**Bước 2 — Tính median:**
- Lấy P/E (hoặc P/B với bank) của 3 năm
- Median = giá trị giữa
- Nếu chỉ có 2 năm, dùng trung bình 2 năm

**Bước 3 — So với P/E hiện tại:**
- Hiện tại / median 3Y > 1.5 → fail B3
- Hiện tại / median 3Y ≤ 1.5 → pass B3

**Lưu ý:** với ngành mới có dữ liệu (< 3 năm), không áp B3 — chuyển sang đánh giá định tính hoặc mark "không đủ data".

### Workaround khi data không đủ

Nếu ngành có `NaN` ở một số trường quan trọng (thường là ngành nhỏ, ít mã):

- Data thiếu > 50% chỉ số B1-B4 → loại ngành khỏi universe (không đủ visibility)
- Data thiếu 20-50% → dùng các chỉ số còn lại, downgrade yêu cầu từ 3/4 xuống 2/3
- Data thiếu < 20% → tiếp tục bình thường, note rõ field nào thiếu

---

## 4. Funnel C — Catalyst ngành

### Input: bảng catalyst active từ tier 0

Agent đọc trực tiếp bảng catalyst trong file output tier 0 (`tier0_YYYYMMDD_confirmed.md`). Không query lại news — bảng catalyst đã được user confirm ở checkpoint 1, tin tưởng input này.

### Mapping catalyst → ngành hưởng lợi

Mỗi catalyst trong bảng có field "Ngành hưởng lợi" đã được điền ở tier 0. Agent gom nhóm theo ngành:

```
Ngành X:
  - Catalyst 1 (điểm Y)
  - Catalyst 2 (điểm Z)
  Tổng điểm ngành: Y + Z (tối đa 2 catalyst mạnh nhất, bỏ qua catalyst điểm 1 khi đã có ≥ 2 catalyst ≥ 2 điểm)
```

**Quy tắc gộp điểm:** max 6 điểm/ngành (2 catalyst × 3 điểm). Ngành có 3 catalyst đều 3 điểm vẫn chỉ tính 6. Nguyên tắc: catalyst thứ 3 trở đi không tăng marginal value.

### Tiêu chí đạt Funnel C

- **Đạt:** tổng điểm catalyst ngành ≥ 3 (có ít nhất 1 catalyst 3 điểm, hoặc 2 catalyst đều 2 điểm, hoặc tương đương)
- **Không đạt:** < 3 (chỉ có catalyst 1-2 điểm đơn lẻ, không đủ lực đẩy)

### Trường hợp ngành không có catalyst

Nhiều ngành sẽ không có catalyst trong bảng tier 0 (chỉ 10 catalyst max cho 24 ngành). Những ngành này:

- Fail Funnel C mặc định
- Chỉ vào universe được qua Funnel B pass + **Catalyst override** (xem dưới)
- Nếu không đủ cả B và C → loại hoàn toàn khỏi universe

### Catalyst override — Exception

Tối đa **1 ngành/chu kỳ** lọt shortlist dù **fail Funnel B**, nếu đồng thời:

1. Tổng điểm catalyst ≥ 6 (có ít nhất 1 catalyst 3 điểm và 1 catalyst 2 điểm hỗ trợ, hoặc 2 catalyst 3 điểm)
2. Xu hướng ngành 20 phiên bật từ đáy — xem phần 5 dưới, pattern "đang bật từ đáy" trong trend tuần

Lý do cơ chế này: catalyst cực mạnh có thể đảo chiều ngành yếu. Nhưng phải có confirmation sớm từ dòng tiền/xu hướng trước khi vào, tránh bắt dao rơi thuần định tính.

**Agent không tự suy luận override** nếu tổng điểm catalyst < 6 — confirmation bias risk cao. User vẫn có thể override với audit log nêu lý do cụ thể (insider info, theo dõi catalyst lâu, thông tin mới chưa vào DB). Agent ghi cảnh báo cường độ rủi ro confirmation bias kèm audit log.

---

## 5. Funnel A — Ranking (Flow + Kỹ thuật)

### Query DB

```
collection: industry_snapshot
filter: {}
projection: { "_id": 0, "industry_name": 1, "snapshot_date": 1,
              "money_flow_score": 1,
              "trend": 1, "technical_zone.overall": 1,
              "change": 1, "breadth": 1 }
```

```
collection: industry_recent
filter: {}
projection: { "_id": 0, "industry_name": 1,
              "series": { "$slice": 20 } }
```

2 query song song cho toàn bộ 24 ngành. Agent tự lọc ngành đã lọt universe (B ∩ C) sau khi có data.

### Tiêu chí xếp hạng

Xếp hạng ngành trong universe theo tổng hợp 4 tiêu chí (không ngưỡng cứng, đây là ranking):

**R1. Xếp hạng dòng tiền ngành (tự tổng hợp theo `week_score`):**

> DB không lưu `industry_rank` tĩnh. Agent query `industry_snapshot.money_flow_score.week_score` cho 18 ngành whitelist (default) hoặc danh sách user yêu cầu (override), sort giảm dần, gán rank 1..N. Xem `K_agent_db_01` mục "Xếp hạng ngành".

Ngưỡng (cho scope 18 whitelist):
- Rank 1-4: rất mạnh
- Rank 5-9: mạnh
- Rank 10-13: trung bình
- Rank 14-18: yếu

(Khi scope khác 18, scale ngưỡng theo tỷ lệ tương ứng.)

**R2. Điểm dòng tiền tuần (week_score):**
- > 20: tốt
- 10-20: OK
- 0-10: yếu
- < 0: âm

**R3. Xu hướng quý (q_trend):**
- 0.5-0.7: lý tưởng
- 0.3-0.5 hoặc 0.7-0.8: OK
- < 0.3 hoặc > 0.8: cực đoan, cân nhắc

**R4. Xu hướng năm (y_trend):**
- 0.4-0.7: lý tưởng
- > 0.8: structural quá mua (cảnh báo)
- < 0.3: structural yếu

### Công thức ranking

Không cố định công thức cứng — Agent đánh giá định tính kết hợp 4 tiêu chí. Nguyên tắc:
- Ngành có ≥ 3/4 tiêu chí rất mạnh → rank đầu
- Ngành có 2-3 tiêu chí OK trở lên → rank giữa  
- Ngành chỉ đạt 1-2 tiêu chí → rank cuối
- Priority cho trend quý (R3) khi có tranh chấp, vì đây là khung phù hợp horizon 1-6 tháng

### Chọn top N theo quota

Từ universe đã rank, chọn top N theo quota regime:
- Risk-on full: N = 5
- Risk-on selective: N = 3
- Defensive only: N = 2 (chỉ ngành phòng thủ — phải filter thêm trước khi rank)

### Ngành phòng thủ (cho Defensive only)

Regime Defensive only chỉ chấp nhận ngành:
- Tiêu dùng thiết yếu (Thực phẩm Đồ uống, Đồ gia dụng cá nhân)
- Điện nước (Điện nước & Xăng dầu khí đốt)
- Y tế (Y tế)
- Công nghệ viễn thông (có tính phòng thủ do doanh thu ổn định)

Các ngành này có doanh thu ít biến động theo chu kỳ, phù hợp khi regime không thuận lợi. Agent filter danh sách ngành phòng thủ trước khi áp universe + ranking filter.

---

## 6. Vận động 20 phiên — flag bối cảnh

Không dùng để filter hay rank, chỉ ghi chú chuyển sang tier 2 làm context khi screen mã.

Từ `industry_recent.series[]` (20 phiên), phân loại 1 trong 5 pattern trend tuần/tháng (theo `K_agent_db_04` B1.5):

1. Đang rơi từ vùng quá mua — trend đã cao > 0.75 trước đây, giảm dần 3-5 phiên
2. Đang bật từ đáy — trend ở thấp < 0.25 trước đây, tăng dần
3. Dao động biên độ lớn — rally lên > 0.75 rồi sập < 0.25, lặp 2+ lần
4. Ổn định trong dải hẹp — ranging
5. Tăng/giảm đều — xu hướng có hướng qua 10-15 phiên

**Xử lý flag cho mỗi ngành lọt shortlist:**

| Pattern | Flag cho tier 2 |
|---|---|
| Đang rơi từ đỉnh | Cảnh báo — tier 2 nên thận trọng, ưu tiên bucket 2 (pullback) hoặc bucket 3 (watchlist) |
| Đang bật từ đáy | Cơ hội sớm — tier 2 có thể vào nhưng size nhỏ, chờ confirm dòng tiền |
| Dao động lớn | Cảnh báo mạnh — tier 2 chỉ screen mã rất chất lượng, size giảm 30-50% |
| Ổn định | Trung tính — screen bình thường |
| Tăng đều | Tích cực — môi trường tốt, screen bình thường |

**Flag y_trend > 0.8:** dù ngành không bị loại vì tiêu chí này, cảnh báo cho tier 2 — ngành ở vùng quá mua dài hạn, rủi ro đảo chiều structural tăng cao. Tier 2 nhận flag này → handler ở `P_invest_memo_03` Section 7 (bucket logic): default downgrade Bucket 1 → Bucket 2 cho mã trong ngành flag. Không auto-cascade, agent flag rõ trong CP3 để user xác nhận downgrade.

---

## 7. Workflow đầy đủ — 6 bước

**Bước 1 — Load input tier 0**

Đọc file `tier0_YYYYMMDD_confirmed.md`. Extract:
- Regime + quota (N ngành)
- Bảng catalyst active
- Flags bối cảnh từ tier 0

Nếu regime = "Đứng ngoài" → dừng tier này, báo lại user.

**Bước 2 — Query DB**

Chạy 3 query song song:
- `industry_finstats` cho 24 ngành (Funnel B)
- `industry_snapshot` cho 24 ngành (Funnel A + context)
- `industry_recent` cho 24 ngành (pattern 20 phiên)

Không query `industry_info` ở tier này (giữ cho tier 2 khi cần full_ticker_list).

**Bước 3 — Áp Funnel B (Fundamental)**

Với mỗi ngành, tính:
- 4 tiêu chí B1-B4 theo type ngành
- Đạt ≥ 3/4 → pass B
- Lưu list ngành pass B

**Bước 4 — Áp Funnel C (Catalyst)**

Với mỗi ngành, từ bảng catalyst tier 0:
- Gom nhóm catalyst theo ngành
- Tính tổng điểm (max 6)
- ≥ 3 → pass C
- Lưu list ngành pass C

**Bước 5 — Xác định universe + ranking**

- Universe = ngành pass (B ∩ C)
- Xem xét catalyst override (tối đa 1 ngành, tổng catalyst ≥ 6 + bật từ đáy)
- Filter ngành phòng thủ nếu regime = Defensive only
- Áp Funnel A ranking — xếp hạng universe theo R1-R4
- Chọn top N theo quota

**Bước 6 — Xuất checkpoint 2**

Theo template Section 8.

---

## 8. Template báo cáo Checkpoint 2

Theo khung 6 phần `P_invest_memo_00` phần Cơ chế checkpoint review. Template dưới đây là cấu trúc bắt buộc.

```
# Checkpoint 2 — Chọn ngành [ngày]

## 1. Summary quyết định
Chọn [N] ngành: [danh sách].
Universe ban đầu: [X] ngành pass (B ∩ C). Top [N] theo ranking.
Catalyst override (nếu có): [tên ngành + lý do].

## 2. Bối cảnh đầu vào
- Regime từ tier 0: [regime + ngày confirm tier 0]
- Quota: [N ngành × Y mã]
- Số catalyst active: [số trong bảng tier 0]
- Ngày snapshot DB: [snapshot_date]
- Cảnh báo độ tươi: [nếu BCTC quý gần nhất > 2 tháng]

## 3. Quyết định + lý do (cho từng ngành trong shortlist)

### Ngành [Tên] — Rank [1-N]
- Funnel B (Fundamental): [số tiêu chí đạt /4, liệt kê]
  - B1 Growth YoY: [giá trị]
  - B2 Biên: [đánh giá]
  - B3 P/E vs median 3Y: [ratio]
  - B4 ICR/NPL: [giá trị]
- Funnel C (Catalyst): [tổng điểm, liệt kê catalyst]
- Funnel A (Ranking):
  - R1 rank ngành (tự tổng hợp theo `week_score`, scope 18 whitelist): [số]
  - R2 week_score: [số]
  - R3 trend quý: [%]
  - R4 trend năm: [%]
- Pattern 20 phiên: [1 trong 5 pattern]
- Flag chuyển sang tier 2: [nếu có]

(Lặp cho mỗi ngành shortlist)

## 4. Số liệu kỹ thuật key (tổng hợp 3-5 số quan trọng)

Bảng so sánh nhanh shortlist:
| Ngành | Rank | Week_score | Trend Q | Trend Y | Catalyst điểm |
|---|---|---|---|---|---|
| A | 3 | 25 | 58% | 65% | 5 |
| B | 7 | 18 | 52% | 70% | 3 |
...

## 5. Lựa chọn sát nút

### Ngành suýt lọt universe (fail 1 trong 2 funnel):
- [Ngành X]: pass B, fail C — không có catalyst. Có thể override nếu user có tin insider
- [Ngành Y]: pass C, fail B — 2/4 tiêu chí B. Thiếu biên mở rộng
- [Ngành Z]: catalyst override ứng viên nhưng xu hướng chưa bật đáy

### Ngành lọt universe nhưng rank thấp:
- [Ngành M]: pass cả B và C, rank 6/8 universe. Ưu tiên thấp vì week_score âm

## 6. Flags bối cảnh chuyển sang tier 2

Cho mỗi ngành shortlist:
- [Tên]: pattern [X], flag [cảnh báo/trung tính/tích cực], gợi ý bucket phân bổ

## 7. Câu hỏi chờ user

Xác nhận shortlist [N] ngành này để sang tier 2?
Hoặc muốn:
- (a) Thêm ngành sát nút [cụ thể] vào shortlist (ghi audit log)
- (b) Loại 1 ngành khỏi shortlist vì lý do qualitative (ví dụ regulatory risk Agent chưa biết)
- (c) Request deep-dive 1 ngành trước khi chốt
- (d) Xem lại Funnel C — Agent có gán catalyst đúng ngành không

Nếu confirm → end session, lưu output.
Nếu override → Agent ghi audit log rồi revise.
```

**Độ dài target:** 2-3 trang (~600-900 từ), bao gồm bảng.

---

## 9. Ví dụ generic

Minh hoạ cho Agent calibrate. Không phải ngành VN thật.

### Ví dụ input tier 0

- Regime: Risk-on selective
- Quota: 3 ngành × 3 mã
- Catalyst active:
  - Fed cắt lãi suất dự kiến Q3 (loại 1, 3đ) — hưởng lợi: ngành A, B, C
  - Luật X hiệu lực 1/7 (loại 2, 3đ) — hưởng lợi: ngành B, D
  - Giá hàng hoá Y đáy chu kỳ (loại 3, 2đ) — hưởng lợi: ngành E
  - NHNN nới tín dụng SX (loại 1, 2đ) — hưởng lợi: ngành F, G

### Funnel B — 24 ngành

Giả định pass B: 12 ngành (B1-B4 đạt ≥ 3/4). Fail B: 12 ngành (growth âm, biên co hẹp, hoặc định giá vượt 150% median 3Y).

### Funnel C — Mapping catalyst

- Ngành A: catalyst 1 (3đ) → tổng 3đ — pass
- Ngành B: catalyst 1 (3đ) + catalyst 2 (3đ) → tổng 6đ — pass (max)
- Ngành C: catalyst 1 (3đ) → tổng 3đ — pass
- Ngành D: catalyst 2 (3đ) → tổng 3đ — pass
- Ngành E: catalyst 3 (2đ) → tổng 2đ — fail (< 3)
- Ngành F: catalyst 4 (2đ) → tổng 2đ — fail
- Ngành G: catalyst 4 (2đ) → tổng 2đ — fail
- 17 ngành còn lại: không có catalyst → fail C

Ngành pass C: 4 (A, B, C, D).

### Universe = B ∩ C

- Ngành A: pass B + pass C → universe ✓
- Ngành B: pass B + pass C → universe ✓
- Ngành C: fail B + pass C → không vào universe mặc định. Check catalyst override: tổng catalyst 3đ (< 6), không đủ override. Loại.
- Ngành D: pass B + pass C → universe ✓

Universe: 3 ngành (A, B, D).

### Funnel A — Ranking trong universe

Giả định:
- Ngành A: rank 3, week_score 22, q_trend 55%, y_trend 62% → rank 1
- Ngành B: rank 8, week_score 15, q_trend 48%, y_trend 70% → rank 2
- Ngành D: rank 12, week_score 5, q_trend 35%, y_trend 50% → rank 3

Quota = 3 ngành, lấy cả 3. Shortlist = A, B, D.

### Pattern 20 phiên

- A: "tăng đều qua 12 phiên" → flag tích cực, tier 2 screen bình thường
- B: "đang bật từ đáy" (3 phiên) → flag cảnh báo nhẹ, tier 2 nên ưu tiên bucket 2
- D: "ổn định trong dải hẹp" → flag trung tính, screen bình thường

### Ngành sát nút (để user review)

- Ngành C: fail B nhưng có catalyst 3đ. Nếu user có tin confirm catalyst mạnh hơn thực tế → có thể override thành ngành thứ 4.
- Ngành E: pass B có thể. Fail C vì catalyst chỉ 2đ. Nếu user thấy giá hàng hoá Y đã đảo chiều rõ hơn thị trường đang phản ánh → cân nhắc.

---

## 10. Failure mode

### 10.1. Catalyst mapping không chính xác

Ngành X trong bảng tier 0 được list là "hưởng lợi" từ catalyst Y, nhưng thực ra cơ chế tác động yếu hoặc gián tiếp. Ví dụ: Fed cắt lãi suất hưởng lợi CK, nhưng tier 0 mapping cả ngành "du lịch" vào → Funnel C pass oan.

**Xử lý:** Agent cần đọc lại cơ chế tác động từng catalyst trước khi cộng điểm. Nếu nghi ngờ mapping tier 0 không chính xác, flag trong checkpoint 2 Phần 5 để user review.

### 10.2. Universe rỗng

Khi regime khắt khe (Risk-on selective hoặc Defensive only) và bảng catalyst ít, có thể universe = 0 ngành. Trường hợp này không ép ra shortlist — báo rõ "tier 1 không có ngành đủ điều kiện trong chu kỳ này".

**Xử lý:** 
- Giảm quota xuống còn 1-2 ngành thay vì ép 3
- Hoặc chờ tuần sau chạy lại khi bảng catalyst tier 0 có thêm
- Không nới Funnel B vì áp lực quota — điều đó làm mất vai trò của universe filter

### 10.3. Ranking dựa quá nhiều vào 1 tiêu chí

Agent có xu hướng sort theo week_score và chọn top N, bỏ qua trend quý/năm. Ngành có week_score cao nhưng trend quý 25% là ngành đang rally ngắn hạn trong downtrend trung hạn — rủi ro cao.

**Xử lý:** priority cho R3 (trend quý) khi có tranh chấp với R1 (rank tự tổng hợp). Ngành rank 5 nhưng trend quý 55% có thể ưu tiên hơn ngành rank 2 nhưng trend quý 20%.

### 10.4. Default về Fundamental khắt khe

Khi nhiều ngành có growth âm hoặc biên co hẹp (thị trường downturn), Agent có xu hướng fail tất cả Funnel B để "an toàn". Lỗi này dẫn đến universe = 0.

**Xử lý:** thừa nhận B1-B4 là yêu cầu tối thiểu tương đối, không tuyệt đối. Trong downturn, pass B = "tốt tương đối so với các ngành khác", không phải "tăng trưởng mạnh tuyệt đối". Có thể nới tạm thời sang "đạt 2/4 tiêu chí" nếu toàn thị trường đang khó, nhưng phải flag rõ trong checkpoint.

### 10.5. Bỏ quên Catalyst override

Catalyst override là exception quan trọng. Agent dễ quên check, dẫn đến bỏ sót ngành có catalyst cực mạnh đang đảo đáy. Ví dụ: ngành BĐS fail B nhưng có catalyst Luật Đất đai + NHNN nới tín dụng (6đ) và pattern trend đang bật từ đáy → đây là cơ hội lớn.

**Xử lý:** sau khi tính universe (B ∩ C), luôn check list ngành fail B — có ngành nào có tổng catalyst ≥ 6 và pattern "bật từ đáy" không. Nếu có, đưa vào shortlist với ghi chú "catalyst override".

### 10.6. Mapping catalyst sang ngành quá rộng

Tier 0 có thể mapping 1 catalyst vào 3-5 ngành (ví dụ Fed cắt lãi suất → CK + BĐS + ngân hàng + tiêu dùng + vàng). Agent cộng điểm catalyst đều 3đ cho tất cả → Funnel C pass cho nhiều ngành thực ra ít hưởng lợi.

**Xử lý:** khi 1 catalyst áp cho ≥ 3 ngành, giảm điểm marginal ngành — ngành hưởng lợi trực tiếp nhất vẫn giữ 3đ, ngành hưởng lợi gián tiếp giảm xuống 2đ hoặc 1đ. Agent cần reason rõ cơ chế tác động để phân biệt.

### 10.7. Thiếu bước context flags

Agent tập trung vào shortlist mà bỏ qua pattern 20 phiên và flag bối cảnh cho tier 2. Thiếu thông tin này làm tier 2 screen mã trong vacuum, không biết điều chỉnh bucket.

**Xử lý:** bắt buộc checkpoint Phần 6 có flag cho mỗi ngành trong shortlist. Nếu không có pattern rõ, ghi "không đủ data 20 phiên, tier 2 screen với criteria chuẩn".

---

## 11. Đầu ra chuẩn để tier 2 dùng

Output tier 1 lưu thành file markdown, tên `tier1_YYYYMMDD_confirmed.md` (sau khi user confirm), gồm:

1. **Header:** ngày chạy, regime kế thừa từ tier 0, quota đã áp
2. **Shortlist ngành + ranking:** bảng với thứ tự ưu tiên
3. **Chi tiết từng ngành shortlist:**
   - Điểm Funnel B, C, A đã tính
   - Catalyst map với điểm
   - Pattern 20 phiên
   - Flag chuyển sang tier 2
4. **Bảng catalyst ngành đã lọc** — chỉ catalyst liên quan các ngành trong shortlist, để tier 2 dùng làm input cho vòng C (catalyst mã)
5. **Danh sách ngành sát nút** — để tier 2 tham khảo nếu shortlist quá ít
6. **Link audit log** nếu có override

File này được enable vào project knowledge session tier 2 kế tiếp. Tier 2 dùng làm input chính, không query lại Funnel B/C/A cho các ngành không trong shortlist.
