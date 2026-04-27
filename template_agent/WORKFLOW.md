# WORKFLOW — Normalize & Render Flow

Workflow 7 stage + 3 checkpoint. Đầu vào: input bất kỳ định dạng. Đầu ra: MD chuẩn hóa (đúng `FORMAT.md`) + binary branded (qua TEMPLATE pack).

## 1. Tổng quan flow

```
─── Stage 1: Ingest ────────────────────────────────────
  Đọc input (PDF/DOCX/MD/text/copy-paste)
  Extract content thô + detect format

─── Stage 1.5: Detect skip-normalize ───────────────────
  Check input có match FORMAT contract sẵn?
  Nếu YES → confirm với user → Stage 6
  Nếu NO → Stage 2

─── Stage 2: Parse ─────────────────────────────────────
  LLM analyze: report_type, sections, data, ambiguities

─── Stage 3: Clarify ───────────────────────────────────
  Multi-choice questions hỏi user (gom 3-5 câu/turn)

─── CHECKPOINT 1: Clarification confirm ────────────────
  User trả lời → agent ghi nhận

─── Stage 4: Normalize ─────────────────────────────────
  LLM produce MD theo FORMAT contract
  Apply: heading hierarchy, frontmatter, chart YAML,
         citation format, locale vi-VN, K hygiene

─── CHECKPOINT 2: MD draft review ──────────────────────
  Show MD draft → user review/edit/confirm

─── Stage 5: Finalize MD ───────────────────────────────
  Apply user edit (nếu có) → MD final
  Self-audit theo FORMAT.md mục 9

─── Stage 6: Brand pre-flight ──────────────────────────
  Hỏi user: VBSE / Finext (whitelist 2)

─── CHECKPOINT 3: Brand confirm ────────────────────────
  User pick brand

─── Stage 7: Render binary ─────────────────────────────
  TEMPLATE pack runtime đọc MD final
  Match section → layout → fill placeholder
  Build native chart từ YAML block
  Save binary + present user
```

## 2. Stage 1 — Ingest

### 2.1. Input format support

| Format | Extraction method |
|---|---|
| `.md` / `.txt` | Đọc trực tiếp text |
| `.pdf` | Extract text + structure (tables, headings) qua PDF tool |
| `.docx` | Extract text + bảng + heading qua DOCX tool |
| Paste content | Đọc trực tiếp từ message user |
| URL public | Web fetch |
| Image có text | OCR — confirm với user nếu confidence thấp |

### 2.2. Output Stage 1

Content thô structured (text + bảng + heading detected). Lưu nội bộ làm input cho Stage 2.

### 2.3. Edge cases

- Input không đọc được → báo user lỗi cụ thể, đề xuất re-upload format khác
- Input rỗng / <100 từ → hỏi user xác nhận có muốn render với content tối thiểu không
- Multi-file input → hỏi user là 1 báo cáo gộp hay N báo cáo riêng

## 3. Stage 1.5 — Detect skip-normalize

Check input có phải MD đã match `FORMAT.md` contract không. Logic detection:

### 3.1. Detection signals

| Signal | Check |
|---|---|
| Frontmatter | Top file có `---` block với `report_type` thuộc 9 whitelist (8 preset + custom) |
| Heading hierarchy | H1 duy nhất + H2 cho section + không H4+ |
| Section count | Match expected count theo `report_type` (vd stock_pitch 13-16) |
| Chart YAML | Chart block dùng syntax ` ```chart` đúng spec |
| Citation pattern | Có ít nhất 1 trong: `(nguồn: Tổng hợp)` / link finext.vn / footnote `[^N]` |
| Locale | Số dùng `.` ngăn nghìn + `,` thập phân |

**Threshold:** ≥4/6 signals match → đề xuất skip.

### 3.2. Confirm với user

Khi detect skip-able, hỏi 1 câu:

> "Input MD đã match contract chuẩn ([N]/6 signals). Skip normalize, đi thẳng chọn brand template?
> (a) Có, đi thẳng Stage 6 [default]
> (b) Không, vẫn review qua normalize stage
> (c) Show signals detect chi tiết để tôi quyết"

Default (a). Nếu user chọn (a) → Stage 6. Nếu (b) → Stage 2.

## 4. Stage 2 — Parse

LLM analyze content thô, extract:

### 4.1. Outputs

- **report_type detection:** infer 1 trong 9 loại (stock_pitch / weekly_market / market_scan / stock_memo / portfolio_plan / portfolio_review_weekly / portfolio_review_monthly / portfolio_review_quarterly / custom). Confidence score. Nếu confidence < 80% hoặc không match 8 preset → đề xuất `custom` ở Stage 3.
- **Section list detected:** liệt kê heading có sẵn trong input
- **Required field check:** ticker / company_name / industry (nếu stock_pitch hoặc stock_memo); week_label (nếu weekly_market); conviction_tier (nếu stock_memo); portfolio_size_vnd (nếu portfolio_plan); etc.
- **Data inventory:** số liệu, bảng, chart data tìm thấy
- **Ambiguities:** điểm cần clarify (vd "Tìm thấy 4 luận điểm nhưng chỉ 3 có headline data — clarify luận điểm 4 có cần thêm data không")
- **Missing for contract:** field/section bắt buộc theo report_type nhưng chưa có (vd thiếu Variant Perception trong stock_pitch)

### 4.2. Báo cáo Stage 2

Internal note, không show user. Dùng để soạn câu hỏi Stage 3.

## 5. Stage 3 — Clarify

Soạn batch multi-choice question gửi user. Format chuẩn:

### 5.1. Format

```
Tôi cần xác nhận [N] điểm trước khi normalize:

