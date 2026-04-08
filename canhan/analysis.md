# UX Case Study: Vietnam Airlines Chatbot NEO
### Bài tập UX — Phân tích sản phẩm AI thật

---

## PHẦN 1 — KHÁM PHÁ (Explore)

### Trước khi dùng — Marketing hứa gì?

| Nguồn | Nội dung marketing |
|-------|-------------------|
| Website VNA | "NEO — Trợ lý ảo thông minh, hỗ trợ 24/7" |
| Zalo VNA | "Chat với NEO để tra cứu, đặt vé, check-in nhanh chóng" |
| PR articles | "AI-powered chatbot, hiểu ngôn ngữ tự nhiên, xử lý đa nhiệm" |

> **Tóm tắt lời hứa:** NEO được giới thiệu như một trợ lý AI thông minh, hiểu ngôn ngữ tự nhiên, xử lý đa dạng nhu cầu — tạo kỳ vọng gần như một human agent ảo.

### Khi dùng thử — Thực tế thấy gì?

- NEO **không phải** AI conversational thực sự → chủ yếu là **rule-based menu bot**
- Giao diện: các nút bấm cố định, ít có input text tự do
- Khi gõ câu tự nhiên → thường fallback vào menu hoặc "không hiểu"
- Không có NLU sâu, không maintain context giữa các turn

> **Gap marketing vs thực tế:** Marketing nói "AI thông minh, hiểu ngôn ngữ tự nhiên" nhưng thực tế là menu bot với vài shortcut cố định. Gap rất lớn.

---

## PHẦN 2 — PHÂN TÍCH 4 PATHS

### PATH 1 — Khi AI ĐÚNG ✅

**Test:** Bấm nút "Tra cứu chuyến bay" → nhập mã VN123 + ngày đúng format

```
[User bấm "Tra cứu chuyến bay"]
        ↓
[NEO hiện form: Mã chuyến + Ngày]  ← UI change: form xuất hiện
        ↓
[User nhập: VN123, 15/01/2025]
        ↓
[NEO trả về: Đúng giờ, Gate B2, Belt 3] ✅
```

**Quan sát:**
- User thấy thông tin chuyến bay đúng
- Hệ thống confirm bằng cách **hiển thị thẳng kết quả**, không có "Tôi nghĩ bạn muốn nói..."
- Nút bấm biến mất sau khi chọn → chuyển sang form
- **Nhận xét:** Path này OK nhưng chỉ đúng khi user đi theo **flow định sẵn** (bấm nút → nhập form). Không phải "AI hiểu" mà là form thu thập data.

> 🟢 **Xử lý khá** — nhưng chỉ vì đây là path đơn giản nhất, ít phụ thuộc AI thực sự.

---

### PATH 2 — Khi AI KHÔNG CHẮC ⚠️

**Test:** Gõ câu hơi lệch: "cho tui xem chuyến bay 123 ngày mốt"

```
[User: "cho tui xem chuyến bay 123 ngày mốt"]
        ↓
[NEO: "Tôi không hiểu yêu cầu của bạn"] ❌
        ↓
[Hiện lại 6 nút menu mặc định]
```

**Quan sát:**
- KHÔNG có xử lý "không chắc" → nhảy thẳng sang "không hiểu"
- KHÔNG hỏi lại ("Bạn có nghĩa là chuyến VN123?")
- KHÔNG show alternatives ("Bạn muốn tra cứu chuyến bay đúng không?")
- Nút menu xuất hiện lại → **reset toàn bộ context**
- **Nhận xét:** Path "không chắc" **KHÔNG TỒN TẠI**. Bot nhảy thẳng từ unsure → wrong mà không có middle ground.

> 🔴 **Yếu nhất** — Thiếu hoàn toàn lớp xử lý uncertainty. Đây là chỗ gãy lớn nhất.

---

### PATH 3 — Khi AI SAI ❌

**Test:** Hỏi thông tin mà bot hiểu sai intent

```
[User: "Chuyến VN123 delay không?"]
        ↓
[NEO hiểu thành "tra cứu chuyến bay"]
        ↓
[Hiện form nhập mã + ngày]  ← Sai intent hoàn toàn
        ↓
[User nhận ra sai → phải bấm lại nút từ đầu]
        = 3 bước mới quay lại đúng flow
```

