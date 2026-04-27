# Audit Plan — Claude Project (analysis_agent / template_agent / db_agent)

**Ngày lập:** 2026-04-27
**Người lập:** AI session (Claude) + user xác nhận scope
**Trạng thái:** Draft, chờ user duyệt trước khi execute

---

## 1. Mục đích

Audit comprehensive 3 agent của dự án để phát hiện **issue còn sót** ở 4 dimension:

- **A. Consistency** — README ↔ filesystem ↔ system_prompt ↔ KERNEL_SKELETON ↔ INDEX
- **B. Methodology integrity** — gate / constraint / nguyên tắc bất biến có enforce nhất quán không
- **C. Cross-agent sync** — `K_agent_db_*` (analysis_agent) ↔ `agent_db_*` (db_agent)
- **D. Legacy / dead-code** — `[LEGACY]`, `/mnt/user-data/outputs/`, `present_files`, "Guide render docx/pptx", section obsolete

Output là **1 audit report duy nhất** (no fix). User đọc → tự quyết priority fix sau.

---

## 2. Scope & boundary

### 2.1. Trong scope

| # | Đối tượng | Phạm vi |
|---|---|---|
| 1 | `analysis_agent/` | 29 file `.md` (KERNEL + 6 K + 9 O + 12 P + 1 system_prompt) |
| 2 | `template_agent/` | 6 file `.md` + 2 `.pptx` (catalog only, không audit binary) |
| 3 | `db_agent/` | 7 file `.md` |
| 4 | `README.md` (root) | Đối chiếu với filesystem thực tế |
| 5 | `CLAUDE.md` (root) | Đọc tham khảo, không audit content |

### 2.2. Out of scope (audit lần này không cover)

- Quality của methodology (vd: framework PTCB 4 type doanh nghiệp có đúng học thuật không) — đây là chuyên môn finance, vượt khả năng audit
- Render quality của `.pptx` template (visual design)
- Performance / token cost của workflow runtime
- Test thực tế agent trong Claude Desktop (chỉ audit static doc/prompt)
- Quality của tiếng Việt (ngữ pháp, văn phong)

### 2.3. Depth — L2 Semantic Check

| Tier | Mô tả | Áp dụng cho audit này |
|---|---|---|
| L1 — Surface scan | grep / diff tự động, file count, dead ref pattern | ✅ Bao gồm |
| L2 — Semantic check | Đọc kỹ master file (`*_00`, `KERNEL_SKELETON`, `INDEX`, `system_prompt`) đối chiếu file con; check 9 report_type ↔ P/O pack thực tế; check 6 nguyên tắc bất biến có enforce ở stage tương ứng | ✅ Bao gồm |
| L3 — Deep methodology | Đọc TOÀN BỘ file con, check methodology end-to-end | ❌ Không |

---

## 3. Convention báo cáo

### 3.1. Severity 4 mức

| Severity | Định nghĩa | Ví dụ |
|---|---|---|
| **Critical** | Phá runtime hoặc gây output sai cho audience cuối (KH) | Gate methodology bị bypass; K hygiene leak token raw vào output KH; brand whitelist không enforce |
| **Major** | Inconsistency thấy rõ giữa doc & implementation, gây nhầm lẫn cho dev/AI session sau | README số file sai; manifest stale (KERNEL/INDEX miss file đã có); cross-reference broken (file A ref file B nhưng B đã đổi tên/xoá) |
| **Minor** | Cleanup debt không ảnh hưởng runtime | `[LEGACY]` markers còn sót; ref `/mnt/user-data/outputs/` từ deployment cũ; `present_files` tool ref |
| **Nit** | Cosmetic, format không đồng nhất, typo, naming convention drift nhỏ | Số / locale chưa convert vi-VN trong 1 ví dụ; heading cấp lệch |

### 3.2. Format mỗi finding

