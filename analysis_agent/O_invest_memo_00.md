# O_invest_memo_00 — Master Output Spec

File master của pack `O_invest_memo`. Spec cách render output cho quy trình `P_invest_memo`. Các file con trong pack (01-06) spec composition cụ thể cho từng loại báo cáo, đều reference rule chung ở đây.

**Pack dependency:** `O_invest_memo` phụ thuộc `P_invest_memo` (workflow produce content) và `K_agent_db` (K hygiene rule + citation pattern). O pack không produce analysis mới — chỉ render content từ state artifacts của P pack thành **MD final**.

> **Render binary out of scope:** Pack này dừng ở MD final. Render pptx/docx/xlsx là concern downstream, không thuộc scope pack. MD final đã đủ structured (heading + chart YAML + citation + locale) để consume bằng tool render bên ngoài. Mục pptx/docx render ở các file con là legacy spec, sẽ được dọn ở audit pass tiếp theo.

## 1. Mục đích & scope

O pack tách **rendering** khỏi **analysis logic**. P pack produce content structured dạng MD state file (`tier0_...md`, `tier5C_<ticker>_...md`, v.v.). O pack đọc các state file này và render thành báo cáo cho user theo format yêu cầu (MD / docx / pptx).

**Boundary với P pack:**
- P pack: quyết định regime, chọn ngành, chấm điểm mã, viết memo 7 phần → output state MD
- O pack: đọc state MD, gộp khi cần, apply giọng văn và structure, render thành deliverable format

**Boundary với K pack:**
- K pack: K hygiene rule (ký hiệu raw cần dịch, taxonomy nội bộ không lộ), citation format finext.vn
- O pack: apply các rule này **run-time khi render** — đảm bảo output cuối cùng không vi phạm K hygiene

**6 báo cáo pack này cover:**

| File con | Báo cáo | Gộp từ state |
|---|---|---|
| `O_invest_memo_01` | Market scan | tier0/1/2/3 (4 file → 1 báo cáo) |
| `O_invest_memo_02` | Stock memo | tier5A + tier5B + tier5C (3 file → 1 memo/mã) |
| `O_invest_memo_03` | Portfolio plan | tier6 portfolio |
| `O_invest_memo_04` | Weekly review | tier7 weekly |
| `O_invest_memo_05` | Monthly review | tier7 monthly |
| `O_invest_memo_06` | Quarterly review | tier7 quarterly |

**Nhóm 5 state file (audit log, lessons learned) không cover** — đây là internal doc, giữ dạng MD nội bộ theo P pack quy định, không cần render formal.

## 2. Hai nguyên tắc operational bất biến

Khác với P pack có default mode, O pack **không có default format, không có default template**. Rendering là user-facing, user phải control explicit.

**Nguyên tắc 1 — Format không mặc định.**

Khi user yêu cầu produce 1 báo cáo, agent phải xác định format output trước khi render. 3 format supported: MD, docx, pptx.

- Nếu user nói rõ ("xuất memo VNM dạng docx", "làm báo cáo tuần, MD thôi") → làm theo
- Nếu user không nói format → **hỏi trước**, không tự quyết. Câu hỏi dạng ngắn: "Bạn muốn xuất ở format nào — MD, docx, hay pptx?"
- Không có rule "loại báo cáo A default format X". Mỗi lần render là 1 lần hỏi hoặc 1 lần user chỉ định.

Lý do: cùng 1 báo cáo có thể user cần MD (copy vào Zalo) hoặc docx (gửi sếp) tuỳ tình huống. Agent không đoán được ý định.

**Nguyên tắc 2 — Template không mặc định.**

Khi format là docx hoặc pptx, agent phải hỏi user dùng template nào trong `templates/` folder. 

- Nếu user nói rõ ("dùng template_finext_memo_v2.docx") → làm theo
- Nếu user không nói → **hỏi trước**. Có thể list các template có sẵn trong project knowledge nếu biết, hoặc hỏi: "Bạn muốn dùng template nào cho docx/pptx này?"
- Không tự pick "template đầu tiên thấy" hoặc "template có tên gần nhất". Không tự build template từ scratch nếu user không chỉ định.

