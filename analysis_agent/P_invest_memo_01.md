# P_invest_memo_01 — Tier 0: Gate vĩ mô + Catalyst scan

Giai đoạn 1 của quy trình. Trả lời 2 câu hỏi: (a) regime thị trường hiện tại cho phép vào mới không, (b) bảng catalyst active trong 0-3 tháng tới cho các tier kế tiếp dùng.

Reference: `P_invest_memo_00` phần Flow chi tiết (overview), `P_invest_memo_00` phần Cơ chế checkpoint review, pack `K_agent_db` (`K_agent_db_01` schema, `K_agent_db_02` query patterns, `K_agent_db_04` interpretation, `K_agent_db_05` news methodology).

---

## 1. Mục tiêu & output expected

**Mục tiêu:** quyết định 1 trong 4 regime thị trường và lập bảng catalyst active để các tier kế tiếp dùng làm input.

**4 regime:**

| Regime | Khi nào | Quota ngành × mã | Cash buffer đề xuất |
|---|---|---|---|
| Risk-on full | Thị trường khoẻ đồng pha, vĩ mô ủng hộ, catalyst dồi dào | 5 ngành × 3 mã | 0-10% |
| Risk-on selective | Thị trường hồi phục sớm hoặc phân hoá, tiêu chí khắt khe hơn | 3 ngành × 3 mã | 20-30% |
| Defensive only | Quá mua đồng pha hoặc vĩ mô siết nhẹ, chỉ phòng thủ | 2 ngành × 3 mã (phòng thủ) | 50-70% |
| Đứng ngoài | Đang rơi từ đỉnh, vĩ mô shock, hoặc catalyst đảo chiều tiêu cực | Dừng quy trình | 100% |

**Output chính:**

1. File markdown với report checkpoint 1 (theo khung 6 phần `P_invest_memo_00` phần Cơ chế checkpoint review)
2. Bảng catalyst active — format chuẩn, dùng làm input cho tier 1 Funnel C
3. Quota ngành × mã + cash buffer đề xuất

**Tần suất:** hàng tuần hoặc khi có macro shock (Fed meeting, tin chính sách lớn, sự kiện địa chính trị).

**Độ tin cậy ước lượng:** regime đúng 70-80% trong điều kiện thị trường bình thường; trong giai đoạn chuyển pha (từ uptrend sang downtrend hoặc ngược lại), regime có thể cần điều chỉnh 1-2 lần trước khi ổn định — đây là bản chất của gating system, không phải lỗi methodology.

---

## 2. Bốn input bắt buộc

Mỗi input có query DB cụ thể + xử lý. Không được skip input nào.

### Input 1 — Market trend đa khung + vận động 20 phiên

**Mục đích:** xác định vị trí thị trường trong chu kỳ trend và hướng vận động 20 phiên gần nhất.

**Lưu ý giới hạn ứng dụng:** với horizon đầu tư 1-6 tháng, trend không phải công cụ dự báo dài hạn. Trend chỉ giúp xác định:
- Có phải thời điểm thuận lợi để vào mới không (entry timing cấp thị trường)
- Mức cash buffer phù hợp tại thời điểm hiện tại
- Cảnh báo rủi ro ngắn hạn để điều chỉnh quota

Trend **không dùng** để dự báo thị trường sẽ đi đâu sau 6 tháng — đó là việc của fundamental và catalyst loại 1-2. Vì vậy trọng tâm của tier 0 **không phải** phân tích trend chi tiết, mà là ra quyết định regime nhanh và đúng để tier 1-4 (chọn ngành, chọn mã) có input làm việc.

**2 query bắt buộc (theo nguyên tắc `K_agent_db_04` B1.5 — snapshot một mình không đủ):**

Query 1.1 — Snapshot hiện tại:
```
collection: market_snapshot
filter: {}
projection: { "_id": 0, "snapshot_date": 1, "price": 1, "breadth": 1,
              "change": 1, "trend": 1, "technical_zone.overall": 1 }
```

Query 1.2 — Vận động 20 phiên:
```
collection: market_recent
filter: {}
projection: { "_id": 0, "index": 1,
              "series.date": 1, "series.price": 1, "series.trend": 1 }
```

**Lưu ý schema (v2):**
- `market_recent` là 1 doc duy nhất, khoá `index: "VNINDEX"`, MỘT array `series` sort mới → cũ (~20 phiên)
- Mỗi item `series[]` có cả `price` lẫn `trend` (4 khung w/m/q/y) — cấu trúc thống nhất với industry/group_recent
- KHÔNG có `money_flow_score` (khác industry_recent/group_recent)

