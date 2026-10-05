# react_native_png_parse_skills

Claude Code skills đúc kết từ việc code app RN/Expo, tập trung vào phân tích ảnh design → code chính xác và dựng nền móng UI.

Repo này vừa là **plugin marketplace** (chứa 3 skill chung), vừa chứa 1 skill riêng project (cài tay).

## Skills

**Trong plugin `react-native-design-skills` (đẩy lên marketplace):**
- **`analyzing-designs-for-code`** — phân tích ảnh design/screenshot bằng PIL/numpy để ra code chính xác (đo màu hex, kích thước, **tỉ lệ** thay vì đoán; radial profile cho icon nhiều lớp; xử lý padding trong PNG).
- **`building-react-native-app-ui`** — (cho **project mới**) dựng nền móng trước: theme tokens + component dùng chung, rồi mới code màn; list performance, gotcha Pressable, kiến trúc, design theo tỉ lệ.
- **`building-react-native-app-infra`** — phần không-UI: type API sinh từ OpenAPI (không viết tay), token + refresh single-flight, TanStack Query + Idempotency-Key cho đơn/tiền, i18n từ màn đầu, 3 trạng thái quyền camera, polling trạng thái đơn, chọn API URL theo applicationId, EAS fingerprint/OTA.

**Backend (KHÔNG nằm trong plugin — chỉ cài tay):**
- **`race-safe-points-and-money`** — chống race khi code đụng điểm/coin/ví/ledger/voucher/quà 1 lần/duyệt/import (FastAPI + SQLAlchemy async): checklist rà, pattern claim-UPDATE / FOR UPDATE theo thứ tự id / SKIP LOCKED / hash import, và cách CHỨNG MINH test race có ý nghĩa (chạy Postgres thật, nới cửa sổ bằng sleep, tắt lock phải fail).

**Riêng project (KHÔNG nằm trong plugin — chỉ cài tay):**
- **`netnam-mobile-coding`** — tên component/theme/OTA cụ thể của NetNam HRApp.

## Cài qua plugin marketplace (khuyến nghị)

Trong Claude Code:
```
/plugin marketplace add hoanghungbk93/react_native_png_parse_skills
/plugin install react-native-design-skills@react-native-png-parse-skills
```
Plugin nạp ngay, không cần restart. 2 skill chung sẽ tự kích hoạt khi ngữ cảnh khớp (cũng gọi tay được: `/react-native-design-skills:analyzing-designs-for-code`).

## Cài tay (fallback, hoặc cho skill netnam)

```bash
git clone git@github.com:hoanghungbk93/react_native_png_parse_skills.git
cd react_native_png_parse_skills
mkdir -p ~/.claude/skills
# 3 skill chung:
cp -R plugins/react-native-design-skills/skills/* ~/.claude/skills/
# skill riêng NetNam (chỉ nếu làm trên repo HRApp):
cp -R netnam-mobile-coding ~/.claude/skills/
# skill backend (race điểm/coin):
cp -R race-safe-points-and-money ~/.claude/skills/
```

Mỗi skill là 1 thư mục chứa `SKILL.md` với frontmatter `name` + `description`. Cài tay thì mở **session mới** để Claude nạp.