Lý do: sau này project sẽ có nhiều template brand khác nhau (public report, internal, client-specific). Agent không tự chọn được — user biết rõ audience cuối nên mới pick đúng template.

**Format MD không cần template** — render theo spec content trong file con của pack này là đủ.

## 3. Workflow render output

Khi user yêu cầu produce báo cáo, agent thực hiện tuần tự:

**Bước 1 — Xác định loại báo cáo.** Map yêu cầu user sang 1 trong 6 file con:
- "memo VNM" / "memo mã X" → `O_invest_memo_02`
- "market scan" / "gate vĩ mô đầy đủ" / "báo cáo chọn ngành" → `O_invest_memo_01`
- "portfolio plan" / "danh mục" → `O_invest_memo_03`
- "báo cáo tuần" → `O_invest_memo_04`
- "báo cáo tháng" → `O_invest_memo_05`
- "báo cáo quý" → `O_invest_memo_06`

Nếu không rõ báo cáo nào → hỏi user clarify.

**Bước 2 — Xác định format.** Theo Nguyên tắc 1. User chỉ định hoặc hỏi.

**Bước 3 — Xác định template (chỉ docx/pptx).** Theo Nguyên tắc 2.

**Bước 4 — Load state artifacts.** Đọc các file MD gốc mà báo cáo cần (xem bảng ở Phần 1). Nếu state file thiếu hoặc chưa confirmed → báo user, không tự render với data incomplete.

**Bước 5 — Đọc spec file con tương ứng.** File con định nghĩa structure, giọng văn cụ thể cho loại báo cáo, cách gộp state.

**Bước 6 — Compose content MD.** Dù format cuối là gì, **luôn produce MD trước làm nguồn truth**. MD này chứa toàn bộ nội dung + chart annotation dạng YAML block (xem Phần 7). Nếu format cuối là MD → đây là output cuối, không cần render tiếp.

**Bước 7 — Render sang format cuối (nếu docx/pptx).** 
- Đọc skill `/mnt/skills/public/docx` hoặc `/mnt/skills/public/pptx` trước khi build
- Load template file user đã chỉ định từ `templates/`
- Convert MD content sang format docx/pptx theo layout template
- Render chart annotation thành chart thật (dùng data trong YAML block)

**Bước 8 — Present file cho user.** Dùng tool `present_files` với path file output.

## 4. Audience & tone default

Hiện tại O pack chỉ support 1 audience default:

**Analyst / broker nội bộ** — người dùng là analyst/broker chuyên nghiệp, đọc để ra quyết định đầu tư cá nhân hoặc chia sẻ peer. Đặc điểm:

- Quen số liệu định lượng, ratio, viết tắt phổ thông (P/E, ROE, YoY, NIM, NPL, CASA)
- Chấp nhận nhận định trực tiếp, không cần soft-pedal
- Cần evidence + source cho claim, không chấp nhận thesis chung chung
- Đọc nhanh, ưu tiên bullet + bảng cho data-heavy, prose cho nhận định

Giọng văn:
- Direct, không filler ("Kết luận: mua" không "Sau khi cân nhắc kỹ lưỡng, chúng tôi cho rằng có thể cân nhắc mua")
- Xưng hô trung tính (không "tôi", không "chúng tôi") — giọng của analyst report
- Nhận định có số liệu kèm ("ROE 25% vs median ngành 15%") không có số liệu ("ROE cao hơn ngành")
- Số liệu tiếng Việt (tỷ VND), ratio có 1 chữ số thập phân (15.5%)

**Các audience khác (Zalo khách, client-facing, formal cho sếp) chưa được build** trong phase này. Khi cần, sẽ extend master bằng cách thêm tone mode mới, hoặc build pack O riêng cho audience đó. Hiện tại mọi O file assume tone analyst/broker.

## 5. 3 format đặc điểm

