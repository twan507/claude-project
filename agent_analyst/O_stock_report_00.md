# O_stock_report_00 — Render spec cho P_stock_report

## 1. Mục đích & scope

Render spec cho deliverable của pack `P_stock_report` — báo cáo phân tích chuyên sâu 1 cổ phiếu VN niêm yết, 3 depth mode (Quick / Standard / Deep), audience flex (nội bộ / KH), pair compare optional.

Pack quy định: structure rigid 6-7 phần, format MD (source of truth), 2 mode branding (plain / branded optional), K hygiene + citation, audit trail metadata, output naming.

**Output cuối là MD final** trong message (theo deployment Claude Desktop hiện tại). Render binary pptx/docx out of scope (xem README.md mục 8.1).

**Dependencies:**
- `P_stock_report_00..04` — pack process
- `K_agent_db` — data + methodology + K hygiene rules
- `K_sector_framework` — sector lens (reference, không direct render)

## 2. Structure 6-7 phần rigid heading

Heading bắt buộc giữ nguyên (không flex như `P_vbse_strategy`). Sub-section trong phần có thể flex theo depth mode.

### 2.1. Frontmatter metadata (đầu file)

```yaml
---
ticker: VNM
company_name: Công ty Cổ phần Sữa Việt Nam
industry: THUCPHAM
type: SXKD
sub_type: Consumer/Defensive
report_date: 2026-05-30
report_mode: standard    # quick / standard / deep
audience: internal       # internal / client
pair_compare: false      # true nếu pair mode
pair_tickers: []         # [VCB, ACB] nếu pair
horizon: 3-6m
recommendation: Long     # Long / Watch / Avoid
conviction: HIGH         # HIGH / MID / LOW
analyst: [optional]
---
```

### 2.2. Heading structure (cố định)

```markdown
# Báo cáo phân tích cổ phiếu [TICKER] — [Tên ngắn công ty]

## Phần 1 — Khuyến nghị tổng quan

## Phần 2 — Phân tích doanh nghiệp

## Phần 3 — Bối cảnh ngành & vĩ mô
   (Skip cho Quick mode)

## Phần 4 — Phân tích tài chính & định giá

## Phần 5 — Tin tức & Catalyst

## Phần 6 — Bear case & Disconfirming signal

## Phần 7 — Exit triggers
   (Chỉ render cho Long recommendation, skip Watch / Avoid)

## Phụ lục — Audit trail data sources
   (Deep mode đầy đủ; Standard rút gọn; Quick optional)

---

[Disclaimer]
```

**Pair mode:** Title đổi thành "Báo cáo so sánh [TICKER1] vs [TICKER2] [vs TICKER3]". Phần 2/4/5/6/7 render side-by-side hoặc sequential (xem `P_stock_report_03` mục 5).

## 3. Format từng phần

### 3.1. Phần 1 — Khuyến nghị tổng quan

**Block đầu (callout box hoặc table 2 cột):**

```markdown
> **📊 [TICKER] — [Long/Watch/Avoid]** | Conviction **[HIGH/MID/LOW]** | Horizon **[1-3m / 3-6m / 6-12m]**

| Tham số | Giá trị |
|---|---|
| Mã | TICKER |
| Ngành | [Tên ngành đầy đủ] |
| Type phân tích | [SXKD / NH / CK / BH] |
| Giá hiện tại | [X.XXX đ] |
| Vốn hóa thị trường | [X.XXX tỷ VND] |
| ADV 1 tháng | [X tỷ VND/ngày] |
| P/E TTM | [X.X x] |
| P/B | [X.X x] |
```

**Block thesis core:**