**Xử lý 4 giá trị trend tuần/tháng/quý/năm — đọc ở mức cơ bản:**

Mỗi khung đọc theo ngưỡng mean-reversion (từ `K_agent_db_04` B2):

| Giá trị | Trạng thái |
|---|---|
| > 0.8 | Quá mua cực đoan |
| 0.6 — 0.8 | Bullish mạnh |
| 0.4 — 0.6 | Cân bằng |
| 0.2 — 0.4 | Bearish mạnh |
| < 0.2 | Quá bán cực đoan |

**Phân loại pattern vận động từ 20 phiên** (`K_agent_db_04` B1.5, 5 pattern) — ở mức nhận diện cơ bản, không cần deep-dive:

1. Đang rơi từ vùng quá mua — trend ở cao > 0.75 trước đây, giảm dần 3-5 phiên
2. Đang bật từ đáy — trend ở thấp < 0.25 trước đây, tăng dần
3. Dao động biên độ lớn — rally lên > 0.75 rồi sập < 0.25, lặp 2+ lần trong 20 phiên
4. Ổn định trong dải hẹp (0.15-0.20 rộng quanh 1 mức) — ranging
5. Tăng/giảm đều — trend di chuyển có hướng qua 10-15 phiên, volatility thấp

**Output Input 1:** câu tóm tắt trạng thái 4 khung + 1 pattern 20 phiên. Đừng cố rút ra kết luận dài hạn từ trend.

### Input 2 — Vĩ mô quốc tế

**Mục đích:** xác định chi phí vốn toàn cầu và rủi ro địa chính trị ảnh hưởng dòng tiền vào VN.

**Query chính:**
```
collection: other_data
filter: { "name": { "$in": ["S&P 500", "Nasdaq", "Nikkei 225", "Shanghai Composite",
  "Dow Jones", "TPCP Mỹ 10 năm", "TPCP Mỹ 5 năm",
  "EUR/USD", "USD/JPY", "Bitcoin (BTC)"] } }
projection: { "_id": 0, "name": 1, "value": 1, "unit": 1,
              "pct_change": 1, "w_pct": 1, "m_pct": 1, "update_date": 1 }
```

**Web search bổ sung bắt buộc:**

- Fed/FOMC (Federal Open Market Committee — Ủy ban thị trường mở Mỹ, ra quyết định lãi suất Fed): lịch họp tới, dot plot (biểu đồ điểm thể hiện kỳ vọng lãi suất Fed của từng thành viên FOMC các năm tới) gần nhất, ngôn ngữ FOMC statement
- Sự kiện địa chính trị 7 ngày: chiến tranh, sanctions, bầu cử lớn
- DXY (US Dollar Index — chỉ số đo sức mạnh USD so với rổ 6 đồng tiền lớn): không có trong DB, search "DXY today"
- VIX (Volatility Index — đo volatility kỳ vọng của S&P 500, còn gọi là "chỉ số sợ hãi"): không có trong DB, search "VIX current"

**Xử lý:**

Đánh giá 4 factor vĩ mô quốc tế:

| Factor | Bullish | Bearish |
|---|---|---|
| Lãi suất toàn cầu | Fed dovish, TPCP 10Y giảm | Fed hawkish, TPCP 10Y tăng mạnh |
| Tâm lý risk-on | S&P 500 tăng, VIX < 18, crypto tăng | S&P giảm > 2% tuần, VIX > 25 |
| USD | DXY giảm (tốt cho EM — emerging markets / thị trường mới nổi) | DXY tăng mạnh, flight to safety |
| Địa chính trị | Ổn định | Conflict leo thang, sanctions mới |

**Output Input 2:** 1 câu về môi trường vĩ mô quốc tế + rating (ủng hộ / trung tính / bất lợi).

### Input 3 — Vĩ mô nội địa

**Mục đích:** xác định chính sách tiền tệ, tài khóa, tỷ giá của VN.

**Query DB:**
```
collection: other_data
filter: { "group": "macro" }
projection: { "_id": 0, "name": 1, "category": 1, "value": 1, "unit": 1,
              "pct_change": 1, "m_pct": 1, "update_date": 1 }
```

Kéo 23 chỉ số vĩ mô VN: lãi suất tái cấp vốn, chiết khấu, liên ngân hàng các kỳ hạn, huy động, cho vay, USD NHTM bán, CPI, PMI, IIP, XNK...