```
### F-NN [Severity] — Title ngắn gọn

**What:** Quan sát thực tế. Bắt buộc có file:line (vd: analysis_agent/P_invest_memo_07.md:124).
**Why it matters:** 1 câu giải thích tác hại / risk.
**Suggested fix:** 1-2 câu gợi ý hướng xử lý. Không phải mệnh lệnh.
```

### 3.3. "Confirm intent" — issue thực ra là design decision

Một số issue có thể là **design decision có chủ đích** đã được README mục 8 hoặc 5.3 explicit accept (vd: duplicate `K_agent_db_*` ↔ `agent_db_*`). Khi audit gặp pattern kiểu này:

- KHÔNG flag là Critical/Major/Minor/Nit
- Flag dạng **`[Confirm Intent]`** — describe quan sát + ref README mục đã chấp nhận → user xác nhận intent còn đúng không

### 3.4. Numbering finding

Dùng `F-NN` liên tục từ 01, không reset theo dimension. Ở report cuối có 1 bảng tổng hợp index theo dimension và severity.

---

## 4. Phương pháp audit — chia 4 phase

### Phase A — Consistency audit

#### A.1. README ↔ filesystem

Đối chiếu README mục 1.1 (số file) + 2.1 (file upload list) với filesystem:
- Đếm file thực tế trong từng folder
- Liệt kê file README mention nhưng không tồn tại
- Liệt kê file tồn tại nhưng README không mention

**Sơ bộ đã thấy:** README claim `analysis_agent` có 25 file, thực tế 29. **→ F-01 Major candidate.**

#### A.2. KERNEL_SKELETON ↔ analysis_agent files

- Mọi file `K_*`, `P_*`, `O_*` trong folder phải có entry trong KERNEL_SKELETON
- Mọi entry trong KERNEL_SKELETON phải có file thực tế
- Trigger activation cho từng pack có present + đúng không

#### A.3. INDEX (template_agent) ↔ FORMAT/WORKFLOW/TEMPLATE

- INDEX.md mục manifest có khớp 6 file `.md` thực tế không
- 9 report_type list ở INDEX có khớp 9 report_type ở FORMAT mục 3 không
- Brand whitelist 2 (VBSE/Finext) có nhất quán giữa INDEX, system_prompt, WORKFLOW không

#### A.4. db_agent agent_db_00 manifest ↔ files

- `agent_db_00.md` mục manifest list `_01` đến `_05` có đầy đủ + đúng mục đích không

#### A.5. Cross-reference giữa các file

Dùng grep tìm pattern `mục N`, `[file_name]`, `Tier N`, `Stage N` — verify pointer còn đúng:
- File ref file khác có tồn tại file đó không
- "Mục 5" trong file A có thực sự là mục 5 trong file đích không (spot check 10-15 ref random)
- File ref tier/stage có khớp tier/stage đã định nghĩa không

#### A.6. system_prompt ↔ pack files

- Rule mention pack/file cụ thể ở system_prompt có exist + behavior khớp
- Mention "mục 2", "mục 5.4" trong system_prompt có đúng số mục không

---

### Phase B — Methodology integrity audit

#### B.1. 6 nguyên tắc bất biến (P_invest_memo)

README mục 3.3 list 6 nguyên tắc. Verify từng nguyên tắc được enforce ở stage tương ứng:

| Nguyên tắc | Expected enforce ở | Check |
|---|---|---|
| 1. Không skip variant perception | P_invest_memo_06 hoặc 07 | Có gate? Flex+downgrade hay strict reject? |
| 2. Không vào position nếu chưa có exit trigger | P_invest_memo_07 / 08 / 09 | Có check exit trigger trước position open? |
| 3. Không size > 5% ADV / phiên | P_invest_memo_08 (portfolio) | Có constraint ADV? Computation đúng? |
| 4. Bear case steelman trước long | P_invest_memo_06/07 | Có gate bear case? Honest 1-2 lo ngại còn yếu? |
| 5. Dòng tiền dương + catalyst tiêu cực → loại | P_invest_memo_01 hoặc 02 | Có rule này filter không? |
| 6. Mỗi giai đoạn kết bằng checkpoint | All P_invest_memo files | Mọi tier có CP định nghĩa rõ? Có check user confirm? |

