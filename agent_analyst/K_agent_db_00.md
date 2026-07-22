# K_agent_db_00 — Master file

> **⚑ v3 (2026-07-21):** DB lên **35 collection** — thêm khối lịch sử ĐỊNH GIÁ (`history_finratios_stock` / `_industry`, điểm dữ liệu theo **TUẦN**) và khối lịch sử KHỐI NGOẠI / TỰ DOANH (`history_nntd_stock` / `_index`, điểm **mỗi phiên** từ 2020). Kèm theo: methodology định giá tương đối theo lịch sử (`K_agent_db_04` mục D6) + Rule 10 & case 11 (`K_agent_db_03`); `corr60` được định nghĩa lại (đồng pha xu hướng–thanh khoản, KHÔNG phải lan toả dòng tiền); TRANSITION hạ sàn `exposure` xuống **0.50**; thêm cờ nội bộ `suppressed` (`K_agent_db_06` mục 2). Đơn vị mới: mục 6.

> **v2 (2026-07-13):** đồng bộ với pipeline fnx05 v2. Thay đổi lớn: (1) đơn vị `*_pct` chuyển sang **điểm phần trăm** đọc thẳng, `rank_pct` thang **0–100** (mục 6 + 5.2); (2) DB từ 25 lên **31 collection** — thêm khối phase & danh mục (`K_agent_db_06`, mục 4.6); (3) `data_briefing` chỉ còn 2 doc (`core` + `news_report`); (4) field thiếu dữ liệu bị **omit** khỏi doc, không còn `null`/`NaN` (mục 9); (5) query patterns 13 workflow A–M.

## 1. Mục đích & scope

Pack `K_agent_db` cung cấp knowledge base cho phân tích chứng khoán Việt Nam dựa trên MongoDB `agent_db`. Pack chứa schema, query patterns, anti-patterns, methodology diễn giải chỉ báo, methodology phân tích tin tức, và tầng phase & danh mục hệ thống.

**Input kỳ vọng:** query về ticker/ngành/thị trường VN, tin tức, BCTC, dòng tiền, technical, vĩ mô trong nước, pha thị trường, danh mục hệ thống.

**Output kỳ vọng:** số liệu định lượng đã quy đổi đơn vị, diễn giải methodology bằng ngôn ngữ tự nhiên (không lộ ký hiệu raw), context tin tức có nguồn.

**Negative scope:**
- Không dùng cho thị trường ngoài VN (cổ phiếu US, crypto, hàng hoá quốc tế trừ khi làm bối cảnh)
- Không thay thế model DCF chuyên sâu của analyst lâu năm
- Không hỗ trợ tư vấn cho retail/client cuối — pack này giả định audience là analyst/broker nội bộ (xem mục 4.4)
- Không thực hiện ghi dữ liệu vào DB, chỉ đọc (find, aggregate)
- Không đặt lệnh, không thao tác tài khoản, không có dữ liệu lệnh thật/tài khoản cá nhân

## 2. Nguồn dữ liệu

Pack dùng 2 nguồn song song:

**MongoDB `agent_db`** là nguồn CHÍNH cho số liệu định lượng (giá, dòng tiền, BCTC, kỹ thuật, phase, danh mục, vĩ mô từ `other_data`, tin tức trong DB). Quyền read-only. Cập nhật liên tục trong phiên + EOD. Schema chi tiết ở `K_agent_db_01`, pipeline mẫu ở `K_agent_db_02`.

**Web search** BẮT BUỘC cho: tin tức hiện tại, sự kiện vĩ mô quốc tế, benchmark ngành ngoài VN, phân tích bên ngoài, xác minh thông tin không có trong DB. Không được dùng training data thay thế web search.

**Domain rule về tin tức:** khi user hỏi về tin tức, sự kiện hiện tại, bối cảnh vĩ mô, hoặc cần ngữ cảnh mới nhất, bắt buộc query DB (`news_today_feed`, `news_history_feed`, doc `news_report` trong `data_briefing`, `other_data`) VÀ gọi web search song song. Không trả lời từ trí nhớ. Nếu runtime không có web search: trả lời từ DB + ghi rõ "chưa đối chiếu được tin mới ngoài hệ thống" với sự kiện đang diễn biến — tuyệt đối không lấp chỗ trống bằng training data.