**1. [Câu hỏi 1]**
   (a) [Option A] [default]
   (b) [Option B]
   (c) [Option C]

**2. [Câu hỏi 2]**
   (a) [...]
   (b) [...]

[...]

Bạn có thể trả lời "1a 2b 3a", hoặc "default" để áp dụng default cho tất cả.
```

### 5.2. Loại câu hỏi (preset report_type)

| Loại | Khi nào hỏi | Ví dụ |
|---|---|---|
| Confirm report_type | Confidence < 80% | "Loại báo cáo: (a) stock_pitch (b) weekly_market (c) market_scan (d) stock_memo (e) portfolio_plan (f) portfolio_review_weekly (g) portfolio_review_monthly (h) portfolio_review_quarterly (i) custom" |
| Required field | Field bắt buộc thiếu theo report_type | "Mã ticker không có trong input. Cung cấp ticker hoặc chuyển sang `custom`?" |
| Section missing | Section bắt buộc theo contract thiếu | "Báo cáo stock_pitch thiếu Variant Perception. (a) Tôi sẽ tự fill từ context (b) Bạn cung cấp (c) Skip" |
| Ambiguous content | Content có thể interpret nhiều cách | "Section 'Phân tích kỹ thuật' có 5 mức giá nhưng không label. (a) R1/R2/S1/S2/POC (b) Hỗ trợ tuần/tháng (c) Bạn label cụ thể" |
| Citation source | Nguồn data unclear | "Số 18.200 tỷ doanh thu: (a) BCTC quý gần nhất (b) Báo cáo annual (c) Sell-side estimate" |

### 5.3. Giới hạn

- Tối đa 5 câu/turn — nhiều hơn thì split 2 turn
- Mỗi câu 2-4 option — 5+ option dấu hiệu câu hỏi quá rộng, refine
- Luôn có default option khi hợp lý — minimize friction cho user

### 5.4. Quiz custom — chỉ chạy khi report_type = `custom`

Khi user pick `custom` (hoặc Stage 2 detect không match 8 preset), agent chạy **bộ trắc nghiệm custom** để xây spec runtime. Quiz có 7 câu, gom 2 turn (4 + 3).

**Turn 1 — Mục đích & audience:**

```
Để xây spec báo cáo tùy chỉnh, tôi cần hỏi 4 câu:

**1. Mục đích chính của báo cáo?**
   (a) Phân tích / nghiên cứu sâu 1 chủ đề (ngành / vĩ mô / chuyên đề)
   (b) Tổng hợp / review định kỳ (không khớp template review có sẵn)
   (c) So sánh / benchmark nhiều đối tượng
   (d) Đề xuất / pitch ý tưởng đầu tư không khớp stock_pitch
   (e) Khác — tả ngắn 1 câu

**2. Audience cuối?**
   (a) Nội bộ analyst / sếp / team
   (b) Khách hàng VIP / institutional
   (c) Khách hàng retail
   (d) Public (báo chí, social media)

**3. Length target?**
   (a) Ngắn 2-4 trang
   (b) Trung 5-8 trang [default]
   (c) Dài 9-15 trang
   (d) Rất dài 15+ trang

**4. Tone?**
   (a) Phân tích viên trung tính, evidence-based [default]
   (b) Formal memo (xưng hô trang trọng)
   (c) Conversational (thân thiện, dễ đọc)
   (d) Pitch (thuyết phục, conviction cao)