**Query tin vĩ mô 14 ngày gần nhất:**
```
collection: news_history_feed
filter: { 
  "news_type": "trong_nuoc", 
  "type": "news_feed",
  "category_name": { "$in": [
    "Kinh tế", "Thị trường", "Chính sách mới", "Ngân hàng",
    "Chỉ đạo, quyết định của Chính phủ - Thủ tướng Chính phủ",
    "Pháp luật", "Tham vấn chính sách"
  ]},
  "created_at": { "$gte": ISODate("<ngày cách đây 14 ngày>") } 
}
sort: { "created_at": -1 }
limit: 30
projection: { "_id": 0, "title": 1, "sapo": 1, "article_slug": 1,
              "category_name": 1, "created_at": 1 }
```

Duyệt title + sapo, chọn 5-10 tin quan trọng (chính sách NHNN, Chính phủ, Bộ Tài chính, số liệu vĩ mô mới). Với tin quan trọng, query `news_history_content` để đọc chi tiết.

**Xử lý — đánh giá 4 factor vĩ mô nội địa:**

| Factor | Ủng hộ | Bất lợi |
|---|---|---|
| Lãi suất VN | LS liên NH giảm hoặc ổn định, LS huy động giảm | LS liên NH tăng đột biến, NHNN hút ròng mạnh |
| Tỷ giá | USD NHTM ổn định hoặc giảm | USD tăng > 1% tháng, áp lực tỷ giá lớn |
| CPI | < 4%, đi ngang hoặc giảm | > 4.5% và tăng tháng liên tiếp |
| Chính sách | Mở rộng tín dụng, đẩy đầu tư công, không siết BĐS/CK | Siết tín dụng, tăng thuế, siết phát hành TPDN |

**Tin chính sách bắt buộc check (override):** một số tin có impact trực tiếp lên regime:

- NHNN hạ lãi suất điều hành → ủng hộ risk-on
- NHNN phát hành tín phiếu quy mô > 50k tỷ → cảnh báo siết
- Quốc hội thông qua luật lớn (Đất đai, Bất động sản, Các TCTD) → catalyst ngành
- Thông tư mới siết/nới điều kiện cho vay → impact BĐS, ngân hàng

### Input 4 — Dòng tiền khối ngoại toàn thị trường

**Mục đích:** NN là proxy cho tâm lý nhà đầu tư tổ chức quốc tế. Đây là **thông tin tham khảo**, không phải yếu tố quyết định regime.

**Nguyên tắc đọc NN đúng:**

- NN mua ròng → tín hiệu tích cực, nhưng không phải mã/ngành nào NN cũng mua (có mã NN gần như không giao dịch)
- NN bán ròng mạnh → cảnh báo cần lưu ý, nhưng không tự động dẫn tới defensive
- NN trung tính → không có tín hiệu, bỏ qua

NN chỉ là 1 yếu tố nhỏ trong đánh giá regime. Không dùng ngưỡng cứng để upgrade/downgrade regime. Trọng số NN < trọng số của Input 1 (market trend) và Input 3 (vĩ mô nội địa).

**Query:**
```
collection: market_nntd
filter: {}
projection: { "_id": 0 }
```

**Xử lý — đánh giá định tính, không ngưỡng cứng:**

Đọc 3 khung NN (`nn.latest`, `nn.week`, `nn.month`) và gán 1 trong 3 rating:

- **Tích cực** — mua ròng rõ ràng trong 1 tháng, xu hướng ổn định hoặc tăng
- **Trung tính** — dao động quanh 0, hoặc bán ròng nhẹ đi kèm xu hướng giảm bán
- **Tiêu cực** — bán ròng mạnh và xu hướng tăng bán

Ngưỡng "mạnh" / "nhẹ" đọc trong bối cảnh thị trường hiện tại — tham chiếu thanh khoản trung bình của thị trường tuần trước để calibrate. Ví dụ: bán ròng 5,000 tỷ/tháng là tiêu cực ở thị trường trung bình nhưng có thể chỉ trung tính ở thị trường sôi động thanh khoản cao.

**Check pattern chuyển đổi:** tương tự các input khác, pattern quan trọng hơn giá trị tuyệt đối. NN đã bán ròng 3 tháng rồi chuyển mua ròng → tín hiệu đảo chiều tích cực sớm. NN mua ròng dài rồi chuyển bán đột ngột → cảnh báo sớm. Các pattern này có thể influence regime nhưng vẫn chỉ là yếu tố bổ sung.

**Tự doanh (TD):** đọc `td.month.net_value` chỉ để tham khảo. TD giao dịch ngắn hạn, không phản ánh smart money, trọng số rất thấp trong quyết định regime.

---

## 3. Bảng quyết định gate — 4 regime

Logic quyết định regime dựa trên tổng hợp 4 input. Agent không áp cứng bảng này như công thức — đây là hướng dẫn, mỗi trường hợp phải đọc cross-reference 4 input để ra kết luận cuối.