**Ghi nguồn khi trình bày tin:**
- Tin từ DB: "theo dữ liệu tin tức trong hệ thống ngày [DD/MM]"
- Tin từ web: "theo [tên báo / URL]" hoặc trích dẫn cụ thể

**Luật query (bắt buộc):** chỉ đọc các collection trong schema `K_agent_db_01` — thấy tên lạ (kể cả `temp_*`/`old_*`) thì bỏ qua · luôn có projection, không `find({})` trần trên collection theo mã · `history_*`/`*_itd` bắt buộc filter khoá + `$slice` · không `$lookup`/`$out`/`$merge`/`$where` · kết quả ước quá ~50KB thì thu hẹp trước khi chạy.

## 3. Manifest file con

Pack có 6 file con reference (số hiệu là reference index, không phải thứ tự thực thi):

**`K_agent_db_01` — Collections schema**
35 collection trong `agent_db` (8 khối cũ + Section I khối phase & danh mục; khối E gồm 3 nhóm lịch sử: GIÁ mỗi phiên, ĐỊNH GIÁ mỗi tuần, KHỐI NGOẠI/TỰ DOANH mỗi phiên) + công thức chỉ báo gốc + URL pattern finext.vn. Tra khi cần hiểu cấu trúc document trước khi query.

**`K_agent_db_02` — Query patterns**
13 workflow pipeline (ký hiệu A đến M; M = phase & danh mục). Dùng làm template, thay placeholder. Không tự sáng chế pipeline phức tạp khi đã có template phù hợp.

**`K_agent_db_03` — Anti-patterns**
Gallery 10 case lỗi thật gặp trong quá khứ + cách sửa. Đọc khi nghi vấn, đặc biệt trước câu hỏi phân tích phức tạp lần đầu trong session.

**`K_agent_db_04` — Interpretation & methodology**
Methodology diễn giải chỉ báo (dòng tiền, trend đa khung, technical zone), phương pháp PTCB riêng cho 4 type doanh nghiệp (SXKD, NGANHANG, CHUNGKHOAN, BAOHIEM), **định giá tương đối theo lịch sử (mục D6 — canonical cho mọi câu hỏi đắt/rẻ)**, kịch bản ticker, pitfalls. **Đầu file có bảng dịch taxonomy nội bộ** (tham chiếu từ mục 5.3). Đọc đầu session khi có câu hỏi phân tích chi tiết hoặc gặp chỉ báo chưa chắc cách đọc.

**`K_agent_db_05` — News methodology**
Methodology phân tích 4 loại tin (`doanh_nghiep`, `quoc_te`, `trong_nuoc`, `thong_cao`), framework chấm điểm impact nội bộ, case study thị trường VN, workflow đa tin, bảng dịch thuật ngữ tiếng Anh (phần 9). Đọc đầu session khi câu hỏi liên quan tin tức, chính sách vĩ mô, hoặc yêu cầu bối cảnh sự kiện.

**`K_agent_db_06` — Phase & 3 danh mục hệ thống**
4 trạng thái pha, exposure, 7 chỉ số, cơ chế cơ cấu 3 danh mục, bộ số hiệu suất chính thức FROZEN + disclaimer bắt buộc, known gaps. Đọc khi user hỏi đích danh về pha thị trường / danh mục hệ thống / hiệu suất / sổ lệnh (xem scope ở mục 4.6).

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

**Giới hạn cơ sở "backtest":** sổ `phase_trading`/`phase_perf` trong DB là backtest của HỆ PHASE danh mục (survivorship-biased, gross) — chỉ dùng khi trả lời câu hỏi trực tiếp về hệ phase, luôn kèm disclaimer `K_agent_db_06` mục 5. KHÔNG dùng làm base rate để gán xác suất cho kịch bản VNINDEX hoặc mã riêng lẻ.

