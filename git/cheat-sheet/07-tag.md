# 07. Tag

[← Mục lục](README.md)

Tag dùng để đánh dấu những điểm quan trọng trong lịch sử repository bằng tên có ý nghĩa, thường là phiên bản phát hành hoặc cột mốc. Khác với branch, tag thường không thay đổi và là tham chiếu cố định tới một commit.

Git có hai loại tag:

- **Lightweight tag**: chỉ là con trỏ tới một commit.
- **Annotated tag**: lưu thêm metadata như tên, email người tạo, ngày tạo và thông điệp.

Tag có thể tạo, liệt kê, đẩy lên remote và xoá dễ dàng, giúp quản lý các mốc quan trọng của dự án.

### `git tag <tag_name>`

Tạo tag (lightweight) trỏ tới commit hiện tại, thường dùng để đánh dấu bản phát hành.

```bash
git tag v1.0.0
git tag            # liệt kê các tag
git tag -l "v1.*"  # lọc tag theo mẫu
```

### `git tag -a <tag_name> -m "message"`

Tạo annotated tag trỏ tới commit hiện tại, kèm thông điệp và metadata (tên, email người tạo, ngày).

```bash
git tag -a v1.0.0 -m "Phát hành phiên bản 1.0.0"
git tag -a v0.9.0 a1b2c3d -m "Tag cho commit cũ"
```

### `git tag -d <tag_name>`

Xoá tag khỏi repository local.

> 💡 Muốn xoá tag trên remote: `git push origin --delete <tag_name>`.

### `git tag -f <tag> <commit>`

Ép tag trỏ sang commit khác.

### `git show <tag_name>`

Hiển thị thông tin chi tiết về tag: commit mà tag trỏ tới và thông điệp/chú thích (nếu có).

### `git push origin <tag_name>`

Đẩy tag chỉ định lên remote để mọi người cùng dùng.

### `git push origin --tags`

Đẩy tất cả tag local lên remote.

### `git push --follow-tags`

Đẩy commit kèm theo các tag.

> ⚠️ **Lưu ý:** `--follow-tags` chỉ đẩy các **annotated tag** trỏ tới commit đang được push. Lightweight tag không được đẩy theo.

### `git fetch --tags`

Lấy tất cả tag từ remote mặc định về repository local, không ảnh hưởng tới các nhánh hiện tại.

---

[← Trước: Stash](06-stash.md) · [Mục lục](README.md) · [Tiếp: Hoàn tác & Reflog →](08-hoan-tac-va-reflog.md)