### Regime 1 — Risk-on full

**Điều kiện cần (đủ ≥ 3/4):**

- **Input 1 (market trend):** ít nhất 3/4 khung trong vùng 0.4-0.7 (bullish mạnh hoặc cân bằng, không quá mua); pattern vận động 20 phiên là "tăng đều qua nhiều phiên" hoặc "đang bật từ đáy"
- **Input 2 (vĩ mô quốc tế):** ủng hộ hoặc trung tính — không có event shock trong 7 ngày
- **Input 3 (vĩ mô nội địa):** ủng hộ — lãi suất ổn định/giảm, CPI < 4%, không tin siết chặt mới
- **Input 4 (dòng tiền NN):** tích cực hoặc trung tính — không có pattern bán ròng mạnh kéo dài

**Tín hiệu confirm bổ sung:** có ít nhất 2 catalyst loại 1 hoặc 2 (vĩ mô quyết định hoặc chính sách ngành) đang active.

**Quota:** 5 ngành × 3 mã, cash buffer 0-10%.

### Regime 2 — Risk-on selective

**Điều kiện cần:**

- **Input 1:** 2/4 khung ủng hộ, pattern "đang bật từ đáy" chưa confirm đủ 10 phiên, HOẶC 1-2 khung đã > 0.75 (chớm quá mua) trong khi các khung khác OK
- **Input 2 hoặc 3:** có ít nhất 1 factor cảnh báo nhẹ (ví dụ DXY đang tăng, VIX 20-25, CPI tăng nhẹ nhưng chưa quá 4.5%)
- **Input 4:** NN rating trung tính hoặc tiêu cực nhẹ với xu hướng đang giảm bán

**Quota:** 3 ngành × 3 mã, tiêu chí tier 2-3 khắt khe hơn (yêu cầu cao hơn ở Funnel B + C). Cash buffer 20-30%.

### Regime 3 — Defensive only

**Điều kiện cần (đủ ≥ 2/4 cảnh báo):**

- **Input 1:** 3/4 khung > 0.75 (quá mua đồng pha), pattern "đang rơi từ vùng quá mua", HOẶC 2/4 khung < 0.2 (quá bán đồng pha chưa confirm đảo chiều)
- **Input 2:** vĩ mô quốc tế bất lợi rõ — S&P giảm > 3% tuần, VIX > 25, Fed hawkish ngoài dự kiến
- **Input 3:** vĩ mô nội địa bất lợi — NHNN hút ròng lớn, lãi suất LNH tăng đột biến, CPI > 4.5%
- **Input 4:** NN bán ròng rõ rệt và kéo dài (đọc định tính theo bối cảnh thị trường)

**Quota:** 2 ngành × 3 mã, chỉ ngành phòng thủ (tiêu dùng thiết yếu, điện nước, y tế). Cash buffer 50-70%.

### Regime 4 — Đứng ngoài

**Điều kiện cần (đủ ≥ 1 critical):**

- **Input 1:** pattern "dao động biên độ lớn" hoặc "đang rơi từ đỉnh" mới bắt đầu (< 5 phiên, chưa xác nhận đáy)
- **Input 2:** event shock lớn đang diễn ra — chiến tranh leo thang, khủng hoảng ngân hàng quốc tế, Fed emergency action
- **Input 3:** chính sách siết chặt đột ngột — NHNN tăng lãi suất, thắt chặt tín dụng mạnh, cấm phát hành TPDN
- **Input 4:** NN bán ròng rất mạnh và không có dấu hiệu chững (đọc định tính theo bối cảnh)

**Quota:** dừng toàn bộ quy trình. Cash buffer 100%. Không vào position mới, giữ hoặc trim position hiện tại.

### Cash buffer + kế hoạch điều chỉnh

Cash buffer không phải số cố định, mà có kế hoạch điều chỉnh theo kịch bản có thể xảy ra trong 1-4 tuần tới. Mục đích: user biết trước nên tăng hay giảm buffer nếu market chuyển regime.

**Nguyên tắc:** khi regime hiện tại gần ranh giới giữa 2 regime (ví dụ Risk-on full sắp thành Selective, hoặc Selective sắp thành Defensive), dùng mức cash buffer ở đầu trên của range để chủ động.