**Hiệu suất 3 danh mục hệ — luật 2 tầng** (chi tiết `K_agent_db_06` mục 4): số tổng kết/dài hạn (CAGR, Sharpe, MaxDD, theo năm) CHỈ trích bảng FROZEN kèm disclaimer, không tự tính; cửa sổ ngắn (tuần/tháng/YTD) được compound từ `phase_perf` nhưng bắt buộc dán nhãn "gross chưa trừ phí/thuế".

**Phân bổ % danh mục:** được phép (user là analyst nội bộ), nhưng phải gắn với giả định rõ về khung thời gian, mức rủi ro, vốn ban đầu giả định.

**Target giá:** chỉ nói khi có mức kỹ thuật xác định (Fibonacci, pivot, volume profile POC). Đây là mô tả mức kỹ thuật, không phải dự báo điểm đến. Không dùng "target" theo nghĩa "giá sẽ về" trừ khi có model định giá độc lập.

### 4.4. Giới hạn tư vấn

Pack giả định user là analyst/broker nội bộ, được phép nhận khuyến nghị cụ thể. Nếu project chuyển cho audience khác (retail, client cuối, intern chưa chứng chỉ), cần swap sang K pack phù hợp — pack này không phục vụ được vì constraint nội dung rộng hơn mức cho phép retail.

**Lưu ý hành văn file con:** một số đoạn trong file con (đặc biệt `K_agent_db_06`) kế thừa từ bản gốc soạn cho agent phục vụ NĐT khách Finext, hành văn "khách/anh chị" — đọc là "user cuối của output". Nội dung số liệu và luật trình bày giữ nguyên giá trị.

Khuyến nghị phải:
- Gắn với giả định rõ (khung thời gian, mức rủi ro, vốn giả định) — nêu tự nhiên trong bài, khi kết luận phụ thuộc vào nó
- Cân bằng lập luận ủng hộ và phản đối, không chỉ 1 chiều
- Rõ ràng rằng quyết định cuối thuộc về người đọc — diễn đạt tự nhiên, không lặp nguyên văn cùng một câu kết ở mọi khuyến nghị
- Không hứa hẹn lợi nhuận, không dùng "chắc chắn tăng/giảm", "không thể lỗ"

### 4.5. Whitelist 18 ngành phân tích — Default scope + User override

**Default mode (user không nói gì cụ thể):** DB lưu **24 ngành** nhưng pack **mặc định chỉ phân tích 18 ngành** trong whitelist. Mọi query / aggregate / ranking / báo cáo cấp ngành filter theo whitelist này; các ngành ngoài whitelist không xuất hiện trong báo cáo. Mã thuộc ngành ngoài whitelist vẫn phân tích đơn lẻ được nếu user hỏi đích danh ticker, nhưng không vào watchlist theme / sector tilts.

**Override mode (user yêu cầu cụ thể ngành ngoài whitelist):** vd "phân tích ngành Bảo hiểm", "so sánh BVH vs VNM", "BCTC ngành Y tế Dược phẩm Q1" — agent **vẫn query được và trả lời bình thường** (dữ liệu 24 ngành đầy đủ trong DB). Khi đó:
- Query thẳng theo `industry_name` user yêu cầu, không apply whitelist filter
- Ghi note "ngoài scope whitelist mặc định" trong output để user biết
- Không tự ý so sánh với các ngành whitelist trừ khi user yêu cầu rõ

Danh sách 18 mã ngắn (user nhập) ↔ tên DB chuẩn (`industry_name` / `industry`) + cách áp dụng filter (cả Default + Override mode) và xử lý re-rank: xem **`K_agent_db_01`** đầu Section B "Khối ngành".

Khi user nhập mã ngắn (vd "DAUKHI", "NGANHANG"): map sang tên chuẩn DB để query; xuất báo cáo dùng tên đầy đủ, không lộ mã ngắn.

**Lưu ý riêng tầng phase:** `phase_industry` chỉ theo dõi **12 ngành** của rổ Sóng Ngành — taxonomy KHÁC với whitelist 18 ngành phân tích. Không trộn hai danh sách (một cái là danh mục hệ, một cái là scope phân tích dòng tiền).

### 4.6. Tín hiệu phase của hệ thống — nguồn tham chiếu (MỚI v2)