```markdown
### Thesis core

[1-2 dòng tóm tắt luận điểm chính]

### Variant Perception
[Nếu có: 2-3 dòng góc nhìn khác consensus + cơ sở định lượng]
[Nếu không có Standard: flag "Consensus-aligned thesis, edge limited"]
[Nếu không có Deep: note auto downgrade conviction]

### Top catalyst (2-3, có timing)

1. **[Catalyst 1]** — kỳ vọng [timing cụ thể]
2. **[Catalyst 2]** — kỳ vọng [timing cụ thể]
3. **[Catalyst 3]** — kỳ vọng [timing cụ thể] (optional cho Deep)

### Bear case chính (1-2 dòng)

[Tóm tắt bear scenario]

### Disconfirming signals chính

1. [Signal X với threshold cụ thể]
2. [Signal Y với threshold cụ thể]
```

**Length:** 10-15 dòng Quick, 15-25 Standard, 25-35 Deep.

### 3.2. Phần 2 — Phân tích doanh nghiệp

**Heading sub-section (theo type, từ `P_stock_report_02` mục 7):**

```markdown
### Mô hình kinh doanh

[1 đoạn business overview + segment mix]

### Earnings drivers

[2-3 đoạn theo type: SXKD / NH / CK / BH]
[Sub-heading optional theo type-specific KPI categories]

### Vị trí chuỗi giá trị
   (MANDATORY cho SXKD Standard+, optional Quick; SKIP cho NH/CK/BH)
   (Áp dụng Porter Value Chain 1985 + Porter 5 Forces 1979 + Smile Curve Stan Shih 1992 + GVC governance Gereffi 2005 + Industry 4.0 CFA Sector Analysis 2020)

#### Chuỗi giá trị ngành
[2-4 dòng: upstream → midstream → downstream map + margin pool note + bottleneck]
(Deep mode: optional ASCII diagram)

#### Chuỗi giá trị doanh nghiệp (Porter Value Chain)

**Primary activities (5):**
- **Nguyên liệu đầu vào (Inbound):** [top NCC + % giá vốn + nguyên liệu key + Tier supplier position]
- **Vận hành sản xuất (Operations):** [công suất + utilization + vị trí cost curve + mức tự động hoá]
- **Phân phối (Outbound):** [kênh owned vs outsourced + logistics integrated]
- **Bán hàng (Marketing & Sales):** [top KH + % doanh thu + B2B/B2C/Export mix + brand position]
- **Dịch vụ (Service):** [after-sales + recurring revenue % nếu có]

**Support activities (4) — Standard+ mandatory:**
- **Hệ thống quản trị:** [HĐQT, ERP, governance]
- **Quản lý nhân lực:** [turnover, training, ESOP]
- **Phát triển công nghệ:** [R&D / Revenue ratio + roadmap digital]
- **Mua sắm chiến lược:** [strategy sourcing + hedging]

**Mức độ tích hợp dọc:** [chuyên 1 mắt xích / tích hợp ngược (backward) / tích hợp xuôi (forward) / tích hợp đầy đủ / tập đoàn đa ngành]

#### Áp lực cạnh tranh (Porter 5 forces)
| Áp lực | Mức | Lý do |
|---|---|---|
| Phụ thuộc nhà cung cấp | Cao/Trung/Thấp | [...] |
| Phụ thuộc khách hàng | Cao/Trung/Thấp | [...] |
| Cạnh tranh nội ngành | Cao/Trung/Thấp | [...] |
| Sản phẩm thay thế | Cao/Trung/Thấp | [...] |
| Rào cản gia nhập | Cao/Trung/Thấp | [...] |

**Quản trị chuỗi cung ứng toàn cầu (GVC governance):** [thị trường (market) / mô-đun (modular) / quan hệ (relational) / phụ thuộc (captive) / tích hợp (hierarchy)]

**Vị trí Tier trong chuỗi cung ứng:** [Tier 1 thương hiệu riêng / Tier 1 mô-đun / Tier 2 phụ thuộc / Tier 3 hàng hoá]

#### Smile Curve — Vị trí capture giá trị (Stan Shih 1992)
**Vị trí trên Smile Curve:** [đáy smile (gia công thuần) / giữa smile (đang leo) / đỉnh smile (thương hiệu + dịch vụ)]

**Chiến lược leo smile:** [stuck (đang kẹt) / đang leo (X-Y năm) / đỉnh smile được bảo vệ tốt]

**Lý do:** [evidence từ gross margin trend + R&D ratio + brand recognition + recurring revenue]

#### Industry 4.0 / Dấu chân số (CFA Sector Analysis 2020)
**Tổng quan độ trưởng thành:** [Leader (dẫn đầu) / Par (ngang ngành) / Laggard (tụt hậu)]

| Khía cạnh | Verdict | Note |
|---|---|---|
| Tạo digital footprint từ customer/production | Strong/Trung/Weak | [...] |
| Tự động hoá sản xuất | Strong/Trung/Weak | [...] |
| Mass customisation capability | Strong/Trung/Weak | [...] |
| IoT deployment | Strong/Trung/Weak | [...] |
| AI/ML deployment | Strong/Trung/Weak | [...] |
| Mô hình kinh doanh (B2C/B2B/C2B) | Strong/Trung/Weak | [...] |
| Tốc độ feedback loop với khách hàng | Strong/Trung/Weak | [...] |

#### Kết luận vị trí chuỗi giá trị (tích hợp)
[5-7 dòng: firm captures margin ở mắt xích nào + vị trí Smile + GVC governance + Tier position + Industry 4.0 readiness + expose risk ở đâu + pricing power verdict tổng hợp (mạnh/trung bình/yếu)]

### Vị thế cạnh tranh

[1 đoạn market share, moat, structural advantage — kết quả của vị trí chuỗi giá trị ở trên]

### Cycle position / Type-specific concern
   (Cho SXKD: classify Value Play / Value Trap / Growth at Premium / Cycle Top)
   (Cho NH/CK/BH: type-specific concern checklist)
```

