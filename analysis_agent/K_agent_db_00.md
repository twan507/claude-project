# K_agent_db_00 — Master file

## 1. Mục đích & scope

Pack `K_agent_db` cung cấp knowledge base cho phân tích chứng khoán Việt Nam dựa trên MongoDB `agent_db`. Pack chứa schema, query patterns, anti-patterns, methodology diễn giải chỉ báo, và methodology phân tích tin tức.

**Input kỳ vọng:** query về ticker/ngành/thị trường VN, tin tức, BCTC, dòng tiền, technical, vĩ mô trong nước.

**Output kỳ vọng:** số liệu định lượng đã quy đổi đơn vị, diễn giải methodology bằng ngôn ngữ tự nhiên (không lộ ký hiệu raw), context tin tức có nguồn.

**Negative scope:**
- Không dùng cho thị trường ngoài VN (cổ phiếu US, crypto, hàng hoá quốc tế trừ khi làm bối cảnh)
- Không thay thế model DCF chuyên sâu của analyst lâu năm
- Không hỗ trợ tư vấn cho retail/client cuối — pack này giả định audience là analyst/broker nội bộ (xem mục 4.4)
- Không thực hiện ghi dữ liệu vào DB, chỉ đọc (find, aggregate)

## 2. Nguồn dữ liệu

Pack dùng 2 nguồn song song:

**MongoDB `agent_db`** là nguồn CHÍNH cho số liệu định lượng (giá, dòng tiền, BCTC, kỹ thuật, vĩ mô từ `other_data`, tin tức trong DB). Quyền read-only. Schema chi tiết ở `K_agent_db_01`, pipeline mẫu ở `K_agent_db_02`.

**Web search** BẮT BUỘC cho: tin tức hiện tại, sự kiện vĩ mô quốc tế, benchmark ngành ngoài VN, phân tích bên ngoài, xác minh thông tin không có trong DB. Không được dùng training data thay thế web search.

**Domain rule về tin tức:** khi user hỏi về tin tức, sự kiện hiện tại, bối cảnh vĩ mô, hoặc cần ngữ cảnh mới nhất, bắt buộc query DB (`news_today_feed`, `news_history_feed`, block news trong `data_briefing`, `other_data`) VÀ gọi web search song song. Không trả lời từ trí nhớ.

**Ghi nguồn khi trình bày tin:**
- Tin từ DB: "theo dữ liệu tin tức trong hệ thống ngày [DD/MM]"
- Tin từ web: "theo [tên báo / URL]" hoặc trích dẫn cụ thể

## 3. Manifest file con

Pack có 5 file con reference (số hiệu là reference index, không phải thứ tự thực thi):

**`K_agent_db_01` — Collections schema**
25 collection trong `agent_db`. Tra khi cần hiểu cấu trúc document trước khi query.

**`K_agent_db_02` — Query patterns**
12 workflow pipeline (ký hiệu A đến L). Dùng làm template, thay placeholder. Không tự sáng chế pipeline phức tạp khi đã có template phù hợp.

**`K_agent_db_03` — Anti-patterns**
Gallery các lỗi thật gặp trong quá khứ + cách sửa. Đọc khi nghi vấn, đặc biệt trước câu hỏi phân tích phức tạp lần đầu trong session.

**`K_agent_db_04` — Interpretation & methodology**
Methodology diễn giải chỉ báo (dòng tiền, trend đa khung, technical zone), phương pháp PTCB riêng cho 4 type doanh nghiệp (SXKD, NGANHANG, CHUNGKHOAN, BAOHIEM), kịch bản ticker, pitfalls. Đọc đầu session khi có câu hỏi phân tích chi tiết hoặc gặp chỉ báo chưa chắc cách đọc.

**`K_agent_db_05` — News methodology**
Methodology phân tích 4 loại tin (`doanh_nghiep`, `quoc_te`, `trong_nuoc`, `thong_cao`), framework chấm điểm impact nội bộ, case study thị trường VN, workflow đa tin, bảng dịch thuật ngữ tiếng Anh. Đọc đầu session khi câu hỏi liên quan tin tức, chính sách vĩ mô, hoặc yêu cầu bối cảnh sự kiện.

Agent đọc file con theo nhu cầu query, không bắt buộc đọc hết đầu session.

## 4. Domain rules

Các rule bổ sung domain-specific, không trùng meta-rules ở system prompt.

### 4.1. Biệt danh thị trường không chuẩn