**MD — Markdown:**
- Render trực tiếp content, không chart thật
- Chart annotation giữ nguyên dạng YAML block (hiển thị như code block khi đọc MD)
- Không cần template — structure đã đủ ở file con
- Output: file `.md` single, share được qua Slack/Zalo/email attachment
- Use case: internal share, draft trước khi render formal, copy vào chat

**Docx — Word document:**
- Render content từ MD sang Word với formatting (heading, bold, bảng, list)
- Chart annotation được render thành chart thật dùng data trong YAML block
- Cần template brand: header/footer, font, color, cover page
- Output: file `.docx`, share formal qua email, attach Slack/Zalo, in giấy
- Use case: memo share khách/sếp, báo cáo tháng/quý formal, archive

**Pptx — Presentation:**
- Render content sang slide deck theo layout template
- Chart annotation render thành chart thật trong slide
- Cần template brand: master slide, color scheme, font, logo
- Output: file `.pptx`, pitch trong họp, present màn hình
- Use case: pitch 1 mã, review meeting tháng/quý, client presentation

**Nguyên tắc trọng tâm:** MD là source of truth content. Docx và pptx là view khác của cùng content, khác format rendering + có chart thật + có template layout. Không được phép xuất docx/pptx mà khác nội dung với MD đã produce.

## 6. Content composition rules (apply mọi báo cáo)

Các rule dưới đây apply khi compose MD content ở Bước 6 của workflow. File con có thể thêm rule specific, không được mâu thuẫn với rule này.

**K hygiene apply-time:**
- Không để ký hiệu DB raw trong output: `w_trend`, `day_score`, `zone: A`, `f382`, `rank_pct`, v.v. Dịch theo bảng ở `K_agent_db_00` mục 5.2
- Không lộ taxonomy nội bộ: "Kịch bản B", "Pitfall F5", "HIGH impact", "framework chấm điểm". Mô tả trực tiếp hiện tượng bằng ngôn ngữ tự nhiên
- Không để thuật ngữ tiếng Anh chưa dịch ở `K_agent_db_05` phần 9: "priced-in", "hawkish", "divergence". Dịch hoặc giải thích ngắn

**Exception:** `article_slug`, `report_slug` cấm lộ dạng trần nhưng khi ghép vào URL finext.vn đầy đủ thì OK (xem `K_agent_db_00` mục 5.1 Exception).

**Thuật ngữ chuyên môn của hệ thống — dịch inline lần đầu xuất hiện:**

Các thuật ngữ workflow của pack `P_invest_memo` cần có ngoặc dịch tiếng Việt khi xuất hiện lần đầu trong mỗi báo cáo. Các lần xuất hiện sau trong cùng báo cáo không cần dịch lại. Pattern: `English term (dịch ngắn)`.

Bảng thuật ngữ + dịch chuẩn:

| Thuật ngữ gốc | Dịch inline |
|---|---|
| Conviction tier: High / Medium / Low | Conviction tier (mức độ tự tin): High (cao) / Medium (trung bình) / Low (thấp) |
| Bucket 1 | Bucket 1 (vào ngay) |
| Bucket 2 | Bucket 2 (chờ pullback xác nhận) |
| Bucket 3 | Bucket 3 (watchlist, chưa vào) |
| Phase 1 / 2 / 3 | Phase 1 (tranche đầu) / Phase 2 (tranche kế tiếp, chờ confirm) / Phase 3 (upgrade từ watchlist) |
| Regime: Risk-on full | Regime (trạng thái thị trường): Risk-on full (ưa rủi ro toàn diện) |
| Regime: Risk-on selective | Risk-on selective (ưa rủi ro chọn lọc) |
| Regime: Defensive only | Defensive only (chỉ phòng thủ) |
| Regime: Đứng ngoài | Đứng ngoài (giữ tiền mặt, không vào mới) |
| Variant perception | Variant perception (góc nhìn khác consensus thị trường) |
| Catalyst | Catalyst (chất xúc tác đẩy giá) |
| Horizon | Horizon (khung thời gian giữ vị thế) |
| Conviction memo | Conviction memo (memo có kết luận đầu tư) |
| Universe filter | Universe filter (bộ lọc loại thẳng) |
| Ranking filter | Ranking filter (bộ lọc xếp hạng, không loại) |
| Funnel A / B / C / D | Funnel A (dòng tiền + kỹ thuật) / B (cơ bản) / C (catalyst) / D (thanh khoản) |
| Steelman bear case | Bear case steelmanned (luận điểm phản đối viết ở dạng mạnh nhất) |
| Checkpoint / CP | Checkpoint (điểm dừng chờ xác nhận) |