#### B.2. Triết lý flex+downgrade nhất quán

README mục 3.6 + 8.2 nói triết lý flex+downgrade thay strict reject áp dụng xuyên suốt P_invest_memo + P_stock_pitch. Audit:
- `P_invest_memo_07` (Gate 1, Gate 2) define flex+downgrade thế nào → so với `P_stock_pitch_00` Stage 4 (Steelman Bear) có cùng triết lý không
- Có chỗ nào còn dùng strict reject không (vi phạm triết lý)
- "Downgrade size 30-50% / 50-70%" có precise number nhất quán không

#### B.3. K hygiene coverage

`K_agent_db_00` mục 5 (analysis_agent) và `agent_db_00` mục 5 (db_agent) có bảng dịch 3 nhóm raw token. Audit:
- Grep TOÀN BỘ pack P/O của analysis_agent tìm raw token (`vsi`, `day_score`, `zone:`, `f382`, `*_pct`, `*_trend`, "Pitfall F", "Kịch bản A-G", "Value Trap"...)
- Đối chiếu xem mỗi token gặp có nằm trong bảng dịch không
- Token nào chưa có trong bảng → Major finding (K hygiene risk)
- File P/O nào dùng raw token mà không apply translation rule → Critical (output KH có thể leak)

#### B.4. Constraint audience KH (P_weekly_market + P_stock_pitch)

README mục 7.6 list 5 constraint. Verify từng:
- Không dùng `*_trend` → grep `trend` trong P_weekly_market_00 + P_stock_pitch_00 + O_weekly_market_00 + O_stock_pitch_00
- Không command "mua/bán/giảm tỷ trọng" → check guidance wording
- Không xác suất % cho kịch bản → check section kịch bản
- Watchlist không kèm level giá → check section watchlist
- Honest steelman 1-2 lo ngại còn yếu → check Stage Bear

#### B.5. 9 report_type FORMAT ↔ P/O pack

README mục 4.3 list 9 report_type ở `FORMAT.md`. Mỗi type có section count + length expectation. Audit:
- `stock_pitch` 13-16 mục ↔ `P_stock_pitch_00` flex 13-16 mục có khớp section structure không
- `weekly_market` 12 phần ↔ `P_weekly_market_00` 12 phần ↔ `O_weekly_market_00` ouput spec — 3 nguồn có nhất quán heading hierarchy không
- `stock_memo` 3-7 conviction tier ↔ `P_invest_memo_07` (Tier 5C memo 7 phần) — có khớp không
- `portfolio_plan` ↔ `P_invest_memo_08`
- `portfolio_review_*` ↔ `P_invest_memo_09` (4 review cycles)
- `market_scan` — có pack tương ứng trong analysis_agent không, hay là orphan?

#### B.6. Workflow reachability (template_agent)

`WORKFLOW.md` 7 stage + 3 checkpoint. Verify:
- Mọi stage có defined entry + exit
- Skip-normalize fast path (Stage 1.5) — 6 signals và threshold ≥4 có precise không
- Custom quiz (Stage 3) 7 câu chia 2 turn — có liệt kê cụ thể không
- CP routing logic (CP1/CP2/CP3) — có abort path defined không

---

### Phase C — Cross-agent sync audit

#### C.1. Diff `K_agent_db_*` ↔ `agent_db_*`

README mục 5.3 claim 99% identical, khác biệt:
- File name prefix
- Internal cross-reference (`agent_db_XX` vs `K_agent_db_XX`)
- Reference đến system_prompt section number
- 1 vài legacy comment ("Rule 6") trong K_agent_db_04