Biệt danh và nhóm mã không chính thức (ví dụ "Tuấn Mượt", "nhóm bà Phương Thảo", "hàng anh Vượng", "hệ Masan", "hàng FLC cũ") không được tự đoán nghĩa. Phải HỎI user xác nhận hoặc web search xác minh trước khi phân tích.

Lý do: nếu đoán sai giả định gốc, toàn bộ phân tích sau dính lỗi. User có thể tin tưởng số liệu (đúng từ DB) nhưng được gắn vào luận điểm sai — đây là loại lỗi tệ nhất.

### 4.2. Clarification format cho domain này

Khi cần clarify trước câu hỏi phân tích (theo rule system prompt mục 5.4), domain này dùng 2 câu hỏi chuẩn:

> **1. Khung thời gian quan tâm:**
> (a) Ngắn hạn dưới 1 tháng
> (b) Trung hạn 3-6 tháng
> (c) Dài hạn trên 1 năm
>
> **2. Mục đích:**
> (a) Tra cứu trạng thái
> (b) Cân nhắc mở vị thế mới
> (c) Review vị thế đang cầm

Skip clarification khi: tra cứu đơn lẻ ("VNM giá bao nhiêu", "KLGD HPG hôm nay"), trạng thái nhanh có 1 cách hiểu ("thị trường hôm nay thế nào"), hoặc câu hỏi tiếp nối khi context đã rõ từ turn trước.

### 4.3. Đưa số có cơ sở

**Xác suất scenario** (ví dụ "40%/45%/15%"): CHỈ được đưa khi có cơ sở định lượng (mô hình, backtest, base rate historical). Không gán theo cảm nhận. Nếu chỉ là định tính, dùng ngôn ngữ định tính: "kịch bản cơ sở", "khả năng cao", "rủi ro đuôi".

**Phân bổ % danh mục:** được phép (user là analyst nội bộ), nhưng phải gắn với giả định rõ về khung thời gian, mức rủi ro, vốn ban đầu giả định.

**Target giá:** chỉ nói khi có mức kỹ thuật xác định (Fibonacci, pivot, volume profile POC). Đây là mô tả mức kỹ thuật, không phải dự báo điểm đến. Không dùng "target" theo nghĩa "giá sẽ về" trừ khi có model định giá độc lập.

### 4.4. Giới hạn tư vấn

Pack giả định user là analyst/broker nội bộ, được phép nhận khuyến nghị cụ thể. Nếu project chuyển cho audience khác (retail, client cuối, intern chưa chứng chỉ), cần swap sang K pack phù hợp — pack này không phục vụ được vì constraint nội dung rộng hơn mức cho phép retail.

Khuyến nghị phải:
- Gắn với giả định rõ: khung thời gian, mức rủi ro, vốn giả định
- Cân bằng lập luận ủng hộ và phản đối, không chỉ 1 chiều
- Ghi cuối: "Quyết định cuối vẫn do anh/chị cân nhắc"
- Không hứa hẹn lợi nhuận, không dùng "chắc chắn tăng/giảm", "không thể lỗ"

### 4.5. Whitelist 18 ngành phân tích — Default scope + User override

**Default mode (user không nói gì cụ thể):** DB lưu **24 ngành** nhưng pack **mặc định chỉ phân tích 18 ngành** trong whitelist. Mọi query / aggregate / ranking / báo cáo cấp ngành filter theo whitelist này; các ngành ngoài whitelist không xuất hiện trong báo cáo. Mã thuộc ngành ngoài whitelist vẫn phân tích đơn lẻ được nếu user hỏi đích danh ticker, nhưng không vào watchlist theme / sector tilts.

**Override mode (user yêu cầu cụ thể ngành ngoài whitelist):** vd "phân tích ngành Bảo hiểm", "so sánh BVH vs VNM", "BCTC ngành Y tế Dược phẩm Q1" — agent **vẫn query được và trả lời bình thường** (dữ liệu 24 ngành đầy đủ trong DB). Khi đó:
- Query thẳng theo `industry_name` user yêu cầu, không apply whitelist filter
- Ghi note "ngoài scope whitelist mặc định" trong output để user biết
- Không tự ý so sánh với các ngành whitelist trừ khi user yêu cầu rõ

Danh sách 18 mã ngắn (user nhập) ↔ tên DB chuẩn (`industry_name` / `industry`) + cách áp dụng filter (cả Default + Override mode) và xử lý re-rank: xem **`K_agent_db_01`** đầu Section B "Khối ngành".

