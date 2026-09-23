# 06. Stash

[← Mục lục](README.md)

Stash cho phép tạm cất các thay đổi chưa sẵn sàng để commit. Lệnh `git stash` cất những thay đổi này đi và đưa working directory về trạng thái sạch, để bạn chuyển nhánh hoặc làm việc khác mà không mất công sức. Sau đó dùng `git stash apply` hoặc `git stash pop` để lấy lại và làm tiếp. Rất hữu ích khi đang dở việc mà phải xử lý gấp một lỗi hoặc thử một hướng khác.

### `git stash` / `git stash save`

Tạm cất các thay đổi chưa commit, để chuyển nhánh hoặc làm thao tác khác mà không phải commit phần việc dở dang.

> ⚠️ **Lưu ý:** `git stash save` đã **deprecated**, nên dùng `git stash push`. Mặc định stash không cất file untracked, thêm `-u` nếu cần: `git stash -u`.

### `git stash -m "message"` / `git stash save "message"`

Giống lệnh trên nhưng lưu kèm thông điệp mô tả.

```bash
git stash push -m "WIP: form đăng nhập"
```

### `git stash show`

Tóm tắt thay đổi trong stash gần nhất (các file bị sửa).

```bash
git stash show -p            # xem diff đầy đủ
git stash show stash@{2}     # xem một stash cụ thể
```

### `git stash list`

Liệt kê tất cả stash trong repository theo danh sách đánh số.

```bash
git stash list
# stash@{0}: On main: WIP: form đăng nhập
# stash@{1}: WIP on feature: a1b2c3d ...
```

### `git stash pop`

Áp dụng stash gần nhất rồi xoá nó khỏi danh sách stash.

### `git stash drop`

Xoá stash gần nhất khỏi danh sách mà không áp dụng vào working directory.

### `git stash apply`

Áp dụng lại stash gần nhất vào working directory nhưng **vẫn giữ** nó trong danh sách stash.

```bash
git stash apply stash@{1}    # áp dụng một stash cụ thể
```

### `git stash clear`

Xoá toàn bộ stash, các thay đổi đã cất sẽ bị mất vĩnh viễn.

### `git stash branch <branch>`

Tạo nhánh mới từ commit mà bạn đang đứng khi stash, rồi áp dụng stash vào nhánh đó. Cách này giúp làm tiếp phần việc đã stash trên nhánh riêng, giữ nguyên bối cảnh ban đầu. Nếu áp dụng thành công, stash sẽ được xoá.

---

[← Trước: Sửa commit & Squash](05-amend-va-squash.md) · [Mục lục](README.md) · [Tiếp: Tag →](07-tag.md)