DB có tầng phase & danh mục (chi tiết `K_agent_db_06`): mô hình 4 trạng thái **UPTREND / DOWNTREND / SIDEWAY / TRANSITION** + tỷ lệ nắm giữ gợi ý `exposure` (0..2.0). Phase là MỘT nguồn tín hiệu ngang hàng với dòng tiền / kỹ thuật / cơ bản — **không phải luật tối cao, không tự động override các lăng kính khác**.

- **NHÃN pha của hệ chỉ trích từ `market_phase`** (headline có sẵn trong `data_briefing` doc `core`) — không tự gán nhãn pha "thay" hệ. Đánh giá xu hướng ĐỘC LẬP từ trend/breadth/dòng tiền (`K_agent_db_04`) vẫn là kết luận của agent; khi lệch với nhãn `market_phase`, trình bày CẢ HAI góc nhìn và nêu rõ điểm lệch — không mặc định bên nào thắng.
- **Nêu pha khi nó thật sự trả lời câu hỏi — tự phán đoán, không có luật chèn.** Pha luôn sẵn trong doc `core` để agent tự định vị. Đưa vào output khi nó là thứ user cần biết (user hỏi thị trường/tỷ lệ nắm giữ/danh mục hệ, hoặc khuyến nghị mâu thuẫn rõ với tín hiệu hệ — ví dụ khuyên mở vị thế lúc hệ đang 100% tiền mặt, khi đó nói rõ điểm lệch + lý do). Không biến thành block "Bối cảnh hệ thống" chèn máy móc vào mọi câu — thiếu thì user hỏi thêm.
- **Scope trong analysis_agent:** tầng phase là knowledge tra cứu (`K_agent_db_02` Workflow M, `K_agent_db_06`). **Các P pack giữ methodology regime riêng của từng pack** (gate vĩ mô, regime call...) — không thay bằng phase, không trộn phase vào checkpoint/regime call của P pack trừ khi user yêu cầu đích danh.

## 5. K hygiene — ký hiệu cần dịch trước khi output

Rule K hygiene ở system prompt mục 5.5 bắt buộc dịch ký hiệu raw và taxonomy nội bộ sang ngôn ngữ tự nhiên. Pack này định nghĩa 3 nhóm cần dịch và bảng dịch tương ứng.

### 5.1. Ba nhóm ký hiệu

**Nhóm 1 — Ký hiệu DB raw:**
`vsi`, `VSI`, `day_score`, `week_score`, `zone` với giá trị `A/AA/AAA/B/C`, `f382`/`f500`/`f618`, `poc`/`val`/`vah`, `r1`/`s1`, `period: "2025_4"`, `m_pct`/`w_pct`/`q_pct`/`y_pct`, `w_trend`/`m_trend`/`q_trend`/`y_trend`, `rank_pct`, `industry_rank_pct`, `market_rank_pct`, các key phase (`breadth_slow`, `breadth_blend`, `breadth_aux`, `conf_dir`, `conf_flat`, `corr60`, `px_ret20_pct`, `exposure`, `market_exposure`, `suppressed`, `held`, `book`), status rank (`trong_ro`/`vung_buffer`/`ung_vien`/`cho_tin_hieu`/`ngoai`), `exit_reason` (`HOLDING`/`DOWNTREND`/`ROTATION`/`REBALANCE`).

**Nhóm 2 — Taxonomy nội bộ methodology (từ file 04, 05):**
- Tên kịch bản trend đa khung: "Kịch bản A/B/C/D/E/F/G"
- Tên kịch bản ticker: "Kịch bản E1/E2/E3"
- Tên pitfall: "Pitfall F1" đến "F12"
- Tên section hoặc workflow: "B5", "B6", "B7", "C6", "D1-D4", "Workflow A-M", "Bước 1/2/3 của B7"
- Tên 4 kịch bản SXKD: "Value Play", "Value Trap", "Growth at Premium", "Cycle Top"
- Tên nhãn chấm điểm tin: "HIGH/MID/LOW impact", "logic gate", "framework chấm điểm", "impact score", "điểm X/Y"

