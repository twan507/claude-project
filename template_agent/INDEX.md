# INDEX — Template Agent Manifest

File index của template_agent. Đọc đầu session để biết file nào available + workflow tổng.

## Cách dùng

1. Đầu session đọc file này 1 lần để có tổng quan
2. Khi user cung cấp input → áp dụng `WORKFLOW.md` từ Stage 1
3. Cần lookup spec MD chuẩn → đọc `FORMAT.md`
4. Cần render binary → đọc `TEMPLATE_X.md` (X = VBSE hoặc FINEXT) tương ứng brand user pick

## Manifest file

| File | Vai trò |
|---|---|
| `system_prompt.md` | Meta-rules vận hành agent (load qua project setting) |
| `INDEX.md` | File này — manifest + workflow overview |
| `FORMAT.md` | Spec MD chuẩn hóa — input/output contract |
| `WORKFLOW.md` | Flow ingest → clarify → normalize → render (7 stage + 3 checkpoint) |
| `TEMPLATE_VBSE.md` | Catalog 27 layout pptx brand VBSE — visual style navy + đỏ + tam giác |
| `TEMPLATE_VBSE.pptx` | Binary template VBSE 27 layout |
| `TEMPLATE_FINEXT.md` | Catalog 27 layout pptx brand Finext — visual style dark + violet + chevron |
| `TEMPLATE_FINEXT.pptx` | Binary template Finext 27 layout |

## Workflow tổng quan

```
USER input (PDF/DOCX/MD/text/paste)
    │
    ▼
┌───────────────────────────────────────────────┐
│ Stage 1: Ingest                               │
│ Stage 1.5: Detect skip-normalize              │
│   ├─ YES → đi thẳng Stage 6                   │
│   └─ NO  → tiếp Stage 2                       │
│ Stage 2: Parse (LLM analyze)                  │
│ Stage 3: Clarify (multi-choice questions)     │
│ ─── CP1: Clarification confirm ───            │
│ Stage 4: Normalize (LLM produce MD)           │
│ ─── CP2: MD draft review ───                  │
│ Stage 5: Finalize MD                          │
└───────────────────────────────────────────────┘
    │
    ▼ (MD final, đã đúng FORMAT contract)
┌───────────────────────────────────────────────┐
│ Stage 6: Brand pre-flight                     │
│   Hỏi user: VBSE / Finext / chỉ MD            │
│ ─── CP3: Brand confirm ───                    │
│ Stage 7: Render binary                        │
│   TEMPLATE pack runtime → pptx                │
└───────────────────────────────────────────────┘
    │
    ▼
USER nhận: MD final + (optional) pptx branded
```

Chi tiết mỗi stage: `WORKFLOW.md`.

## Brand whitelist

Hiện chỉ render được 2 brand:

| Brand | File pack | Visual signature |
|---|---|---|
| **VBSE** | `TEMPLATE_VBSE.md/.pptx` | Navy `#003D7A` + đỏ `#D71249` + tam giác vuông cân |
| **Finext** | `TEMPLATE_FINEXT.md/.pptx` | Dark `#0A0A0A` + violet `#8B5CF6` + chevron `>>` |

Brand khác → reject. Không có fallback "render plain branded" — chỉ chọn 1 trong 2 hoặc skip render binary (chỉ xuất MD).

## 9 loại báo cáo support

8 preset type + 1 custom. Custom là quiz-driven — agent hỏi user xây spec runtime. Chi tiết structure xem `FORMAT.md` mục 3.

| report_type | Mô tả | Section count | Length |
|---|---|---|---|
| `stock_pitch` | Khuyến nghị MUA mã đơn lẻ gửi KH | 15-18 (rigid + 3 luận điểm flex) | 12-18 trang |
| `weekly_market` | Báo cáo thị trường tuần | 12 rigid | 9-11 trang |
| `market_scan` | Báo cáo scan đầu cycle (top-down) | 7 flex | 8-15 trang |
| `stock_memo` | Memo deep-dive nội bộ 1 mã | 3-7 theo conviction tier | 3-15 trang |
| `portfolio_plan` | Kế hoạch portfolio + order list | 8 flex | 6-10 trang |
| `portfolio_review_weekly` | Review portfolio tuần | 6 rigid | 0.5-2 trang |
| `portfolio_review_monthly` | Review portfolio tháng | 8 rigid | 3-5 trang |
| `portfolio_review_quarterly` | Review portfolio quý + BCTC | 9 rigid | 5-8 trang |
| `custom` | Báo cáo tùy chỉnh — quiz-driven | flex 3-15 (theo spec quiz) | tùy user |

### Custom quiz

Khi user pick `custom` (hoặc Stage 2 detect không match 8 preset), agent chạy bộ trắc nghiệm 7 câu chia 2 turn để xác định: mục đích / audience / length / tone / số section / chart count / citation style. Sau khi user trả lời, agent build spec runtime + lưu vào frontmatter `custom_spec_id`. Detail xem `WORKFLOW.md` mục 5.4-5.5.

## Task ngoài scope

Pack này chỉ làm 1 việc: **nhận document có sẵn → chuẩn hoá MD → render binary branded**.

Task ngoài scope (từ chối lịch sự):
- Phân tích cổ phiếu / tra cứu data / sinh thesis gốc
- Query database market / financials
- Sinh nội dung mới không có trong input
- Render brand ngoài whitelist (xem `FORMAT.md` brand whitelist)