**Thuật ngữ tài chính widely adopted KHÔNG cần dịch inline:** P/E, P/B, ROE, ROA, ROIC, EBIT, EBITDA, YoY, QoQ, MoM, CAGR, ADV, NPL, NIM, CASA, FCF, FCFF, WACC, DCF, IRR, TV (terminal value), BVPS. Analyst/broker đọc mặc định hiểu.

**Thuật ngữ technical ít gặp hơn vẫn cần dịch inline:** LLCR, FVTPL, SOP (sum-of-parts), RIM (residual income model), DSO/DIO/DPO/CCC, tornado chart, sensitivity analysis, embedded value, margin of safety, re-rating, overhang, moat, soft-pedal. Apply theo convention pack `P_invest_memo`.

**Citation & source:**

Phân loại 4 nhóm nguồn với citation format khác nhau:

**Nhóm 1 — Dữ liệu từ agent_db nội bộ (stock_finstats, stock_snapshot, industry_*, market_*, other_data, v.v.):** Ghi chung **"(nguồn: Tổng hợp)"**. Không lộ tên collection, không ghi cụ thể ticker/period vì thông tin này đã có trong context body. Đây là extension của K hygiene (tên collection thuộc ký hiệu DB raw, cấm lộ trong output).

Ví dụ đúng: "Revenue VNM 2025 đạt 18,200 tỷ VND, tăng 10.3% YoY (nguồn: Tổng hợp)."
Ví dụ sai: "Revenue VNM 2025 đạt 18,200 tỷ VND (nguồn: stock_finstats Q4/2025)."

**Nhóm 2 — Tin/báo cáo từ agent_db (news_*):** Dẫn link finext.vn đầy đủ, link chính là source — không cần kèm "(nguồn: Tổng hợp)". Pattern: `https://finext.vn/news/{article_slug}` hoặc `https://finext.vn/reports/{report_slug}`.

Ví dụ: "NVL công bố bổ sung nội dung ĐHCĐ liên quan tái cấu trúc nợ ([Finext, 18/4/2026](https://finext.vn/news/nvl-truoc-dhcd-novaland-bo-sung-noi-dung-quan-trong))."

**Nhóm 3 — PDF user upload (BCTC, BCTN, báo cáo sell-side nếu user upload):** Ghi cụ thể tên tài liệu + trang (nếu có). Đây là nguồn external user cung cấp, không thuộc agent_db.

Ví dụ: "DSO tăng từ 45 lên 52 ngày trong 2 năm (BCTC VNM Q4/2025 soát xét, thuyết minh mục 8)."

**Nhóm 4 — Web external (tradingeconomics, damodaran, báo chí ngoài finext, sell-side public):** Dùng markdown link cụ thể.

Ví dụ: "ERP VN Damodaran 2025 là 8.35% ([NYU Stern country risk](https://pages.stern.nyu.edu/~adamodar/))."

**Footnote khi nhiều citation:**

Body text dày citation đọc khó. Khi 1 đoạn có ≥ 3 citation external (Nhóm 3 hoặc 4), dùng footnote số `[1]` trong body và liệt kê nguồn cuối section hoặc cuối báo cáo:

```
Revenue 2025 đạt 18,200 tỷ VND (nguồn: Tổng hợp), cao hơn consensus sell-side 8-10%[^1] 
và đúng trend CAGR 15% mà Damodaran ngành retail VN dự báo[^2].

[^1]: [SSI VNM Update 03/2026](https://www.ssi.com.vn/...)
[^2]: [Damodaran retail Vietnam outlook](https://pages.stern.nyu.edu/...)
```