| Regime hiện tại | Cash buffer base | Kịch bản cần điều chỉnh |
|---|---|---|
| Risk-on full | 0-10% | Nếu xu hướng tuần vượt 0.75 và VIX > 22 → nâng lên 10-20%. Nếu có tin Fed hawkish ngoài dự kiến → nâng lên 15-25% |
| Risk-on selective | 20-30% | Nếu xu hướng tuần tiếp tục lên quá 0.80 → nâng lên 30-40% chuẩn bị defensive. Nếu dòng tiền NN bật sang mua ròng ổn định → có thể giảm về 15-20% |
| Defensive only | 50-70% | Nếu xu hướng tuần đảo chiều xuống dưới 0.50 mà volume confirm → có thể giảm về 40-50% và chuyển sang selective. Nếu tiếp tục rơi → giữ 70% và sẵn sàng lên 90-100% |
| Đứng ngoài | 100% | Giữ nguyên cho đến khi xu hướng quý xác nhận đáy (bật lên từ < 0.2 và giữ > 0.3 trong 10 phiên) → giảm về 60-70% chuyển defensive |

**Viết kế hoạch cụ thể trong checkpoint:** Agent không chỉ báo mức buffer hiện tại, mà ghi rõ 2-3 kịch bản có thể xảy ra trong 1-4 tuần tới và action tương ứng. User có kế hoạch trước nên không bị động khi thị trường chuyển regime.

**Lưu ý với horizon 1-6 tháng:** cash buffer chủ yếu dựa trên trend tuần + tháng (ngắn-trung hạn), không dựa trên trend năm. Trend năm chỉ là thông tin cảnh báo structural.

---

## 4. Catalyst scan

Bảng catalyst active là output quan trọng thứ 2 của tier 0 (sau regime). Các tier kế dùng bảng này làm input cho Funnel C.

### Định nghĩa catalyst hợp lệ — 3 đặc tính

Không phải tin nào cũng là catalyst. Tin phải đồng thời có:

1. **Timing cụ thể** — ngày hoặc khung thời gian ước lượng được (ĐHCĐ ngày X, KQKD Q tới, họp FOMC tháng Y, hiệu lực luật Z)
2. **Magnitude đo được** — ước lượng được tác động tới doanh thu/LNST/định giá (ít nhất dạng khoảng: tăng 5-10%, giảm 2%, không phải "sẽ tích cực")
3. **Verifiable** — biết được sau sự kiện là đúng hay sai, không để "chờ dài hạn" mơ hồ

Tin thiếu 1 trong 3 → không tính là catalyst. Treat như context để đánh giá vĩ mô nội địa (Input 3), không vào bảng catalyst.

### Phân loại 4 loại catalyst

| Loại | Ví dụ | Timeline | Độ chắc | Impact scope |
|---|---|---|---|---|
| 1. Vĩ mô quyết định | NHNN hạ lãi suất, Fed xoay trục, nới tín dụng hệ thống | 0-3 tháng | Cao — văn bản chính thức | Toàn thị trường hoặc nhóm lớn |
| 2. Chính sách ngành | Luật Đất đai hiệu lực, đầu tư công đợt mới, thuế carbon | 3-12 tháng | Trung — lộ trình dần | Ngành cụ thể |
| 3. Chu kỳ hàng hoá | Giá dầu/thép/urea đảo chiều cấu trúc, USD đỉnh chu kỳ | 6-24 tháng | Thấp-Trung — phụ thuộc global | Ngành nhạy hàng hoá |
| 4. Sự kiện doanh nghiệp | KQKD Q tới, M&A, thoái vốn NN, tăng vốn, niêm yết sàn mới | 0-6 tháng | Biến động — tin đồn vs công bố | Mã cụ thể |

**Scope tier 0:** chỉ quét catalyst loại 1, 2, 3 (cấp ngành trở lên). Catalyst loại 4 (doanh nghiệp) quét ở tier 2 trong từng ngành đã chọn.

### Quy trình quét catalyst

**Bước 1 — DB scan:**
```
collection: news_history_feed
filter: { 
  "$or": [
    { "news_type": "quoc_te" },
    { "news_type": "trong_nuoc",
      "category_name": { "$in": [
        "Kinh tế", "Thị trường", "Chính sách mới", "Ngân hàng",
        "Chỉ đạo, quyết định của Chính phủ - Thủ tướng Chính phủ",
        "Pháp luật", "Tham vấn chính sách"
      ]}
    }
  ],
  "type": "news_feed",
  "created_at": { "$gte": ISODate("<cách đây 30 ngày>") } 
}
sort: { "created_at": -1 }
projection: { "_id": 0, "title": 1, "sapo": 1, "article_slug": 1,
              "category_name": 1, "created_at": 1, "news_type": 1 }
```

Duyệt title + sapo. Với mỗi tin tiềm năng catalyst, query content để đọc chi tiết:
```
collection: news_history_content
filter: { "article_slug": "<slug>" }
projection: { "_id": 0, "plain_content": 1 }
```

