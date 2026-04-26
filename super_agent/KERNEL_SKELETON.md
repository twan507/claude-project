# KERNEL SKELETON

File index của project knowledge. Agent đọc file này đầu session để biết pack nào available và trigger nào activate pack nào. Chi tiết nội dung pack nằm trong file `_00` master của pack tương ứng, không duplicate vào đây.

## Cách dùng file này

1. Agent scan file này đầu session.
2. Khi query user đến, match với trigger của pack để quyết định activate pack nào.
3. Activate pack thì đọc `_00` master của pack đó trước (bắt buộc theo rule master-first reading trong system prompt mục 5.7).
4. Nếu không match pack nào, fall back về Default inline hoặc Default report (xem system prompt mục 6).
5. Pack không có trong file này = không tồn tại. Agent không được suy diễn pack ngoài danh sách.

## K — Knowledge packs

### K_agent_db

**Mục đích:** Knowledge base về dữ liệu chứng khoán Việt Nam trong MongoDB `agent_db`. Bao gồm schema 23 collection, query patterns, anti-patterns, methodology diễn giải chỉ báo (dòng tiền, technical zone, trend đa khung, PTCB theo 4 type doanh nghiệp), methodology phân tích tin tức 4 loại (doanh nghiệp, quốc tế, trong nước, thông cáo).

**Master:** `K_agent_db_00`

**Trigger:** Mọi query về cổ phiếu Việt Nam, thị trường VN, ticker, ngành, BCTC, dòng tiền, khối ngoại, technical, tin tức chứng khoán VN, hoặc khi cần số liệu định lượng từ `agent_db`.

**Depends:** Không có.

**Status:** Active.

## P — Process packs

### P_invest_memo

**Mục đích:** Workflow đầu tư cổ phiếu Việt Nam niêm yết, horizon 1-6 tháng, chỉ long, portfolio dưới 1 triệu USD. Pipeline 5 giai đoạn: gate vĩ mô → chọn 3-5 ngành → screen 6-10 mã/ngành → chấm điểm top 3/ngành → memo deep-dive 7 phần. Bổ sung song song: portfolio construction + monitoring & exit. Mỗi giai đoạn kết bằng checkpoint review 6 phần, chờ user confirm trước khi qua tier kế.

**Master:** `P_invest_memo_00`

**Trigger:** User mention tier hoặc giai đoạn cụ thể (ví dụ "tier 3", "giai đoạn 2", "gate vĩ mô"); yêu cầu "chạy quy trình đầu tư", "viết memo", "deep-dive mã X", "screen ngành", "shortlist", "portfolio construction", "monitoring danh mục", "rebalance".

**Depends:** `K_agent_db`.

**Status:** Active.

## O — Output packs

Chưa có pack nào. Dự kiến các O pack tương lai:

- Output memo dạng docx với template cố định
- Output pitch deck dạng pptx
- Output inline giọng phân tích viên chuyên nghiệp
- Output inline giọng chat gọn (kiểu Zalo)
- Output model định giá dạng xlsx

Khi chưa có O pack tương ứng, Kernel dùng Default inline hoặc Default report theo system prompt mục 6.

## Naming convention tham chiếu

Pack tương lai đặt tên theo pattern (lặp lại từ system prompt mục 2 để agent tiện đối chiếu):

```
K_{domain}_{NN}            ví dụ K_realestate_00, K_macro_00
P_{flow_name}_{NN}         ví dụ P_quick_check_00, P_earnings_review_00
O_{format_or_style}_{NN}   ví dụ O_memo_docx_00, O_inline_chat_00
```

Số `NN` ý nghĩa nội bộ pack, quy định trong file `_00` của pack đó.

## Lịch sử update

- 2026-04-24: Khởi tạo kernel skeleton. Thêm K_agent_db và P_invest_memo.