**Quan sát:**
- User **không biết ngay** rằng AI sai → bot im lặng show form, user phải tự nhận ra
- **Không có indicator** "Tôi hiểu câu hỏi của bạn là..."
- Sửa = **quay lại từ đầu** (bấm nút menu lại) → không có "undo" hay "đó không phải ý tôi"
- Ít nhất **3 bước** để sửa: nhận ra sai → thoát form → chọn lại nút đúng
- **Nhận xét:** Không có feedback loop. AI sai nhưng không cho user cơ hội sửa nhanh.

> 🔴 **Rất yếu** — Không có confirm-before-act, không có cách sửa ngắn.

---

### PATH 4 — Khi USER MẤT TIN 😤

**Test:** Thử 2 câu bot không hiểu liên tiếp, rồi hỏi câu phức tạp

```
[Turn 1: "Chuyến bay VN123 delay không?" → "Không hiểu"]
[Turn 2: "Cho tui đổi vé"]              → "Liên hệ hotline"]
[Turn 3: "Tại sao?"                     → "Không hiểu"]
        ↓
[User bực, muốn thoát / gọi người thật]
        ↓
[Tìm hotline...]
        ↓
[Hotline 1900 1100 nằm ở ĐÁY trang, font nhỏ] 😤
        ↓
[KHÔNG có nút "Nói chuyện với nhân viên" trong chat]
```

**Quan sát:**
- **Exit trong chat:** KHÔNG CÓ nút "Chuyển sang nhân viên" hoặc "Gọi hotline"
- **Exit ngoài chat:** Hotline bị ch buried ở footer website
- **Fallback:** Chỉ có "Liên hệ 1900..." dạng text, không phải clickable button
- User mất tin → **không có lối thoát rõ ràng** → friction tối đa
- **Nhận xét:** Path mất tin là **đen nhất**. Không có escape hatch, user bị kẹt trong vòng lặp "không hiểu".

> 🔴🔴 **Không tồn tại** — Đây là critical failure point.

---

### TỔNG KẾT 4 PATHS

| Path | Trạng thái | Đánh giá |
|------|-----------|----------|
| 1. AI Đúng | ✅ Tồn tại | Chỉ đúng khi đi theo flow cứng |
| 2. AI Không chắc | ❌ Không tồn tại | Nhảy thẳng sang "sai" |
| 3. AI Sai | 🔴 Rất yếu | Không confirm, sửa mất 3+ bước |
| 4. Mất tin | ❌❌ Không tồn tại | Không exit, không fallback |

**Path xử lý tốt nhất:** Path 1 (AI đúng) — vì nó không phụ thuộc AI, chỉ là form.
**Path yếu nhất:** Path 4 (Mất tin) — không có exit route, user bị kẹt hoàn toàn.

**Gap marketing vs thực tế:**
- Marketing: "AI thông minh, hiểu ngôn ngữ tự nhiên"
- Thực tế: Rule-based menu bot, Paths 2-3-4 gần như không được thiết kế
- **Gap:** Khoảng 80% — lời hứa vs thực tế cách xa nhau

---

## PHẦN 3 — SKETCH "LÀM TỐT HƠN"

### Chọn path yếu nhất: **PATH 2 — Khi AI KHÔNG CHẮC**

*(Vì path này là gốc rễ: nếu xử lý tốt "không chắc", thì giảm được "sai" và "mất tin")*

---

### SKETCH AS-IS (Bên trái)

```
┌─────────────────────────────┐
│  AS-IS: Path "Không chắc"   │
│  (KHÔNG TỒN TẠI)            │
├─────────────────────────────┤
│                             │
│  User: "chuyến 123 ngày     │
│         mốt"                │
│           │                 │
│           ▼                 │
│     ┌───────────┐           │
│     │  ❌ KHÔNG │           │
│     │  TỒN TẠI! │           │
│     └─────┬─────┘           │
│           │                 │
│           ▼                 │
│  Bot: "Tôi không hiểu"      │
│           │                 │
│           ▼                 │
│  [6 nút menu hiện lại]      │
│  ← RESET CONTEXT            │
│           │                 │
│           ▼                 │
│  User: 😤 bực, thoát        │
│                             │
│  ⚡ ĐÁNH DẤU CHỖ GÃY:      │
│  → Bot từ "unsure" nhảy    │
│    thẳng xuống "wrong"     │
│  → Không có lớp hỏi lại    │
│  → Context bị mất          │
│                             │
└─────────────────────────────┘
```