**K hygiene cho Vị trí chuỗi giá trị:**
- Audience nội bộ: dùng đủ tên framework gốc — "Porter 5 forces", "supplier power Cao/Trung/Thấp", "vertical integration backward/forward/full", "Smile Curve position", "GVC governance captive/modular", "Tier 2 supplier", "Industry 4.0 Leader/Laggard", tên KH/NCC cụ thể
- Audience KH:
  - Dịch jargon framework gốc sang tiếng Việt tự nhiên:
    - "Porter 5 forces" → "5 nhóm áp lực cạnh tranh"
    - "vertical integration full" → "tích hợp dọc đầy đủ chuỗi giá trị"
    - "backward / forward integration" → "tích hợp ngược / xuôi chuỗi giá trị"
    - "cost curve position" → "vị trí trên đường cong chi phí ngành"
    - "Smile Curve position" → "vị trí trên đường cong giá trị (giá trị tập trung ở 2 đầu chuỗi — R&D + thương hiệu/dịch vụ)"
    - "GVC governance" → "mô hình quản trị chuỗi cung ứng toàn cầu"
    - "Tier 2 captive supplier" → "nhà cung cấp cấp 2 phụ thuộc nặng vào khách hàng dẫn dắt chuỗi"
    - "Industry 4.0 Laggard" → "doanh nghiệp tụt hậu về chuyển đổi số / Cách mạng Công nghiệp 4.0"
  - Ẩn tên KH/NCC nhạy cảm thương mại — dùng generic "1 tập đoàn FDI lớn ngành [X]", "1 chuỗi bán lẻ top 3 VN" — giữ % concentration
  - BỎ tên framework academic (Stan Shih, Gereffi, Porter, CFA Sector Analysis) — chỉ giữ concept giải thích bằng tiếng Việt thông thường

**Bảng KPIs đặc thù theo type (mandatory cho Standard+):**

Render bảng từ template `P_stock_report_02` mục 7:

```markdown
| KPI | Q[N-7] | Q[N-6] | ... | Q[N] (latest) | Trend |
|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ↑/↓/→ |
```

