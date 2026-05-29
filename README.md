# react_native_png_parse_skills

Claude Code skills đúc kết từ việc code app RN/Expo, tập trung vào phân tích ảnh design → code chính xác và dựng nền móng UI.

Repo này vừa là **plugin marketplace** (chứa 2 skill chung), vừa chứa 1 skill riêng project (cài tay).

## Skills

**Trong plugin `react-native-design-skills` (đẩy lên marketplace):**
- **`analyzing-designs-for-code`** — phân tích ảnh design/screenshot bằng PIL/numpy để ra code chính xác (đo màu hex, kích thước, **tỉ lệ** thay vì đoán; radial profile cho icon nhiều lớp; xử lý padding trong PNG).
- **`building-react-native-app-ui`** — (cho **project mới**) dựng nền móng trước: theme tokens + component dùng chung, rồi mới code màn; list performance, gotcha Pressable, kiến trúc, design theo tỉ lệ.

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
# 2 skill chung:
cp -R plugins/react-native-design-skills/skills/* ~/.claude/skills/
# skill riêng NetNam (chỉ nếu làm trên repo HRApp):
cp -R netnam-mobile-coding ~/.claude/skills/
```

Mỗi skill là 1 thư mục chứa `SKILL.md` với frontmatter `name` + `description`. Cài tay thì mở **session mới** để Claude nạp.