---

### SKETCH TO-BE (Bên phải)

```
┌─────────────────────────────┐
│  TO-BE: Path "Không chắc"   │
│  (THÊM CONFIDENCE LAYER)    │
├─────────────────────────────┤
│                             │
│  User: "chuyến 123 ngày     │
│         mốt"                │
│           │                 │
│           ▼                 │
│  ┌──────────────────┐       │
│  │ 🤔 AI UNSURE     │       │
│  │ Confidence: 65%  │       │
│  └────────┬─────────┘       │
│           │                 │
│           ▼                 │
│  Bot: "Bạn muốn tra cứu     │
│  chuyến bay VN123           │
│  ngày 17/01 đúng không?"    │
│                             │
│  ┌──────────┐ ┌──────────┐  │
│  │ ✅ Đúng │ │ ❌ Sai   │  │
│  └────┬─────┘ └────┬─────┘  │
│       │            │        │
│       ▼            ▼        │
│  [Tra cứu    [Bot hỏi lại]  │
│   ngay]        │            │
│                ▼            │
│         "Bạn có thể nói     │
│          rõ hơn được không?"│
│         + Quick replies:    │
│         [Tra cứu] [Đổi vé]  │
│         [Hành lý] [Khác]    │
│                             │
│  ═══════════════════════    │
│  THÊM GÌ:                   │
│  + Confidence scoring       │
│  + Confirm trước khi act    │
│  + Quick replies sửa sai    │
│  + Giữ nguyên context       │
│                             │
│  BỚT GÌ:                    │
│  - Bỏ "Tôi không hiểu"      │
│  - Bỏ reset menu            │
│                             │
│  ĐỔI GÌ:                    │
│  - Unsure → Confirm         │
│    thay vì Unsure → Fail    │
│                             │
└─────────────────────────────┘
```

---

### SO SÁNH AS-IS vs TO-BE

| | AS-IS | TO-BE |
|--|-------|-------|
| **Bot nhận diện unsure** | ❌ Không | ✅ Confidence score |
| **Hỏi lại user** | ❌ Không | ✅ "Bạn có nghĩa là...?" |
| **User sửa sai** | 3+ bước (quay lại menu) | 1 bước (bấm "Sai") |
| **Context** | Bị mất | Được giữ |
| **Fallback options** | 6 nút menu chung chung | Quick replies context-aware |
| **Cảm xúc user** | 😤 Bực | 🤔 Được tôn trọng |

---

## PHẦN 4 — NHẬN XÉT BỔ SUNG

### Tại sao Path "Không chắc" là quan trọng nhất?

```
         AI Đúng (Path 1)
              ↑
     AI Không chắc (Path 2)  ← THÊM LỚP NÀY = GIẢM 2 PATH DƯỚI
              ↓
         AI Sai (Path 3)
              ↓
      User mất tin (Path 4)
```

> **Insight:** Path 2 là **van giảm áp** của toàn bộ hệ thống. Nếu xử lý tốt "không chắc" → AI ít sai hơn (Path 3 giảm) → User ít mất tin hơn (Path 4 giảm). Đây là điểm can thiệp ROI cao nhất.

### Marketing vs Thực tế — Gap Map

```
Marketing hứa              Thực tế
─────────────             ─────────────
"AI thông minh"    ←──gap──→ Menu bot rule-based
"Hiểu ngôn ngữ     ←──gap──→ Chỉ hiểu lệnh cứng
 tự nhiên"
"Hỗ trợ đa dạng    ←──gap──→ 6 nút menu cố định
 nhu cầu"
"24/7"             ←──gap──→ 24/7 nhưng 80% fallback
```

---

## CHECKLIST NỘP BÀI

- [x] **Phân tích 4 paths** — đủ 4 paths + nhận xét yếu nhất (4đ)
- [x] **Sketch as-is + to-be** — rõ ràng, ghi chú thêm/bớt/đổi (4đ)
- [x] **Gap marketing vs thực tế** — có bảng so sánh + insight (2đ)
- [ ] **Bonus** — trình bày 30s nhóm vote

---