```

**Turn 2 — Section + chart + citation:**

```
3 câu cuối:

**5. Số section H2 dự kiến?**
   (a) 3-5 (ngắn gọn, focus)
   (b) 6-8 (vừa, đủ depth) [default]
   (c) 9-12 (sâu, đa chiều)
   (d) Để tôi đề xuất section list dựa trên mục đích

**6. Chart cần bao nhiêu?**
   (a) Không cần chart, prose-heavy
   (b) 1-3 chart minh họa key data [default]
   (c) 4-6 chart (data-heavy)
   (d) 7+ chart (visual-first report)

**7. Citation style preference?**
   (a) Inline link markdown [default]
   (b) Footnote [^1] cuối section
   (c) Hỗn hợp inline + footnote khi nhiều nguồn
   (d) Citation tối thiểu (chỉ "Tổng hợp" / "Internal data")

Bonus (optional): Bạn có **section list** muốn theo cụ thể không? Nếu có, paste list. Nếu không, tôi sẽ đề xuất từ catalog 27 layout có sẵn.
```

### 5.5. Build custom spec từ quiz answer

Sau Turn 2, agent compile spec:

1. **Generate `custom_spec_id`** = timestamp `YYYYMMDD_HHMMSS`
2. **Map quiz answer → frontmatter custom_*** field
3. **Section list:**
   - Nếu user paste list → dùng list đó, validate mỗi section có khớp layout TEMPLATE không
   - Nếu user chọn "Để tôi đề xuất" → agent suggest dựa trên mục đích:
     - Phân tích ngành/vĩ mô: Bối cảnh / Diễn biến / Driver chính / Top picks / Risks / Outlook / Disclaimer
     - Review định kỳ custom: Exec summary / Performance / Highlights / Issues / Next period focus / Disclaimer
     - So sánh: Giới thiệu / Methodology / [Per-entity sections] / Verdict / Disclaimer
     - Pitch: Hook / Thesis / Evidence / Counterargument / Action / Disclaimer
4. **Validate layout coverage:** mỗi section ánh xạ với 1 trong 27 layout. Nếu không match → flag user, đề xuất layout fallback `BULLET_LIST_SUMMARY`
5. **Show spec cho user confirm trước khi đi Stage 4:**

```
─── CUSTOM SPEC PROPOSAL ─────────────────────────────

custom_spec_id: 20260427_153022
custom_purpose: [tóm tắt 1 câu]
custom_audience: [audience]
custom_length_target: [pages]
custom_tone: [tone]
custom_chart_count: [N]
custom_citation_style: [style]

Section list đề xuất:
1. [Section 1] — match layout `[LAYOUT_ID]`
2. [Section 2] — match layout `[LAYOUT_ID]`
...

Confirm spec để tôi normalize content theo, hay edit?
(a) Confirm, đi Stage 4 normalize
(b) Edit section list (paste list mới)
(c) Đổi audience / length / tone
(d) Hủy custom, quay về chọn report_type khác
```

User confirm → Stage 4 normalize content theo spec custom; reject → quay lại quiz.

## 6. CP1 — Clarification confirm

User trả lời câu hỏi Stage 3. Agent ghi nhận response → tiếp Stage 4.

Nếu user reply không match option (vd reply tự do, partial answer): hỏi lại 1 lần với câu cụ thể; nếu vẫn không rõ thì áp default + báo user "đã áp default cho câu [N], ghi note để bạn review ở CP2".

## 7. Stage 4 — Normalize (LLM-based)

Đây là stage chính, LLM produce MD theo `FORMAT.md` contract.

### 7.1. Input cho LLM

- Content thô từ Stage 1
- Parse output từ Stage 2
- User clarification từ Stage 3
- `FORMAT.md` (full content) làm reference

### 7.2. LLM tasks

**Task 1: Generate frontmatter**
- Set `report_type` theo user confirm
- Fill required field (ticker, company_name, etc.)
- Set `publish_date` = today nếu input không có
- Set `language: vi`

**Task 2: Apply heading hierarchy**
- H1 = title báo cáo theo report_type
- H2 = section theo rigid structure (stock_pitch / weekly_market / portfolio_review_*) hoặc flex (market_scan / stock_memo / portfolio_plan / custom)
- H3 = subsection khi cần

**Task 3: Translate raw symbols**
- Detect ký hiệu DB raw (`vsi`, `zone: AAA`, `f382`, `period: "2025_4"`, etc.)
- Translate sang ngôn ngữ tự nhiên theo bảng FORMAT.md mục 7
- KHÔNG để raw symbol lọt vào MD final

**Task 4: Locale vi-VN**
- Số: dấu `.` ngăn nghìn, `,` thập phân
- % với dấu `+/-` rõ
- Date: `ngày DD/MM/YYYY` trong prose, `Q[N]/YYYY` hoặc `tháng MM/YYYY` khi nói quý/tháng (xem FORMAT.md mục 6.2)

**Task 5: Citation format**
- Detect mỗi claim có data → assign 1 trong 4 nhóm citation
- Nếu nguồn unclear (đã hỏi ở Stage 3) → dùng default user pick

**Task 6: Chart YAML generation**
- Detect bảng/data có visualizable không (≥3 điểm trend, ≥3 categories, composition)
- Convert sang ` ```chart ` block với syntax đúng
- Suggest type phù hợp (bar/line/pie/etc.)