Khi user nhập mã ngắn (vd "DAUKHI", "NGANHANG"): map sang tên chuẩn DB để query; xuất báo cáo dùng tên đầy đủ, không lộ mã ngắn.

## 5. K hygiene — ký hiệu cần dịch trước khi output

Rule K hygiene ở system prompt mục 5.5 bắt buộc dịch ký hiệu raw và taxonomy nội bộ sang ngôn ngữ tự nhiên. Pack này định nghĩa 3 nhóm cần dịch và bảng dịch tương ứng.

### 5.1. Ba nhóm ký hiệu

**Nhóm 1 — Ký hiệu DB raw:**
`vsi`, `VSI`, `day_score`, `week_score`, `zone` với giá trị `A/AA/AAA/B/C`, `f382`/`f500`/`f618`, `poc`/`val`/`vah`, `r1`/`s1`, `period: "2025_4"`, `m_pct`/`w_pct`/`q_pct`/`y_pct`, `w_trend`/`m_trend`/`q_trend`/`y_trend`, `rank_pct`, `industry_rank_pct`, `market_rank_pct`.

**Nhóm 2 — Taxonomy nội bộ methodology (từ file 04, 05):**
- Tên kịch bản trend đa khung: "Kịch bản A/B/C/D/E/F/G"
- Tên kịch bản ticker: "Kịch bản E1/E2/E3"
- Tên pitfall: "Pitfall F1" đến "F12"
- Tên section hoặc workflow: "B5", "B6", "B7", "C6", "D1-D4", "Workflow A-L", "Bước 1/2/3 của B7"
- Tên 4 kịch bản SXKD: "Value Play", "Value Trap", "Growth at Premium", "Cycle Top"
- Tên nhãn chấm điểm tin: "HIGH/MID/LOW impact", "logic gate", "framework chấm điểm", "impact score", "điểm X/Y"

Đây là công cụ nội bộ để agent dùng khi suy luận, không bao giờ lộ ra output. Thay vào đó mô tả trực tiếp tác động cụ thể và cơ chế bằng ngôn ngữ tự nhiên.

**Nhóm 3 — Thuật ngữ tiếng Anh chuyên môn chưa dịch:**
"mean-reversion", "exhaustion", "dead-cat bounce", "confluence level", "Market Profile", "DuPont decomposition", "Golden Ratio retracement", "Value Area", "bear trap", "bull trap", "whip-saw".

Thuật ngữ tiếng Anh trong phân tích tin tức (bảng dịch đầy đủ ở `K_agent_db_05` phần 9): "sell on news", "priced-in", "dot plot", "forward guidance", "hawkish/dovish", "contango/backwardation", "tailwinds", "confluence", "divergence", "smart money", "wash-out", "risk-off/risk-on", "pump and dump", "going concern", "cross-default", "one-off", "pre-sales".

Viết tắt thông dụng có thể giữ nguyên: Fed, FOMC, CPI, NFP, PCE, PMI, DXY, VIX, FDI, FII, ESOP, M&A. Giải thích ngắn khi dùng lần đầu trong session.

**Exception — Slug trong URL finext.vn:**

`article_slug` và `report_slug` thuộc Nhóm 1 (ký hiệu DB raw), cấm lộ dạng trần trong output (ví dụ không viết `article_slug: vnm-bao-cao-q1`). Tuy nhiên khi ghép vào URL đầy đủ `https://finext.vn/news/{article_slug}` hoặc `https://finext.vn/reports/{report_slug}`, đây là output user-facing hợp lệ — URL công khai, không phải ký hiệu nội bộ DB. Chi tiết pattern xem `K_agent_db_01` section F (URL pattern — Dẫn link finext.vn).

### 5.2. Bảng dịch ký hiệu DB sang ngôn ngữ tự nhiên