**Bước 2 — Web search bổ sung:**

- Lịch họp NHNN (thường tháng 1 lần)
- Lịch FOMC 8 lần/năm
- Lịch công bố số liệu vĩ mô VN (GDP, CPI, PMI — mỗi tháng)
- Luật lớn sắp hiệu lực (Luật Đất đai, Luật Các TCTD, Luật Kinh doanh BĐS)
- Chu kỳ hàng hoá — giá dầu/thép/quặng đảo chiều cấu trúc nếu có (phân biệt với biến động ngắn hạn)

**Bước 3 — Chấm điểm từng catalyst (1-3 điểm):**

| Điểm | Tiêu chí |
|---|---|
| 3 | Catalyst đã xác nhận, magnitude lớn, timing trong 0-3 tháng, chưa priced-in |
| 2 | Catalyst có khả năng cao, timing 3-6 tháng, priced-in một phần |
| 1 | Catalyst possible nhưng chưa rõ, timing > 6 tháng, hoặc magnitude nhỏ |

**Constraint:** tối đa 10 catalyst active trong bảng một thời điểm để tránh "catalyst inflation". Nếu liệt được > 10, siết tiêu chí — giảm catalyst điểm 1 trước.

### Bảng catalyst — format output

```
Catalyst | Loại | Timing | Điểm | Ngành hưởng lợi | Ngành bất lợi
Fed dự kiến cắt lãi suất Q3 | 1 | 0-2 tháng | 3 | CK, BĐS, vàng | Bảo hiểm nhân thọ
Luật Đất đai hiệu lực 1/8 | 2 | 3 tháng | 2 | BĐS dân dụng, xây dựng | —
Giá urea đáy chu kỳ 2 năm | 3 | 6-12 tháng | 2 | Phân bón | —
NHNN nới tín dụng ngành SX | 1 | 1 tháng | 3 | Thép, hoá chất, dệt may | —
```

**Bảng này được lưu vào file và đưa vào project knowledge** để tier 1-4 reference. Update hàng tuần khi chạy tier 0.

---

## 5. Template báo cáo Checkpoint 1

Theo khung 6 phần của `P_invest_memo_00` phần Cơ chế checkpoint review. Template dưới đây là cấu trúc bắt buộc Agent dùng khi kết thúc tier 0.

```
# Checkpoint 1 — Gate vĩ mô [ngày]

## 1. Summary quyết định
Regime: [1 trong 4]. Quota: [X ngành × Y mã]. Cash buffer: [%].
Bảng catalyst active: [N catalyst, chi tiết ở Phần 3].

## 2. Bối cảnh đầu vào
- Ngày snapshot: [ngày market_snapshot]
- Ngày vĩ mô nội địa gần nhất: [update_date mới nhất, cảnh báo nếu > 2 tuần]
- Ngày tin vĩ mô quan trọng đã đọc: [ngày gần nhất]
- Web search cut-off: [ngày]
- Cảnh báo độ tươi nếu có

## 3. Quyết định + lý do
### Regime [X] vì:
- Input 1 (market trend): [giá trị 4 khung + pattern 20 phiên + đánh giá]
- Input 2 (vĩ mô quốc tế): [rating + 1-2 điểm chính]
- Input 3 (vĩ mô nội địa): [rating + 1-2 điểm chính]
- Input 4 (dòng tiền NN): [xu hướng 1 tháng + rating]
- Tổng hợp: [câu giải thích tại sao kết hợp 4 input ra regime này]

## 4. Số liệu kỹ thuật key
- Xu hướng tuần/tháng/quý/năm hiện tại: [4 số %]
- NN net value 1 tháng: [số tỷ đồng]
- Lãi suất LNH qua đêm gần nhất: [%]
- USD NHTM bán: [giá trị VND]
- S&P 500 1 tuần: [%]

## 5. Lựa chọn sát nút
- Regime đang cân nhắc ngoài [X]: [Y] — thiếu tiêu chí nào, sao không chọn
- Nếu user có quan điểm khác về market trend hoặc có tin nội bộ về vĩ mô, có thể override

## 6. Bảng catalyst active
[Bảng format ở Section 4]

## 7. Câu hỏi chờ user
Xác nhận regime [X] và quota [Y ngành × Z mã]?
Hoặc muốn:
- (a) Override regime thành [khác] với lý do
- (b) Bổ sung catalyst Agent miss
- (c) Yêu cầu deep-dive 1 input cụ thể trước khi chốt

Nếu confirm → end session, output file này làm input cho tier 1.
Nếu override → Agent ghi audit log rồi revise.
```

**Độ dài target:** 1.5-2 trang (400-600 từ core, chưa tính bảng catalyst).