Số quý hiển thị:
- Quick: 4 quý gần nhất
- Standard: 8 quý gần nhất
- Deep: 12 quý gần nhất

### 3.3. Phần 3 — Bối cảnh ngành & vĩ mô

**Skip cho Quick mode.**

**Heading sub-section:**

```markdown
### Cấu trúc ngành

[1 đoạn từ K_sector_framework mục 5.X: DD/MP highlights]

### Xu hướng dài hạn (3-5 năm)

[1 đoạn structural influences SI]

### Bối cảnh vĩ mô liên quan

[1 đoạn macro variables relevant: lãi suất / commodity / FX tuỳ ngành]

### ESG context
   (Optional cho Standard, đầy đủ cho Deep)

[1 đoạn ESG hotspots + status firm + controversy nếu có]
```

**Length:** 0.5 trang Standard, 1-1.5 trang Deep.

### 3.4. Phần 4 — Phân tích tài chính & định giá

**Heading sub-section:**

```markdown
### Lịch sử tài chính 3-5 năm

[Bảng historical 3-5Y annual + 8-12Q quarterly]

### Phân tích DuPont (Deep mode)

[ROE decomposition: Net margin × Asset turnover × Equity multiplier]

### Định giá hiện tại

[Bảng valuation: P/E, P/B, EV/EBITDA hiện tại + phân vị 3Y, 5Y + so industry median]

### Peer compare
   (Standard: 3 peer; Deep: 5 peer)

[Bảng peer compare 8-10 cột]
[1-2 đoạn note ưu/khuyết so peer]

### Forensic flags (từ BCTC PDF)

[Bảng 15-point checklist với status ✓/⚠/✗ + flag chính]
[1 đoạn forensic verdict]

### Macro sensitivity (Deep only)

[Bảng sensitivity NIM/EPS/margin sensitivity to key variables]

### Định giá conclusion

[1-2 đoạn: fair / expensive / cheap so historical + peer]
[Implied upside/downside range Bear/Base/Bull — Standard+ only]
```

**Bảng historical financials format (audience nội bộ):**

```markdown
| Năm | Revenue (tỷ) | YoY% | Gross M | EBIT M | Net M | ROE | Net debt/EBITDA | FCF/Revenue |
|---|---|---|---|---|---|---|---|---|
| 2021 | ... | ... | ... | ... | ... | ... | ... | ... |
| 2022 | ... | ... | ... | ... | ... | ... | ... | ... |
| 2023 | ... | ... | ... | ... | ... | ... | ... | ... |
| 2024 | ... | ... | ... | ... | ... | ... | ... | ... |
| 2025 | ... | ... | ... | ... | ... | ... | ... | ... |
| TTM | ... | ... | ... | ... | ... | ... | ... | ... |
```

**Audience KH:** Có thể rút gọn xuống 4-5 cột (Revenue, YoY, Gross M, Net M, ROE) — bỏ leverage + FCF detail.

### 3.5. Phần 5 — Tin tức & Catalyst

```markdown
### Tin tức recent (30-90 ngày)

[3-5 tin material với ngày, headline, impact 1 dòng, nguồn]

| Ngày | Tin | Tác động | Nguồn |
|---|---|---|---|
| ... | ... | ... | ... |

### Sell-side consensus
   (Standard+ if available)

[1 đoạn về view của broker VN / foreign nếu có]

### Catalyst pipeline (2-4)

1. **[Catalyst 1]**
   - Mô tả: [...]
   - Timing: [...]
   - Tác động kỳ vọng: [tăng/giảm conviction]

2. **[Catalyst 2]**
   ...

### Earnings calendar

- Next BCTC: [period] dự kiến công bố [date]
- Consensus EPS (nếu có): [...]
- AGM upcoming: [date nếu có]

### Macro catalyst upcoming (Deep only)

[1-2 macro event relevant: Fed meeting, OPEC, Circular SBV, etc.]
```

### 3.6. Phần 6 — Bear case & Disconfirming signal