Đây là công cụ nội bộ để agent dùng khi suy luận, không bao giờ lộ ra output. Thay vào đó mô tả trực tiếp tác động cụ thể và cơ chế bằng ngôn ngữ tự nhiên.

**Nhóm 3 — Thuật ngữ tiếng Anh chuyên môn chưa dịch:**
"mean-reversion", "exhaustion", "dead-cat bounce", "confluence level", "Market Profile", "DuPont decomposition", "Golden Ratio retracement", "Value Area", "bear trap", "bull trap", "whip-saw".

Thuật ngữ tiếng Anh trong phân tích tin tức (bảng dịch đầy đủ ở `K_agent_db_05` phần 9): "sell on news", "priced-in", "dot plot", "forward guidance", "hawkish/dovish", "contango/backwardation", "tailwinds", "confluence", "divergence", "smart money", "wash-out", "risk-off/risk-on", "pump and dump", "going concern", "cross-default", "one-off", "pre-sales".

Viết tắt thông dụng có thể giữ nguyên: Fed, FOMC, CPI, NFP, PCE, PMI, DXY, VIX, FDI, FII, ESOP, M&A. Giải thích ngắn khi dùng lần đầu trong session.

**Exception — 4 nhãn pha thị trường:**

`UPTREND` / `DOWNTREND` / `SIDEWAY` / `TRANSITION` là tên hiển thị chính thức của hệ phase (đã publish trên UI Finext) — được dùng NGUYÊN VĂN trong output, không cần dịch. Ngưỡng công bố của 7 chỉ số phase (±0.30 · 0.45 · 0.35 · −10%) cũng được phép nói; công thức/trọng số/cách kết hợp thì KHÔNG BAO GIỜ (xem `K_agent_db_06` mục 2).

**Exception — Slug trong URL finext.vn:**

`article_slug` và `report_slug` thuộc Nhóm 1 (ký hiệu DB raw), cấm lộ dạng trần trong output (ví dụ không viết `article_slug: vnm-bao-cao-q1`). Tuy nhiên khi ghép vào URL đầy đủ `https://finext.vn/news/{article_slug}` hoặc `https://finext.vn/reports/{report_slug}`, đây là output user-facing hợp lệ — URL công khai, không phải ký hiệu nội bộ DB. Field `link` (URL bài báo GỐC nguồn ngoài, có sẵn trong news feed từ v2) cũng là output hợp lệ, dùng nguyên văn. Chi tiết pattern xem `K_agent_db_01` section F (URL pattern — Dẫn link finext.vn).

### 5.2. Bảng dịch ký hiệu DB sang ngôn ngữ tự nhiên

⚠ Đơn vị theo quy ước v2 (mục 6): `*_pct` đã là điểm %, `rank_pct` thang 0–100.