Nhóm 1 "(nguồn: Tổng hợp)" giữ inline, không cần footnote — nó ngắn và xuất hiện nhiều.
Nhóm 2 link finext.vn tuỳ ngữ cảnh: body flow cần thoáng → footnote, body ngắn → inline.

**Unit convention:**
- VND: mặc định tỷ VND cho tổng, triệu VND cho per-share khi cần, nghìn VND (hoặc "k") cho giá. Ghi đơn vị rõ: "18.200 tỷ VND", "85k/cp"
- Tỷ lệ: 1 chữ số thập phân cho % ("15,5%", "22,3%"). Không dùng basis point trừ khi ngữ cảnh banking/fixed income
- USD: "45 triệu USD", "1,2B USD" cho lớn
- Ticker: uppercase, không dấu nháy ("VNM", không "'VNM'" hay "vnm")
- Date: "Q1/2026", "tháng 4/2026", "ngày 15/4/2026". Không dùng "Q1 2026" hay "2026-04-15" trong prose (OK trong bảng)

**Number formatting (locale vi-VN, sync PROJECT_STATUS root convention):**
- Dấu chấm ngăn cách nghìn: "18.200 tỷ" không "18200 tỷ" hay "18,200 tỷ"
- Dấu phẩy thập phân: "15,5%" không "15.5%"
- Phần trăm có dấu rõ: `+18,2%` / `-3,5%`
- Rounding: revenue/LNST đến tỷ, margin đến 0,1%, price đến 0,1k

**Lưu ý:** ví dụ trong các file con (`O_invest_memo_01-06`) hiện có thể còn dùng convention cũ (`18,200 tỷ`, `15.5%`) — sẽ được normalize qua audit pass sau. Khi compose output mới, agent áp convention vi-VN root ở mục này.

**Heading & structure:**
- Heading cấp 1 (`#`) chỉ 1 lần ở đầu file (tiêu đề báo cáo)
- Heading cấp 2 (`##`) cho section chính
- Heading cấp 3 (`###`) cho subsection nếu cần
- Tránh cấp 4+ — báo cáo không nên quá sâu nesting
- Mỗi section ngăn cách bằng 1 dòng trống, không dùng horizontal rule (`---`) trong body trừ khi separate phần phụ lục

## 7. Chart annotation spec (quan trọng)

MD không render chart thật. Tại chỗ cần chart, agent chèn YAML block với data đầy đủ. Khi render docx/pptx, agent đọc block này build chart thật. MD đọc raw thì block hiển thị như code block.

**Syntax bắt buộc:**

````
```chart
type: bar | line | pie | scatter | stacked_bar | combo
title: [tiêu đề chart, tiếng Việt]
x_axis: [array hoặc field name]
y_axis: [array hoặc field name, có thể multiple series]
y_label: [unit label, ví dụ "Tỷ VND", "%"]
x_label: [optional, thường không cần với năm/tháng]
source: [nguồn data, ví dụ "stock_finstats", "BCTC VNM Q4/2025"]
render_in_md: skip  # luôn skip, MD không render
render_in_docx: true | false
render_in_pptx: true | false
note: [optional, chú thích hiển thị dưới chart]
```
````

**Chart types hỗ trợ:**
- `bar` — cột đứng, so sánh giữa categories (revenue 5 năm, margin so với peer)
- `line` — đường, trend theo thời gian (price history, ratio qua thời gian)
- `pie` — tròn, cơ cấu (segment revenue, allocation portfolio)
- `scatter` — điểm rời rạc, correlation 2 biến (P/E vs ROE peer)
- `stacked_bar` — cột chồng, composition theo time (revenue segments qua quý)
- `combo` — line + bar combined (revenue bar + growth rate line)

**Ví dụ chart revenue 5 năm VNM:**