Audit method:
- Diff từng cặp file (`_00` ↔ `_00`, ..., `_05` ↔ `_05`)
- Phân loại từng diff:
  - **Expected** (file name ref, system_prompt ref) → ignore
  - **Drift** (methodology / data / threshold khác nhau) → Major finding
  - **Forgotten port** (1 nguồn update nhưng nguồn kia chưa) → Major finding

#### C.2. Convention sync

- Bảng dịch 3 nhóm (mục 5 của 2 file `_00`) có giống nhau không
- Citation 4 nhóm có giống nhau không
- Schema 23 collection ở `_01` có giống nhau không

---

### Phase D — Legacy / dead-code audit

#### D.1. Pattern dead-code rõ ràng (grep-based)

Tìm các pattern sau trên toàn project:

| Pattern | Lý do flag | Severity expect |
|---|---|---|
| `/mnt/user-data/outputs/` | Path Linux từ deployment cũ Claude Code/Skill | Minor |
| `present_files` | Tool không tồn tại trên Claude Desktop | Minor |
| `[LEGACY]` | Marker đã admit ở README mục 8.1 | Minor |
| "Guide render docx/pptx" / "Guide render pptx" | Section obsolete sau rev 6 (MD final là output cuối) | Minor |
| `output_format: pdf|docx|xlsx` (frontmatter) | Format cũ không còn dùng | Minor |
| Reference đến tool `artifacts` cũ | Nếu có | Minor |

#### D.2. Section obsolete bởi rev mới

- Tìm marker `rev N` trong file — có chỗ nào nói "rev 5" hay "rev cũ" mà rev 6 đã supersede không
- Section "Phụ lục" / "Appendix" / "Old guide" — có ai mention nhưng không còn link đến không

#### D.3. Orphan file / orphan section

- File trong folder không được mention bất kỳ đâu trong system_prompt hoặc master file → orphan
- Section trong file không được ref bất kỳ đâu → potential orphan

---

## 5. Deliverable

### 5.1. File output

`docs/audit/2026-04-27-audit-report.md`

### 5.2. Cấu trúc report

```
# Audit Report — Claude Project — 2026-04-27

## 0. Executive summary
- Tổng số finding theo severity (Critical: N / Major: N / Minor: N / Nit: N / Confirm Intent: N)
- Top 3 findings cần ưu tiên xử lý
- Health overall của 3 agent (1-2 câu mỗi agent)

## 1. Index findings theo dimension
Bảng F-NN | Severity | Dimension (A/B/C/D) | Title | File:line

## 2. Index findings theo severity
Bảng F-NN | Severity | Title | Suggested fix effort (S/M/L)

## 3. Findings detail
### Phase A — Consistency
F-01, F-02, ...

### Phase B — Methodology integrity
F-NN, ...

### Phase C — Cross-agent sync
F-NN, ...

### Phase D — Legacy / dead-code
F-NN, ...

### Confirm Intent (design decision đã accept)
CI-01, CI-02, ...

## 4. Methodology
- L2 semantic check, không deep L3
- Tools dùng: grep, diff, đọc kỹ master file
- File đã đọc full: [list]
- File chỉ scan partial: [list]

## 5. Recommendations
- Priority order fix (theo severity, không strict)
- Fix nào nên gộp 1 commit, fix nào nên tách
- Fix nào cần user xác nhận intent trước

## 6. Appendix
- Bảng raw token gặp + bảng dịch coverage
- Bảng diff K_agent_db ↔ agent_db (per file, line count diff)
- Pattern dead-code matches (raw grep output)
```

### 5.3. Effort fix tag

Mỗi finding kèm tag effort fix (S = <30 phút, M = 30-120 phút, L = >2h) để user prioritize:
- Critical S → fix ngay
- Major L → cân nhắc batch
- Nit S → batch cleanup pass cuối

---