**Task 7: Section content fulfill**
- Map content input → section theo report_type structure (preset 8 type — xem `FORMAT.md` mục 3.1-3.8)
- Nếu `report_type: custom`, follow `custom_sections` list trong frontmatter (đã build ở Stage 3 mục 5.5)
- Section thiếu nội dung → ghi placeholder `[Cần bổ sung — chưa có data]` (user sẽ thấy ở CP2)
- KHÔNG bịa data, không tự sinh số

### 7.3. Output Stage 4

MD draft. Lưu nội bộ, đi vào CP2.

## 8. CP2 — MD draft review

Show full MD draft cho user. Format show:

```
─── MD DRAFT — Review ──────────────────────────────────

[Show toàn bộ MD]

────────────────────────────────────────────────────────

Self-audit:
- [✓] Frontmatter đầy đủ
- [✓] Heading hierarchy đúng
- [✓] Section count khớp report_type
- [✗] Section 13 thiếu honest steelman (lo ngại còn yếu)
- [✓] Chart YAML 3 block, syntax OK
- [✓] Citation 4 nhóm
- [✓] Locale vi-VN
- [✓] Không lộ ký hiệu raw

Confirm hoặc edit?
(a) Confirm, tiếp Stage 6 chọn brand
(b) Tôi edit trực tiếp MD, paste lại bản sửa
(c) Yêu cầu agent fix issue cụ thể: [user mô tả]
(d) Quay lại Stage 3 clarify thêm
```

User reply → agent xử lý theo option.

### 8.1. Xử lý option

| User chọn | Action |
|---|---|
| (a) Confirm | Stage 5 finalize |
| (b) User edit | User paste MD edit. Agent diff với draft, ghi nhận change, rerun self-audit, present lại nếu có issue mới |
| (c) Fix issue | Agent fix theo yêu cầu cụ thể, present lại |
| (d) Re-clarify | Quay Stage 3 với câu hỏi thêm |

### 8.2. Audit fail nghiêm trọng

Nếu self-audit fail ≥2 mục bắt buộc (vd thiếu section bắt buộc + thiếu disclaimer): agent KHÔNG present MD draft, quay Stage 3 hỏi user fix root cause trước.

## 9. Stage 5 — Finalize MD

Apply user edit (nếu có ở CP2). Rerun full self-audit theo `FORMAT.md` mục 9.

Output: file `.md` chuẩn hóa.

Naming convention output (theo `report_type`):
- `stock_pitch_<TICKER>_<YYYYMMDD>.md`
- `weekly_market_<YYYYMMDD>.md`
- `market_scan_<YYYYMMDD>.md`
- `stock_memo_<TICKER>_<YYYYMMDD>.md`
- `portfolio_plan_<YYYYMMDD>.md`
- `portfolio_review_weekly_<YYYYMMDD>.md`
- `portfolio_review_monthly_<YYYYMM>.md`
- `portfolio_review_quarterly_<YYYY_Q>.md`
- `custom_<slug>_<YYYYMMDD>.md`  (slug từ `custom_purpose` hoặc user-provided)

Trên Claude Desktop, agent xuất nội dung MD trong message để user copy/save thủ công.

## 10. Stage 6 — Brand pre-flight

Hỏi user pick brand template từ whitelist:

```
MD final đã sẵn sàng. Chọn brand template để render binary:

(a) VBSE — navy + đỏ accent + tam giác signature, dùng cho KH VBSE
(b) Finext — dark BG + violet primary + chevron signature, dùng cho KH Finext
(c) Chỉ MD, không render binary

Lưu ý: chỉ có 2 brand sẵn. Brand khác phải build template mới (ngoài scope session này).
Note: nếu pick (a) hoặc (b), bạn cần attach file pptx template tương ứng (`TEMPLATE_VBSE.pptx` hoặc `TEMPLATE_FINEXT.pptx`) vào chat — file pptx không nằm trong project knowledge.
```

