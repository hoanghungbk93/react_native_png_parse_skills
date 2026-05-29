# react_native_png_parse_skills

Claude Code skills đúc kết từ việc code app NetNam HRApp (Expo/React Native), tập trung vào:

- **`analyzing-designs-for-code`** — phân tích ảnh design/screenshot bằng PIL/numpy để ra code chính xác (đo màu hex, kích thước, **tỉ lệ** thay vì đoán; radial profile cho icon nhiều lớp; xử lý padding trong PNG).
- **`netnam-mobile-coding`** — pattern code app: gotcha Pressable callback-style → TouchableOpacity+StyleSheet, component dùng chung, list tránh re-render (memo/useCallback/lazy), theme tokens, quy trình OTA.

## Cài đặt (Claude Code, personal)

Copy mỗi thư mục skill vào `~/.claude/skills/`:

```bash
cp -R analyzing-designs-for-code ~/.claude/skills/
cp -R netnam-mobile-coding       ~/.claude/skills/
```

Mỗi skill là 1 thư mục chứa `SKILL.md` với frontmatter `name` + `description`. Claude tự nạp khi `description` khớp ngữ cảnh.