| DB raw | Cách nói với user |
|---|---|
| `vsi: 2.1` | thanh khoản gấp 2.1 lần trung bình 5 phiên |
| `technical_zone.overall.w: "AAA"` | vùng kỹ thuật rất mạnh khung tuần |
| `technical_zone.overall.w: "AA"` | vùng kỹ thuật mạnh khung tuần |
| `technical_zone.overall.w: "A"` | vùng kỹ thuật tích cực khung tuần |
| `technical_zone.overall.w: "B"` | vùng kỹ thuật trung tính khung tuần |
| `technical_zone.overall.w: "C"` | vùng kỹ thuật yếu khung tuần |
| `day_score: 68` | điểm dòng tiền ngày 68 |
| `week_score: -18` | dòng tiền tuần âm 18, đang rút ra |
| `breadth_in: 127, breadth_out: 171` | 127 mã tăng, 171 mã giảm, bên bán thắng thế |
| `industry_rank_pct: 0.9` | top 10% mạnh nhất ngành (percentile mã trong ngành) |
| `market_rank_pct: 0.95` | top 5% mạnh nhất thị trường (percentile mã trong thị trường) |
| Rank ngành-vs-ngành | DB không lưu — tự tổng hợp sort `week_score` (dòng tiền tuần) qua 18 ngành whitelist; xem `K_agent_db_01` mục "Xếp hạng ngành" |
| `fibonacci.w.f382: 1763` | hỗ trợ Fibonacci 38.2% khung tuần quanh 1763 |
| `volume_profile.w.poc: 1750` | vùng giá tập trung giao dịch quanh 1750 |
| `volume_profile.w.val / vah` | biên dưới / biên trên vùng giá chấp nhận |
| `nn.week.net_value: 4.2` | khối ngoại mua ròng 4.2 tỷ tuần qua |
| `ROE: 0.23` | ROE 23% |
| `period: "2025_4"` | Q4/2025 |
| `period: "2025_5"` | năm 2025 |
| `m_pct: 0.062` | tăng 6.2% trong tháng qua |
| `w_pct: 0.04` | tăng 4% trong tuần qua |
| `q_pct: -0.033` | giảm 3.3% trong quý qua |
| `y_pct: 0.46` | tăng 46% trong năm qua |
| `w_trend: 0.35` | xu hướng tuần 35% (35% số mã đang trên đường trend tuần) |
| `m_trend: 0.68` | xu hướng tháng 68% |
| `q_trend: 0.28` | xu hướng quý 28% |
| `y_trend: 0.32` | xu hướng năm 32% |

### 5.3. Bảng dịch taxonomy nội bộ sang mô tả trực tiếp

| Internal | Mô tả cho user |
|---|---|
| Kịch bản A (đồng pha trung tính tích cực) | Thị trường tăng khỏe đồng đều 4 khung, chưa cực đoan |
| Kịch bản B (ngắn yếu + dài khỏe) | Điều chỉnh ngắn hạn trong xu hướng dài vẫn mạnh, cơ hội mua pullback |
| Kịch bản C (ngắn yếu + dài cũng yếu) | Cả ngắn và dài đều yếu, tránh bắt đáy, bounce có thể chỉ là hồi kỹ thuật |
| Kịch bản D (ngắn quá mua + dài chưa) | Điều chỉnh ngắn sắp tới trong uptrend dài, chờ tuần pullback mới vào |
| Kịch bản E (đồng pha quá mua) | Cảnh báo đỉnh lớn, cả thị trường lan toả cực đoan, giảm tỷ trọng |
| Kịch bản F (đồng pha quá bán) | Cảnh báo đáy lớn, canh tích luỹ dần, không all-in vì đáy có thể kéo dài |
| Kịch bản G (sóng hồi trung hạn) | Rally trung hạn từ đáy dài hạn, chưa xác nhận dài hạn, rủi ro cao |
| Kịch bản E1 (đã tăng nhưng còn khoẻ) | Mã đã có sóng tăng rõ nhưng chưa có dấu hiệu cạn lực |
| Kịch bản E2 (chưa tăng nhưng dòng tiền quay lại) | Mã đang tích luỹ hoặc vừa đảo chiều đáy, tín hiệu sớm chưa xác nhận |
| Kịch bản E3 (rủi ro cao, tránh vào) | Mã có nhiều cảnh báo đồng thời, không nên mua |
| warning mean-reversion | Cảnh báo khả năng đảo chiều do quá mua hoặc quá bán |
| exhaustion | Rally đuối hơi, cạn lực tăng |
| dead-cat bounce | Hồi kỹ thuật trong downtrend, không bền |
| confluence level | Vùng giao nhau của nhiều mức hỗ trợ hoặc kháng cự, mạnh hơn mức đơn |
| Value Area | Vùng giá chấp nhận, nơi diễn ra khoảng 70% giao dịch |
| Value Trap | Bẫy giá trị, P/E rẻ phản ánh kỳ vọng xấu có cơ sở, không phải undervalued |
| DuPont decomposition | Tách ROE thành 3 thành phần: biên lợi nhuận × vòng quay tài sản × đòn bẩy |
| Golden Ratio retracement | Mức Fibonacci 61.8%, mức hỗ trợ sâu nhất; vượt xuống là cấu trúc trend có thể đã gãy |
| whip-saw | Dao động biên độ lớn, rally rồi sập lặp nhiều lần |
| B5, B6, B7, Workflow E, Bước 1/2/3 | Không nhắc tên section, làm theo flow tự nhiên |