| DB raw | Cách nói với user |
|---|---|
| `vsi: 2.1` | thanh khoản gấp 2.1 lần trung bình 5 phiên |
| `technical_zone.overall.w: "AAA"/"AA"/"A"/"B"/"C"` | vùng kỹ thuật khung tuần: rất mạnh / mạnh / tích cực / trung tính / yếu |
| `day_score: 68` | điểm dòng tiền ngày 68 |
| `week_score: -18` | dòng tiền tuần âm 18, đang bị rút ra |
| `breadth_in: 127, breadth_out: 171` | 127 mã tăng, 171 mã giảm, bên bán thắng thế |
| `industry_rank_pct: 90` | top 10% mạnh nhất ngành (percentile 0–100 của mã trong ngành) |
| `market_rank_pct: 95` | top 5% mạnh nhất thị trường (percentile 0–100 của mã trong thị trường) |
| Rank ngành-vs-ngành | DB không lưu — tự tổng hợp sort `week_score` (dòng tiền tuần) qua 18 ngành whitelist; xem `K_agent_db_01` mục "Xếp hạng ngành" |
| `fibonacci.w.f382: 1763` | hỗ trợ Fibonacci 38.2% khung tuần quanh 1763 |
| `volume_profile.w.poc: 1750` | vùng giá tập trung giao dịch quanh 1750 |
| `volume_profile.w.val / vah` | biên dưới / biên trên vùng giá chấp nhận |
| `nn.week.net_value: 4.2` | khối ngoại mua ròng 4.2 tỷ tuần qua (block `nn`/`td` vắng mặt = không có dữ liệu, KHÔNG phải "mua ròng 0") |
| `ROE: 0.23` (trong `stock_finstats`) | ROE 23% (bộ finstats còn thập phân — nhân 100) |
| `period: "2025_4"` / `"2025_5"` | Q4/2025 / cả năm 2025 |
| `w_pct: -1.06` | giảm 1.06% trong tuần qua (điểm % — đọc thẳng, KHÔNG nhân 100) |
| `m_pct: 6.2` / `q_pct: -3.3` / `y_pct: 46` | tăng 6.2% tháng / giảm 3.3% quý / tăng 46% năm (cùng quy ước điểm %) |
| `w_trend: 0.35` | xu hướng tuần 35% (35% số mã đang trên đường trend tuần — tỷ lệ 0..1, nhân 100 khi nói) |
| **Phase:** `breadth_slow` | Cấu trúc xu hướng tăng (vượt +0.30 mới đủ điều kiện TĂNG) |
| `breadth_blend` / `breadth_aux` | Cấu trúc xu hướng giảm (dưới −0.30 → GIẢM) / Tín hiệu xu hướng suy yếu |
| `conf_dir` / `conf_flat` | Độ tin cậy xu hướng / Độ tin cậy Sideway |
| `corr60` | **Đồng pha xu hướng – thanh khoản** — cấu trúc xu hướng và cường độ thanh khoản có đi cùng nhịp không (dưới 0.35 = rời nhịp, đà chưa được thanh khoản xác nhận; dưới 0 = ngược nhịp). ⚠ KHÔNG đo dòng tiền vào/ra, KHÔNG suy ra "vài mã lớn kéo chỉ số" |
| `px_ret20_pct` | Quán tính biến động giá (lợi suất 20 phiên, điểm %) |
| `exposure: 0.85` | tỷ lệ nắm giữ gợi ý 85% (thang 0..2.0, nhân 100 khi nói; >1.0 = có margin, kèm cảnh báo) ⚑ **Trạng thái KHÔNG quyết định mức an toàn** — cùng TRANSITION vẫn có thể 1.0 hoặc 0.5; ≤0.55 ở TRANSITION = vùng rủi ro cao, phải nói rõ, KHÔNG giải thích cơ chế |
| `market_intensity` | thước đo cường độ thị trường (−1 tới +1) |
| `suppressed: true` | cờ NỘI BỘ — tín hiệu giảm đã hội đủ nhưng chưa được xác nhận → tỉ trọng gợi ý bị hạ sâu. ⛔ KHÔNG giải thích cơ chế/công thức cho user; chỉ nói bối cảnh thị trường xấu, rủi ro cao |
| rank `status`: `trong_ro`/`vung_buffer`/`ung_vien`/`cho_tin_hieu`/`ngoai` | đang nắm giữ / đang giữ nhưng sắp ra / chờ vào / đủ hạng chờ tín hiệu giá / ngoài danh mục |
| `exit_reason`: `HOLDING`/`DOWNTREND`/`ROTATION`/`REBALANCE` | đang giữ / bán cả rổ do thị trường phòng thủ / đảo ngành / cơ cấu định kỳ |

### 5.3. Bảng dịch taxonomy nội bộ

Bảng dịch taxonomy đầy đủ (Kịch bản A–G, E1–E3, thuật ngữ kỹ thuật EN như mean-reversion / exhaustion / Value Trap / DuPont / Golden Ratio / whip-saw) đặt ở **đầu `K_agent_db_04`** (mục "Bảng dịch taxonomy nội bộ"). Thuật ngữ tin tức: `K_agent_db_05` phần 9. Nhãn phase và status danh mục: bảng 5.2 trên + `K_agent_db_06`.

## 6. Quy đổi đơn vị (v2 — pipeline fnx05 đã chuẩn hoá)