## 11. CP3 — Brand confirm + pptx upload check

User pick (a), (b), hoặc (c).

| User chọn | Action |
|---|---|
| (a) VBSE | Check session có `TEMPLATE_VBSE.pptx` chưa → có: Stage 7 render. Chưa: request upload (xem dưới) |
| (b) Finext | Check session có `TEMPLATE_FINEXT.pptx` chưa → có: Stage 7 render. Chưa: request upload |
| (c) Skip | Present MD final, kết thúc workflow |
| Brand khác | Reject theo system_prompt mục 5.4, hỏi lại |

**Pptx upload request flow** (khi user pick brand nhưng chưa attach pptx):

```
Để render binary brand [VBSE/Finext], tôi cần file `TEMPLATE_[BRAND].pptx` (binary template, không nằm trong project knowledge).

(a) Upload file pptx → tôi tiếp tục Stage 7 render
(b) Tôi xuất MD final làm output cuối, bạn dùng tool render bên ngoài tự apply template
(c) Quay lại Stage 6 chọn lại brand khác
```

User upload pptx → agent verify filename match brand đã pick, nếu OK → Stage 7. Nếu mismatch (vd pick VBSE attach FINEXT.pptx), agent flag yêu cầu confirm. Chi tiết rule ở `system_prompt.md` mục 5.7.

## 12. Stage 7 — Render binary

**Prerequisite:** user phải đã attach `TEMPLATE_VBSE.pptx` hoặc `TEMPLATE_FINEXT.pptx` trong session (verified ở CP3, xem mục 11). Nếu chưa có pptx, không vào Stage 7.

TEMPLATE pack runtime:

1. Đọc MD final + file pack `TEMPLATE_X.md` (project knowledge) + binary `TEMPLATE_X.pptx` (session attachment)
2. Scan section heading + chart annotation block trong MD
3. Match semantic với layout có sẵn trong TEMPLATE pack
4. Clone slide layout → fill placeholder từ MD content
5. Tại chart placeholder: build native PowerPoint chart từ YAML block
6. Save thành báo cáo pptx cuối

### 12.1. Output

File `.pptx`. Naming theo MD source + suffix `<brand>`:
- `stock_pitch_<TICKER>_<YYYYMMDD>_<brand>.pptx`
- `weekly_market_<YYYYMMDD>_<brand>.pptx`
- `market_scan_<YYYYMMDD>_<brand>.pptx`
- `stock_memo_<TICKER>_<YYYYMMDD>_<brand>.pptx`
- `portfolio_plan_<YYYYMMDD>_<brand>.pptx`
- `portfolio_review_<period>_<id>_<brand>.pptx` (period = weekly/monthly/quarterly)
- `custom_<slug>_<YYYYMMDD>_<brand>.pptx`

`<brand>` = `vbse` hoặc `finext`.

Trên Claude Desktop, agent xuất file binary qua artifact hoặc attach trong message.

### 12.2. Render error fallback

- Layout không match (section heading không tìm được layout) → ghi note, dùng `BULLET_LIST_SUMMARY` fallback, báo user
- Chart YAML parse fail → render slide không chart, báo user issue ở YAML
- Placeholder không fill được (data thiếu) → leave placeholder dạng `{{NAME}}` trong slide, báo user
- Render crash hoàn toàn → present MD final + báo user lỗi cụ thể

## 13. Re-render (fast path)

User đã có MD final muốn render brand khác:

> "Đổi brand: VBSE → Finext"

Agent skip Stage 1-5, đi thẳng Stage 6 brand pre-flight. MD final giữ nguyên.

Tương tự khi user muốn re-render với binary mới (pptx bị lỗi, muốn render lại): skip về Stage 7.

## 14. Self-audit toàn flow trước present binary

Chạy 7 câu trước khi present binary:

1. MD final pass `FORMAT.md` mục 9 checklist?
2. Mọi clarification có user response, không tự đoán?
3. Brand do user pick từ whitelist (VBSE/Finext)?
4. Binary render: layout placeholder fill hết (regex check `{{[A-Z_]+}}` không còn)?
5. Chart placeholder build native, không còn rectangle giả?
6. Số slide đầu cuối khớp section count trong MD?
7. Format số locale vi-VN đồng nhất trong slide?

Vi phạm câu nào sửa rồi mới present.