## 6. Quy đổi đơn vị

- **BCTC** (`Net Revenue`, `Total Assets`, `Total Equity`, `Net Income`): giá trị là **đồng**. Chia 10^9 ra tỷ đồng. Ví dụ `9864419377152` thành 9.864 tỷ đồng.
- **Vốn hoá thị trường** trong `valuation_ratios`: đã là **tỷ đồng**, giữ nguyên.
- **Khối ngoại / Tự doanh** (`buy_value`, `sell_value`, `net_value`): đã là **tỷ đồng**.
- **Tỷ lệ** (ROE, ROA, Gross Margin, pct_change, các `*_pct`): dạng **thập phân**, nhân 100 ra %.
- **`other_data.value`**: luôn đọc `unit` kèm (USD/ounce, USD/thùng, Đồng/kg, %, Triệu USD, Tỷ VND).
- **Lãi suất** trong `other_data`: `value: 0.045, unit: "%"` bằng 4.5% (đã ở dạng thập phân, nhân 100 khi nói).

## 7. Độ tươi dữ liệu

- `snapshot_date`, `latest.date`: nếu không phải hôm nay, ghi rõ "số liệu đến phiên [DD/MM]"
- `other_data.update_date`: chỉ số vĩ mô tháng (CPI, XNK, PMI) có thể cũ 2-3 tuần, luôn ghi chú ngày cập nhật
- Tin từ DB: rolling 30 ngày, luôn đối chiếu web search để lấy tin mới hơn nếu có

## 8. Lăng kính phân tích cốt lõi

Dòng tiền là lăng kính trung tâm. DB `agent_db` được tối ưu cho phân tích dòng tiền, đây là lợi thế cạnh tranh của pack. Mọi phân tích tổng hợp phải có tối thiểu 1 luận điểm dòng tiền: điểm số ngày hoặc tuần, xếp hạng ngành, xếp hạng thị trường, khối ngoại, độ rộng, cường độ thanh khoản.

**Ba lăng kính** cho phân tích chi tiết theo thứ tự:
1. Dòng tiền (trước)
2. Kỹ thuật (MA, Fibonacci, volume profile, zone)
3. Cơ bản (định giá, BCTC, tăng trưởng)

**Lồng vĩ mô khi liên quan** (gợi ý mapping ngành và chỉ số cần theo dõi):

- Dầu khí: Dầu Brent, WTI
- Thép: quặng sắt, than cốc, HRC
- Ngân hàng: lãi suất điều hành, tỷ giá USD/VND
- BĐS: lãi suất huy động, lãi suất cho vay
- Xuất khẩu: USD/VND, EUR/USD
- Vàng: giá vàng thế giới
- Nông nghiệp: giá hàng hoá nông sản (cà phê, gạo, cao su, đường)

Khi mã hoặc ngành nhạy vĩ mô, kéo `other_data` và web search tin quốc tế song song.

## 9. Xử lý lỗi và thiếu dữ liệu

- Query rỗng thì báo "chưa có dữ liệu cho [X]", đề xuất hướng thay thế nếu có
- Field `null` hoặc `NaN` thì bỏ qua, không đoán
- Ticker không có trong `stock_info` thì báo "Mã [X] không có trong hệ thống, kiểm tra lại", không đoán mã tương tự
- `data_briefing` thiếu block nào thì tiếp tục với block còn lại, ghi chú phần thiếu
- Web search không có kết quả thì báo "không tìm được tin gần đây về [X]", không bịa

## 10. Output contract

Pack này sinh ra **structured content** để layer trên (P pack, O pack) tiêu thụ. Ràng buộc:

- Số liệu định lượng phải đã quy đổi đơn vị theo mục 6
- Ký hiệu DB raw phải đã dịch theo bảng mục 5.2
- Taxonomy nội bộ phải đã thay bằng mô tả trực tiếp theo bảng mục 5.3
- Mỗi claim có nguồn truy được: tên collection + field, hoặc URL web search
- Thông tin vĩ mô tháng hoặc tin tức có ghi chú ngày cập nhật

Pack KHÔNG tự quyết format output cuối (heading, xưng hô, length, tone). Layer O quyết nếu có O pack active, ngược lại fall back Default Kernel (xem system prompt mục 6).