## 6. Trình tự execute

```
Step 1   Pre-flight: đọc README + CLAUDE.md + 4 system_prompt + KERNEL_SKELETON + INDEX + 3 master file (`_00`)
         → Verify: nắm chắc scope & convention trước audit

Step 2   Phase A (Consistency)
         → Verify: filesystem count, cross-reference spot check, manifest sync
         → Output: A.1 đến A.6 findings

Step 3   Phase D (Legacy/dead-code) — chạy trước Phase B vì grep-based, nhanh
         → Verify: 6 pattern grep + section obsolete + orphan check
         → Output: D.1 đến D.3 findings

Step 4   Phase C (Cross-agent sync) — chạy trước Phase B vì diff-based, mechanical
         → Verify: 6 cặp file `K_agent_db_*` ↔ `agent_db_*` diff đầy đủ
         → Output: C.1, C.2 findings

Step 5   Phase B (Methodology integrity) — chạy cuối vì cần đọc kỹ nhất, mất nhiều thời gian
         → Verify: 6 nguyên tắc + flex+downgrade + K hygiene coverage + constraint KH + 9 report_type ↔ P/O + workflow reachability
         → Output: B.1 đến B.6 findings

Step 6   Compile report
         → Verify: mọi finding có file:line, severity, suggested fix, effort tag
         → Verify: index theo dimension + theo severity + executive summary

Step 7   Self-review report
         → Check: placeholder, internal contradiction, ambiguity, scope drift
         → Fix inline

Step 8   Hand-off user
         → User đọc report → quyết priority fix
```

---

## 7. Estimate effort

| Phase | Effort estimate | Note |
|---|---|---|
| Step 1 (Pre-flight) | 30-45 phút | Đọc nắm chắc convention |
| Phase A | 60-90 phút | Filesystem + cross-ref check |
| Phase D | 30-45 phút | Grep mechanical |
| Phase C | 45-60 phút | 6 cặp diff |
| Phase B | 120-180 phút | Đọc kỹ, đối chiếu nhiều file |
| Compile + self-review | 30-45 phút | |
| **Tổng** | **~6-8h** | Có thể tách 2 session |

---

## 8. Risk / limitation của plan

- **Risk 1 — False positive Critical:** Một số finding tôi flag Critical có thể là design decision tôi chưa hiểu hết. Mitigation: dùng `[Confirm Intent]` thay vì Critical khi không chắc.
- **Risk 2 — Miss methodology nuance:** L2 không đọc toàn bộ file con. Có thể miss issue chỉ phát hiện được khi đọc file con kỹ. Mitigation: đối với Critical/Major suspect, sẽ escalate lên đọc file con liên quan.
- **Risk 3 — Grep miss:** Pattern dead-code có thể miss nếu naming variant. Mitigation: list multiple variant cho mỗi pattern (vd: `present_files`, `present-files`, `presentFiles`).
- **Risk 4 — Diff noise C.1:** Hai file 99% identical sẽ có rất nhiều diff line. Mitigation: phân loại trước (Expected vs Drift vs Forgotten port) rồi mới quyết flag.
- **Risk 5 — Effort overrun:** 6-8h là estimate. Nếu Phase B tìm ra nhiều methodology drift, effort có thể vượt. Mitigation: nếu vượt 50%, pause hỏi user có muốn tiếp hay scope down.

---

## 9. Pre-execute checklist (gate trước khi bắt đầu)

- [ ] User đã đọc plan này
- [ ] User confirm scope A+B+C+D, depth L2, severity 4 mức
- [ ] User confirm OK với `docs/audit/2026-04-27-audit-report.md` là output path
- [ ] User confirm OK với effort estimate ~6-8h (có thể tách session)
- [ ] User confirm "no fix" — chỉ audit report, không fix trực tiếp file
- [ ] User confirm OK với risk/limitation ở mục 8

---

**End of plan.**