---

## 6. Ví dụ generic cho 4 regime

Ví dụ minh hoạ chỉ để Agent calibrate. Không phải case thật.

### Ví dụ regime Risk-on full

- Input 1: xu hướng tuần 55%, tháng 62%, quý 58%, năm 65%. Pattern 20 phiên là "tăng đều" — trend các khung tăng dần qua 12 phiên, từ ~45% lên ~60%. Không khung nào quá mua.
- Input 2: S&P 500 tăng 2% tuần, VIX 16, DXY giảm 1% tuần, không event shock. Fed dovish tone.
- Input 3: CPI 3.2% (ổn định), LS LNH qua đêm 3.8% (giảm từ 4.5% tháng trước), USD NHTM ổn định. Tin mới: NHNN vừa giảm lãi suất điều hành 0.25%.
- Input 4: NN mua ròng ổn định 3 tuần qua, chuyển từ bán sang mua. Rating: tích cực.

Regime = Risk-on full. 4/4 input ủng hộ. Catalyst: Fed dovish (3đ), NHNN hạ lãi suất (3đ), số liệu PMI cải thiện (2đ).

Cash buffer base: 0-10%. Kế hoạch: nếu xu hướng tuần vượt 0.75 trong 2 tuần tới hoặc có tin Fed hawkish bất ngờ → nâng lên 10-20%.

### Ví dụ regime Risk-on selective

- Input 1: xu hướng tuần 78% (chớm quá mua), tháng 72%, quý 55%, năm 60%. Pattern 20 phiên là "tăng đều" nhưng 5 phiên gần nhất dao động trong 76-80% — chững lại sau rally.
- Input 2: S&P 500 đi ngang, DXY tăng 0.8% tuần, VIX 20.
- Input 3: CPI 3.8% (đang tăng), LS LNH ổn định, tin đồn NHNN có thể siết nhẹ tăng trưởng tín dụng.
- Input 4: NN bán ròng nhẹ nhưng xu hướng đang giảm bán. Rating: trung tính.

Regime = Risk-on selective. 2/4 input ủng hộ. Xu hướng tuần chớm quá mua. Tiêu chí tier 2-3 khắt khe hơn.

Cash buffer base: 20-30%, chọn đầu trên ~28%. Kế hoạch: nếu xu hướng tuần tiếp tục lên > 0.80 → nâng lên 30-40%. Nếu tin siết tín dụng confirmed → nâng lên 35-45%.

### Ví dụ regime Defensive only

- Input 1: xu hướng tuần 82%, tháng 85%, quý 78%, năm 72%. Pattern 20 phiên là "đang rơi từ vùng quá mua" — trend tuần đã giảm từ 88% xuống 82% qua 4 phiên, đi kèm volume rút.
- Input 2: S&P 500 giảm 2% tuần, DXY tăng 1.5%, VIX 24. Fed tone gần nhất hawkish.
- Input 3: CPI 4.3%, tăng 2 tháng liên tiếp. NHNN phát hành tín phiếu quy mô lớn tuần trước.
- Input 4: NN bán ròng kéo dài 4-5 tuần với cường độ tăng dần. Rating: tiêu cực.

Regime = Defensive only. 3/4 cảnh báo. Chỉ xem ngành phòng thủ.

Cash buffer base: 50-70%, chọn 60%. Kế hoạch: nếu xu hướng tuần đảo xuống dưới 0.50 và volume xác nhận → giảm về 40-50% chuyển selective. Nếu tiếp tục rơi thủng 0.30 → nâng lên 80-90% chuẩn bị đứng ngoài.

### Ví dụ regime Đứng ngoài

- Input 1: xu hướng tuần 15%, tháng 22%, quý 28%, năm 35%. Pattern "đang rơi từ đỉnh" chỉ 3 phiên trước, đã phá support quan trọng.
- Input 2: xung đột địa chính trị vừa leo thang, S&P giảm 5% trong 3 ngày, VIX 32.
- Input 3: tin NHNN tăng lãi suất điều hành 0.5% bất ngờ.
- Input 4: NN bán ròng rất mạnh và tăng tốc. Rating: tiêu cực mạnh.

Regime = Đứng ngoài. 4/4 critical. Dừng quy trình, giữ tiền mặt.

Cash buffer: 100%. Kế hoạch: giữ nguyên cho đến khi xu hướng quý bật lên từ < 0.2 và giữ > 0.3 trong 10 phiên → giảm về 60-70% chuyển defensive.

---

## 7. Failure mode

### 7.1. Agent dễ chọn regime trung dung (Risk-on selective)