````
```chart
type: bar
title: VNM Revenue 2021-2025
x_axis: [2021, 2022, 2023, 2024, 2025]
y_axis:
  - name: Revenue
    data: [10200, 12800, 14200, 16500, 18200]
y_label: Tỷ VND
source: stock_finstats
render_in_md: skip
render_in_docx: true
render_in_pptx: true
note: CAGR 15.5% qua 5 năm, gia tốc trong 2024-2025
```
````

**Ví dụ combo chart revenue + growth:**

````
```chart
type: combo
title: VNM Revenue & Growth 2021-2025
x_axis: [2021, 2022, 2023, 2024, 2025]
y_axis:
  - name: Revenue (bar)
    type: bar
    data: [10200, 12800, 14200, 16500, 18200]
    y_label: Tỷ VND
  - name: YoY Growth (line)
    type: line
    data: [null, 25.5, 10.9, 16.2, 10.3]
    y_label: '%'
    axis: secondary
source: stock_finstats + tính toán
render_in_md: skip
render_in_docx: true
render_in_pptx: true
```
````

**Khi nào đề xuất chart:**
- Dữ liệu ≥ 3 điểm có trend theo thời gian → line/bar
- So sánh ≥ 3 entity (peer, ngành, scenario) → bar
- Composition (segment, allocation) với ≥ 3 thành phần → pie hoặc stacked_bar
- Sensitivity analysis 2D → scatter hoặc heatmap (nếu hỗ trợ)

Không chèn chart cho:
- Dữ liệu đơn (1 số) — viết prose
- Bảng ≤ 3 dòng — giữ bảng MD
- Thông tin qualitative không có số

**Giới hạn per báo cáo:**
- Market scan: 2-4 chart (trend thị trường, dòng tiền ngành)
- Stock memo: 4-8 chart (revenue/margin/ROE trend, valuation sensitivity, peer comparison)
- Portfolio plan: 1-2 chart (allocation pie, sizing distribution)
- Weekly/monthly: 1-3 chart (P&L cumulative, drift composition)
- Quarterly: 4-6 chart (BCTC mới vs assumption, target update)

Chart nhiều quá làm loãng nhận định — quality > quantity.

## 8. Section structure — rigid vs flex

Theo quyết định thiết kế: **rigid cho recurring, flex cho one-off**.

**Rigid structure (weekly / monthly / quarterly):**
- File con spec exact heading, thứ tự phần, bảng cố định
- Agent fill data vào slot, không tự thêm/bớt phần
- Lý do: user đọc weekly 24 lần/cycle — structure nhất quán giúp scan nhanh, biết tìm info ở đâu

**Flex structure (market scan / stock memo / portfolio):**
- File con spec **must-have elements** (section bắt buộc có) + **optional elements** (thêm theo context)
- Agent compose linh hoạt độ dài, thứ tự subsection tuỳ story
- Lý do: mỗi memo một câu chuyện khác, ép structure làm mất nuance

Chi tiết structure cụ thể ở từng file con.

## 9. Master-first reading rule

Khi user gọi produce báo cáo, agent đọc file **theo thứ tự**:

1. `O_invest_memo_00` (file này — master rules)
2. File con tương ứng (`O_invest_memo_01` đến `06`)
3. State MD từ P pack

Không skip master. Rule composition ở master apply cho mọi báo cáo, file con chỉ thêm spec cụ thể không được mâu thuẫn.

## 10. Cross-reference priority

Khi không chắc cách render:

1. File con tương ứng của O pack → spec format cụ thể cho báo cáo
2. `O_invest_memo_00` (master này) → rule chung, audience, K hygiene apply-time, chart spec
3. `K_agent_db_00` mục 5 → K hygiene bảng dịch chi tiết, citation pattern
4. `P_invest_memo_<NN>` → nếu cần hiểu content gốc của state file
5. System prompt mục 6 → output style framework layer

Nếu spec vẫn chưa rõ → hỏi user clarify trước khi quyết, theo rule clarification-before-analysis ở system prompt mục 5.4.