```markdown
### Bear case steelmanned

[2-3 đoạn bear scenario — KHÔNG soft-pedal]

### Bear target price (range)
   (Standard+ only)

| Scenario | Target | Downside vs current |
|---|---|---|
| Bear | [...] | -X% |
| Base | [...] | +Y% |
| Bull | [...] | +Z% |

[Nếu Bear cumulative < current → red flag warning callout]

### Disconfirming signals (cái gì khiến thesis sai)

Top signal cần theo dõi:

1. **[Signal X]** — threshold: [số/sự kiện cụ thể]
   - Cơ chế: [tại sao signal này material]
   - Action nếu materialize: [downgrade conviction / exit / reduce]

2. **[Signal Y]** — threshold: [...]
   ...

3. **[Signal Z]** — threshold: [...] (optional Deep)
```

**Audience KH:** Section "Disconfirming" đổi tên "Tín hiệu cần theo dõi" — wording mềm, KHÔNG ghi "exit/reduce action" mà chỉ "xem xét lại quan điểm".

### 3.7. Phần 7 — Exit triggers (CHỈ Long)

**Skip cho Watch / Avoid.**

```markdown
### Take-profit triggers

- **TP1** — Price reaches Base target [X.XXX đ]: reduce 50%
- **TP2** — Price reaches Bull target [X.XXX đ]: reduce 80%
- **Total exit** — Price > Bull × 1.1 [X.XXX đ]

### Stop-loss triggers

- **SL technical**: price break support tuần [X.XXX đ] hoặc Fibonacci 61.8% retracement
- **SL fundamental**: 1+ disconfirming signal materialized (xem Phần 6)
- **Combined SL**: cả 2 trigger → hard exit

### Time-stop

- Horizon expiry: [date kết thúc horizon]
- Review checkpoint: [BCTC quý tới] hoặc [ngày AGM]

### Catalyst trigger

- Nếu **catalyst chính fail** (vd earnings miss consensus material) → exit
- Nếu **catalyst chính materialize**: review book profit hoặc hold tiếp
```

**Audience KH:** Section này render rất khác — bỏ TP1/TP2/SL số cụ thể (command):

```markdown
### Tín hiệu xem xét lại quan điểm

Khuyến nghị anh/chị theo dõi các tín hiệu sau để xem xét điều chỉnh quan điểm:

1. **Diễn biến giá** — nếu giá biến động đáng kể về phía bất lợi (vd giảm 10-15% từ vùng giá hiện tại không có catalyst rõ), nên review lại
2. **Tín hiệu cơ bản** — [list disconfirming signals]
3. **Catalyst chính** — nếu các catalyst kỳ vọng không materialize trong horizon, nên review

Quyết định mua/bán/giữ phụ thuộc tổng thể danh mục, khẩu vị rủi ro của anh/chị và tư vấn từ chuyên viên.
```

### 3.8. Phụ lục — Audit trail (Deep mode đầy đủ)

```markdown
## Phụ lục — Audit trail data sources

### DB collections sử dụng

- `stock_info` (sub-step 1a)
- `stock_finstats` (sub-step 1b)
- `stock_snapshot` (sub-step 1b, 1c, 1d, 1e)
- `stock_recent` (sub-step 1c)
- `news_today_feed`, `news_history_feed` (sub-step 1g)
- `industry_finstats` (sub-step 1l peer compare)
- `history_stock` (sub-step 1m ADV)
- `other_data` (sub-step 1k macro)

### Web search queries thực hiện

1. "[Query 1]"
2. "[Query 2]"
...

### File user upload

- BCTC [period] — filename, ngày upload
- BCTN [năm] (nếu có) — ...
- Báo cáo phân tích cũ (nếu có) — ...

### External sources cite

- Finext.vn link [list]
- Sell-side reports (nếu có) — [broker name + ngày]
- Macro sources EN (nếu cần) — [Bloomberg / Reuters / Fed.gov / OPEC.org]

### Data freshness

- Snapshot date DB: [YYYY-MM-DD]
- BCTC latest period: [period]
- News window: [N ngày]
- Macro data date: [YYYY-MM-DD]

### Sub-steps executed

✓ 1a Stock info + Type classification
✓ 1b FA data DB
✓ 1c Dòng tiền + Technical zone
...
✗ 1l Peer compare (skip Quick mode)
...

### Flags raised + caveats

1. ⚠ [Flag 1 với context]
2. ⚠ [Flag 2 với context]

### Conviction adjustment

- Initial assessment: HIGH
- Final conviction: MID
- Reason for downgrade: [vd: Variant Perception thiếu ở Deep mode]
```