| Field / suffix | Quy ước | Ví dụ |
|---|---|---|
| mọi `*_pct`, `pct_change` | **ĐIỂM PHẦN TRĂM** — đọc thẳng, KHÔNG nhân 100 | `w_pct: -1.06` = giảm 1.06% |
| `industry_rank_pct` / `market_rank_pct` | percentile **0–100** | `90` = vượt 90% mã (top 10%) |
| `*_trend` | **tỷ lệ 0..1** (ngoại lệ có chủ đích) — nhân 100 khi nói | `w_trend: 0.35` = 35% số mã trên trend tuần |
| `exposure` / `market_exposure` | thang **0..2.0** — nhân 100 khi nói; >1.0 = dùng margin | `0.85` = nắm 85% |
| `held` / `book` / `avg_weight` | tỷ trọng **0..1** — nhân 100 khi nói | `0.0909` = 9.09% danh mục |
| `ret_1d_1x` (phase_perf) | lợi suất ngày **thập phân** — compound `Π(1+r)−1`, nhân 100 khi nói | `0.0021` = +0.21% |
| BCTC trong `stock_finstats` (Doanh thu, Tổng tài sản…) | **đồng** — chia 10^9 ra tỷ đồng | `9864419377152` → 9.864 tỷ đồng |
| tỷ lệ trong `stock_finstats` (ROE, biên, tăng trưởng) | ⚠ còn **thập phân** (bộ cũ, chờ curated) — nhân 100 khi nói | `0.216` = 21.6% |
| Vốn hoá trong `valuation_ratios` · GTGD (`trading_value`) · NN/TD (`buy/sell/net_value`, kể cả `history_nntd_*`) | **tỷ đồng** — `sell_value` luôn ÂM, `net_value = buy + sell`; >0 mua ròng, <0 bán ròng | `-94.73` = bán ròng 94.73 tỷ |
| `pe` `pb` `ps` `pcf` `ev_ebitda` `peg` (history_finratios_*) | **số lần** — đọc thẳng, KHÔNG nhân 100 | `13.08` = 13.08 lần |
| `marketcap` `revenue_ttm` `profit_ttm` (history_finratios_*) | **tỷ đồng** — ⚠ KHÔNG chia 10^9 (khác BCTC trong `stock_finstats` vốn là đồng) | `187856` = 187.856 tỷ |
| `eps` `bvps` (history_finratios_*) | **đồng / cổ phiếu** | `2499` = 2.499 đ/cp |
| `volume`, share counts, `foreignerRoom`, breadth | số nguyên (cổ phiếu / số mã) | — |
| `vsi` / `volume_strength_index` | lần so trung bình 5 phiên | `2.1` = gấp 2.1 lần |
| `other_data.value` | đọc kèm `unit`; lãi suất unit `%` là thập phân (`0.045` = 4.5%) — riêng các field `*_pct` cùng doc ĐÃ là điểm % | — |
| ngày `date`/`as_of` | string `YYYY-MM-DD` · intraday (`*_itd`) `YYYY-MM-DDTHH:MM` | — |

Field không có trong doc = không có dữ liệu (pipeline omit null) — nói "chưa có dữ liệu", không đoán, không coi là 0.

## 7. Độ tươi dữ liệu

- `data_briefing` doc `core` → `as_of` = mốc dữ liệu vòng ghi mới nhất; không phải hôm nay thì ghi "số liệu đến phiên [DD/MM]"
- `market_phase.as_of` (EOD đã chốt) có thể trễ hơn `core.as_of` (realtime) 1 phiên trong giờ giao dịch — lệch thì nêu cả hai mốc; lệch >1 phiên thì cảnh báo dữ liệu phase cũ
- `other_data.update_date`: chỉ số vĩ mô tháng (CPI, XNK, PMI) có thể cũ 2-3 tuần, luôn ghi chú ngày cập nhật
- BCTC công bố trễ 1-2 tháng sau quý — check `period` mới nhất, ghi rõ "số cơ bản đến Qx/YYYY"
- ⚠ `history_finratios_*`: BCTC được gán vào **ngày kết thúc kỳ** (31/12, 31/03…) chứ không phải ngày công bố → chuỗi có **look-ahead 1–2 tháng**. Mô tả/so sánh thì được; CẤM nói "lúc đó P/E đã rẻ rồi" hay dùng làm tín hiệu backtest. Điểm dữ liệu là **TUẦN**, không phải phiên (methodology: `K_agent_db_04` mục D6)
- ⚠ `history_nntd_*` (lịch sử khối ngoại/tự doanh) có thể **trễ vài phiên** so với `stock_nntd`/`market_nntd`. Cần số MỚI NHẤT → dùng bản snapshot; cần CHUỖI dài → dùng bản history. Luôn đọc `date` của điểm cuối trước khi gọi nó là "phiên hôm nay"
- Tin từ DB: rolling 30 ngày, luôn đối chiếu web search để lấy tin mới hơn nếu có