Khi input mixed (vài ủng hộ vài cảnh báo), Agent có xu hướng default về Risk-on selective để "an toàn". Lỗi này dẫn đến bỏ sót cơ hội ở Risk-on full hoặc underestimate rủi ro ở Defensive only.

**Xử lý:** ép Agent đọc từng input riêng biệt trước khi tổng hợp. Nếu 3/4 input rõ ràng ủng hộ nhưng 1 input trung tính → vẫn Risk-on full. Nếu 3/4 input rõ ràng cảnh báo nhưng 1 input trung tính → vẫn Defensive only.

### 7.2. Bỏ sót pattern vận động 20 phiên

Agent query `market_recent` nhưng chỉ đọc giá trị hiện tại, không phân loại pattern. Dẫn đến đánh giá snapshot 65% (cân bằng) mà không biết pattern là "đang rơi từ 85%" — rủi ro rất khác.

**Xử lý:** bắt buộc Agent phân loại 1/5 pattern trong báo cáo checkpoint. Nếu không phân loại được rõ, ghi "ranging chưa xác nhận" và hạ regime 1 bậc để thận trọng.

### 7.3. Catalyst inflation

Agent gán điểm dễ dãi, list > 10 catalyst trong bảng. Làm bảng catalyst mất ý nghĩa, tier 1 Funnel C match đại trà.

**Xử lý:** constraint 10 catalyst max. Nếu > 10, loại theo thứ tự: catalyst 1 điểm → catalyst 2 điểm có timing > 6 tháng → catalyst 2 điểm độ chắc thấp. Giữ catalyst 3 điểm.

### 7.4. Tin đồn gán thành catalyst

Tin không có văn bản chính thức (chỉ 1 nguồn báo đại chúng, hoặc diễn đàn) không được tự động gán catalyst 3 điểm. Agent dễ bị fool khi nguồn tin mạnh nhưng chưa có confirmation.

**Xử lý:** quy tắc mặc định — catalyst 3 điểm bắt buộc có ít nhất 2 nguồn chính thống xác nhận (báo chính thống + văn bản cơ quan nhà nước, hoặc 2 văn bản độc lập). Tin đồn 1 nguồn → catalyst 1 điểm max.

**Cho phép override với audit:** user có thể override quy tắc này nếu có insight nội bộ mà Agent không biết (ví dụ: tin đồn đã theo dõi lâu, nguồn đáng tin cậy trong ngành). Khi override, Agent ghi vào `audit_overrides.md` với format:

```
[Ngày] - CP1 - Loại: Gán catalyst 3 điểm với 1 nguồn
Catalyst: [tên]
Nguồn user dẫn: [nguồn]
Lý do user: [nhập từ user]
```

Sau 3-6 tháng, so sánh override này với kết quả thực tế để đánh giá chất lượng insight.

### 7.5. Bỏ sót checkpoint trước khi chuyển tier

Agent tự chuyển sang tier 1 mà không chờ user confirm. Dẫn đến output tier 1 dựa trên regime mà user có thể không đồng ý.

**Xử lý:** Nguyên tắc 6 của `P_invest_memo_00` — Agent phải dừng sau khi xuất checkpoint, chờ user trả lời Phần 7 template. Không chuyển tier tự động trong bất kỳ trường hợp nào.

### 7.6. Không cập nhật độ tươi dữ liệu

Chỉ số vĩ mô tháng (CPI, PMI, XNK) trong `other_data` có thể cũ 2-3 tuần. Agent đọc giá trị mà không ghi chú thời điểm update, ra kết luận sai về state hiện tại.

**Xử lý:** luôn kiểm tra `update_date` của mỗi chỉ số quan trọng. Nếu > 2 tuần, ghi chú rõ trong checkpoint Phần 2 và hạn chế dùng làm yếu tố quyết định chính.

---

## 8. Đầu ra chuẩn để tier 1 dùng

Output tier 0 phải lưu thành file markdown, đặt tên `tier0_YYYYMMDD_confirmed.md` (sau khi user confirm), gồm:

1. **Header:** ngày chạy, regime đã chốt, quota
2. **Bảng catalyst active** (format Section 4) — dùng trực tiếp làm input Funnel C ở tier 1
3. **Flags bối cảnh cho tier 1:**
   - Ngành phòng thủ ưu tiên nếu regime = Defensive only (gợi ý top 3-5)
   - Ngành nhạy catalyst lớn — tier 1 sẽ score cao Funnel C cho ngành này
   - Cảnh báo về khung trend năm quá mua (nếu có) để tier 3 cân nhắc bucket 3 chặt hơn
4. **Link audit log** nếu có override

File này được enable vào project knowledge session tier 1 kế tiếp.