**Standard mode rút gọn:** Chỉ Data freshness + Sub-steps executed + Top 3 flag.

**Quick mode:** Optional, có thể skip.

## 4. 2 mode branding

### 4.1. Mode "plain" (default)

- Header đơn giản: `# Báo cáo phân tích cổ phiếu [TICKER]`
- Không logo, không branding visual
- Đầy đủ disclaimer cuối

### 4.2. Mode "branded" (optional)

User cung cấp branding info ở pre-flight:
- Brand name (vd "Finext Research", "[Tên broker]")
- Logo link (nếu có)
- Color scheme (nếu có)

Render:
- Header có brand name + ngày + analyst name (nếu có)
- Footer có brand name + disclaimer chuẩn cho audience tương ứng

**Branding KHÔNG ảnh hưởng content** — chỉ shell visual. Content vẫn theo `P_stock_report` workflow.

Render binary (pptx/docx branded) **out of scope** project hiện tại — nếu user cần, dùng tool render bên ngoài.

## 5. K hygiene — bảng dịch áp dụng riêng cho pack này

Reference `K_agent_db_00` mục 5. Thêm dịch riêng pack này:

| Internal taxonomy | Audience nội bộ analyst | Audience KH |
|---|---|---|
| Recommendation: Long | Long (giữ raw) | "Quan điểm tích cực" |
| Recommendation: Watch | Watch (giữ raw) | "Tiếp tục theo dõi" |
| Recommendation: Avoid | Avoid (giữ raw) | "Cẩn trọng / Chưa khuyến nghị tham gia" |
| Conviction: HIGH | HIGH (giữ raw) | "Quan điểm tích cực mạnh" |
| Conviction: MID | MID (giữ raw) | "Quan điểm tích cực trung bình" |
| Conviction: LOW | LOW (giữ raw) | "Quan điểm thận trọng" |
| Variant Perception | Variant Perception (giữ raw) | "Góc nhìn khác với quan điểm chung thị trường" hoặc bỏ phần này |
| Bucket entry 1/2/3 | Giữ raw nếu nội bộ | KHÔNG render — bỏ |
| Kịch bản A/B/C/E1/E2/E3 | Mô tả trực tiếp (xem `K_agent_db_00` 5.3) | Mô tả trực tiếp |
| HIGH/MID/LOW impact (news) | Mô tả trực tiếp | "Tác động mạnh / vừa / nhẹ" |
| Disconfirming signal | Disconfirming signal hoặc "Tín hiệu phản chứng" | "Tín hiệu cần theo dõi để xem xét lại" |
| Forensic flag | Giữ raw | "Vấn đề cần lưu ý từ BCTC" |
| Value Play / Value Trap / Growth at Premium / Cycle Top | Giữ raw (analyst lingo) | "Định giá hấp dẫn có cơ sở / Định giá hấp dẫn nhưng có rủi ro / Tăng trưởng cao có cơ sở / Định giá đỉnh chu kỳ" |
| TP1 / TP2 / SL / Time-stop | Giữ raw | KHÔNG render — chỉ "Tín hiệu xem xét lại quan điểm" |

## 6. Citation — 4 nhóm cho pack này

Reference system prompt mục 5.2 (source attribution) + `K_agent_db_00` mục 10 (output contract). Apply cho pack:

| Loại data | Format | Ví dụ |
|---|---|---|
| Data từ `agent_db` | `(nguồn: Tổng hợp)` | "Revenue VNM 2025 đạt 60.200 tỷ VND (nguồn: Tổng hợp)" |
| Tin / báo cáo finext.vn | Markdown link | `[Finext, 18/04/2026](https://finext.vn/news/<slug>)` |
| BCTC PDF user upload | Tên tài liệu + section/page | "BCTC VNM Q4/2025 soát xét, Thuyết minh 27, trang 45" |
| Tin web external (VN news, sell-side) | Markdown link | `[Vietstock, 12/05/2026](https://vietstock.vn/...)` |
| Macro EN sources | Markdown link tiếng Anh | `[Fed FOMC Minutes May 2026](https://federalreserve.gov/...)` |

## 7. Render rules

### 7.1. Length budget

| Mode | Pages target | Words approx | Vượt → action |
|---|---|---|---|
| Quick | 1-2 | 500-1.000 | Flag self-audit, cô đọng |
| Standard | 3-5 | 1.500-2.500 | Flag self-audit, cô đọng |
| Deep | 5-10 | 2.500-5.000 | Flag self-audit, split sub-section |
| Pair (Standard) | 5-7 | 2.500-3.500 | Flag self-audit |
| Pair (Deep) | 8-12 | 4.000-6.000 | Flag self-audit, optional split sang 2 file |

### 7.2. Heading hierarchy

- H1: Tên báo cáo
- H2: Phần 1-7 + Phụ lục
- H3: Sub-section trong từng phần
- H4: Optional cho sub-sub-section (Deep mode đào sâu)

KHÔNG dùng H5/H6.

### 7.3. Bảng

- Format MD chuẩn (pipe `|`)
- Mỗi bảng có header + alignment
- Quy đổi đơn vị: BCTC sang **tỷ VND**, ratios sang **%**, giá sang **đ** (per share) hoặc **k** (33.000 đ = 33k)
- Numerical separator: dấu chấm ngăn nghìn, dấu phẩy thập phân (vi-VN locale): `60.200 tỷ`, `15,5%`

### 7.4. Callout / Highlight

Dùng cho:
- Big warning (mã suspended, penny, conglomerate disclaimer) — prepend đầu báo cáo
- Recommendation summary đầu Phần 1
- Bear target < current red flag

Format MD:

```markdown
> ⚠️ **CẢNH BÁO**: [content]

> 📊 **Khuyến nghị**: [content]

> 🔴 **Red flag**: [content]
```

### 7.5. Chart annotation YAML (optional)

Pack KHÔNG yêu cầu chart bắt buộc. Nếu user request render chart (vd peer compare visual), agent có thể nhúng YAML chart annotation cho tool render ngoài:

```yaml
---chart---
type: bar
title: Peer compare ROE TTM
x: [VNM, MSN, KDC, peer4, peer5]
y: [22.5, 8.2, 12.4, 15.1, 18.9]
unit: '%'
note: ROE TTM trailing 4 quarters
---/chart---
```

Tool render bên ngoài có thể parse YAML này. Pack không tự render visual.

## 8. Disclaimer template

### 8.1. Disclaimer nội bộ (audience internal)

```markdown
---

**Disclaimer**

Báo cáo này chỉ dùng nội bộ trong tổ chức. Số liệu lấy từ MongoDB `agent_db` 
(snapshot date [YYYY-MM-DD]) và các nguồn web đã ghi rõ. BCTC reference: [period].

Phân tích phản ánh quan điểm tại thời điểm publish, không phải dự báo chắc chắn. 
Diễn biến thực tế có thể khác. Quyết định đầu tư cuối cùng do người đọc cân nhắc 
trên tổng thể danh mục và khẩu vị rủi ro cá nhân.

Không bảo đảm lợi nhuận. Không thay thế tư vấn pháp lý / thuế / kế toán cá nhân.
```