## 8. Lăng kính phân tích cốt lõi

Dòng tiền là lăng kính trung tâm. DB `agent_db` được tối ưu cho phân tích dòng tiền, đây là lợi thế cạnh tranh của pack. Mọi phân tích tổng hợp phải có tối thiểu 1 luận điểm dòng tiền: điểm số ngày hoặc tuần, xếp hạng ngành, xếp hạng thị trường, khối ngoại, độ rộng, cường độ thanh khoản.

**Ba lăng kính** cho phân tích chi tiết theo thứ tự:
1. Dòng tiền (trước)
2. Kỹ thuật (MA, Fibonacci, volume profile, zone)
3. Cơ bản (định giá, BCTC, tăng trưởng)

Pha hệ thống (`market_phase`) là tín hiệu tham chiếu bổ sung cho phân tích tổng hợp/khuyến nghị — trích làm bối cảnh theo mục 4.6, không đứng trên các lăng kính khác.

**Lồng vĩ mô khi liên quan** (gợi ý mapping ngành và chỉ số cần theo dõi — chi tiết `K_agent_db_02` Workflow I):

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
- **Field vắng mặt trong doc = không có dữ liệu** (pipeline v2 omit null, không còn `null`/`NaN`) — bỏ qua, không đoán, KHÔNG coi là 0. Ví dụ: mã không có block `nn` = không có dữ liệu khối ngoại, không phải "mua ròng 0"
- Ticker không có trong `stock_info` thì báo "Mã [X] không có trong hệ thống, kiểm tra lại", không đoán mã tương tự
- `data_briefing` chỉ có 2 doc (`core` + `news_report`) — cần bảng ngành / vĩ mô / nhóm chi tiết thì query thẳng collection gốc (`industry_snapshot`, `other_data`, `group_snapshot`)
- Web search không có kết quả thì báo "không tìm được tin gần đây về [X]", không bịa
- **Known gaps — hệ thống KHÔNG có, nói thẳng thay vì query lung tung:** lịch cổ tức & sự kiện quyền (GDKHQ, ESOP, phát hành thêm, ngày ĐHCĐ) · danh sách cổ đông lớn chi tiết (chỉ có tỷ lệ tổng `major_holdings_pct`) · dữ liệu lệnh/tài khoản cá nhân. Các câu này: dùng tin tức trong DB + web search, nói rõ giới hạn

## 10. Output contract

Pack này sinh ra **structured content** để layer trên (P pack, O pack) tiêu thụ. Ràng buộc:

- Số liệu định lượng phải đã quy đổi đơn vị theo mục 6
- Ký hiệu DB raw phải đã dịch theo bảng mục 5.2
- Taxonomy nội bộ phải đã thay bằng mô tả trực tiếp (bảng taxonomy đầu `K_agent_db_04`, xem mục 5.3)
- Mỗi claim có nguồn truy được: tên collection + field, hoặc URL web search
- Thông tin vĩ mô tháng hoặc tin tức có ghi chú ngày cập nhật
- Số hiệu suất danh mục hệ (nếu có) đúng luật 2 tầng + kèm disclaimer theo `K_agent_db_06` mục 4-5

Pack KHÔNG tự quyết format output cuối (heading, xưng hô, length, tone). Layer O quyết nếu có O pack active, ngược lại fall back Default Kernel (xem system prompt mục 6).
