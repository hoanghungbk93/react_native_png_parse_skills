# react_native_png_parse_skills

Claude Code skills đúc kết từ việc code app NetNam HRApp (Expo/React Native), tập trung vào:

- **`analyzing-designs-for-code`** — phân tích ảnh design/screenshot bằng PIL/numpy để ra code chính xác (đo màu hex, kích thước, **tỉ lệ** thay vì đoán; radial profile cho icon nhiều lớp; xử lý padding trong PNG).
- **`building-react-native-app-ui`** — (tổng quát, cho **project mới**) cách dựng nền móng trước: theme tokens + bộ component dùng chung, rồi mới code màn; list performance, gotcha Pressable, kiến trúc thư mục, design theo tỉ lệ.
- **`netnam-mobile-coding`** — (riêng project NetNam HRApp) tên component/theme tokens cụ thể + quy trình OTA. Dùng kèm `building-react-native-app-ui`.

## Cài đặt (Claude Code, personal)

Copy mỗi thư mục skill vào `~/.claude/skills/`:

```bash
cp -R analyzing-designs-for-code   ~/.claude/skills/
cp -R building-react-native-app-ui ~/.claude/skills/
cp -R netnam-mobile-coding         ~/.claude/skills/
```

Mỗi skill là 1 thư mục chứa `SKILL.md` với frontmatter `name` + `description`. Claude tự nạp khi `description` khớp ngữ cảnh.