### 8.2. Disclaimer cho KH (audience client)

```markdown
---

**Tuyên bố miễn trừ trách nhiệm**

Báo cáo này được cung cấp cho mục đích tham khảo, không phải tư vấn đầu tư cá nhân. 
Số liệu và phân tích dựa trên thông tin công khai sẵn có tại thời điểm phát hành 
([YYYY-MM-DD]). Báo cáo tài chính tham chiếu kỳ [period].

Quan điểm trong báo cáo phản ánh nhận định của bộ phận phân tích tại thời điểm 
publish và có thể thay đổi theo diễn biến thị trường mà không cần báo trước.

Quyết định đầu tư cần dựa trên đánh giá đầy đủ của quý anh/chị về tình hình tài chính, 
mục tiêu đầu tư, khẩu vị rủi ro, và khung thời gian phù hợp. Quá khứ không bảo đảm 
kết quả tương lai. Không có cam kết về lợi nhuận.

Để biết thêm chi tiết hoặc thảo luận specific case, vui lòng liên hệ với chuyên viên 
tư vấn được cấp phép.
```

### 8.3. Forward-looking statement (Deep mode + audience client)

Append sau disclaimer cho Deep + client:

```markdown
---

**Forward-looking statement**

Quan điểm và kỳ vọng trong báo cáo này là "forward-looking" — phụ thuộc các giả định 
về tương lai có thể không thành hiện thực. Các yếu tố làm sai lệch kỳ vọng bao gồm 
(nhưng không giới hạn): biến động lãi suất, tỷ giá, giá hàng hoá, chính sách vĩ mô, 
sự kiện địa chính trị, dịch bệnh, thiên tai, và các yếu tố bất ngờ khác. 

Vui lòng tham khảo phần "Tín hiệu cần theo dõi" để hiểu các tín hiệu sẽ làm quan điểm 
trong báo cáo cần được xem xét lại.
```

## 9. Output naming convention

File output naming theo `P_stock_report_04` mục 4.6:

```
Single mode:
stock_report_<TICKER>_<YYYYMMDD>_<mode>.md

Pair mode:
stock_report_<TICKER1>vs<TICKER2>_<YYYYMMDD>_pair_<mode>.md
stock_report_<TICKER1>vs<TICKER2>vs<TICKER3>_<YYYYMMDD>_pair_<mode>.md
```

Ví dụ:
```
stock_report_VNM_20260530_standard.md
stock_report_HPG_20260530_deep.md
stock_report_VCBvsACB_20260530_pair_standard.md
```

## 10. Self-checklist trước khi render finalized output

Agent check trước Stage 3 finalize:

- [ ] Frontmatter metadata đầy đủ
- [ ] Heading 6-7 phần đầy đủ + đúng order
- [ ] Phần 3 skip cho Quick mode (hoặc render cho Standard+)
- [ ] Phần 7 chỉ render cho Long (skip Watch/Avoid)
- [ ] Bảng quy đổi đơn vị + locale vi-VN
- [ ] K hygiene đầy đủ — không lộ raw + dịch taxonomy
- [ ] Citation 4 nhóm đầy đủ cho claim định lượng
- [ ] Wording audience-aware (nội bộ raw, KH mềm)
- [ ] Disclaimer cuối báo cáo (match audience)
- [ ] Forward-looking statement nếu Deep + client
- [ ] Audit trail metadata (Deep đầy đủ, Standard rút gọn)
- [ ] File naming đúng convention
- [ ] Length trong budget mode

## 11. Cross-reference với pack khác

| Pack | Quan hệ với O_stock_report |
|---|---|
| `P_stock_report_00..04` | Source content — render spec này consume content |
| `K_agent_db_00` mục 5 | K hygiene rules — apply cho mọi render |
| `K_agent_db_00` mục 6 | Unit conversion rules — apply cho bảng số liệu |
| System prompt mục 5.2 | Source attribution — mọi claim định lượng có nguồn truy được |
