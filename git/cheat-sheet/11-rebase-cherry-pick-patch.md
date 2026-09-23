# 11. Rebase, Cherry-pick, Patch

[← Mục lục](README.md)

## Rebase

Rebase áp dụng lại các thay đổi của bạn lên trên lịch sử của một nhánh khác, tạo ra lịch sử dự án tuyến tính và gọn hơn. Trong thực tế, rebase giúp cập nhật nhánh mà không sinh merge commit thừa, và chuỗi commit dễ theo dõi hơn.

> ⚠️ Rebase **viết lại lịch sử** (tạo commit mới). Không nên rebase các commit đã push lên nhánh người khác đang dùng.

### `git rebase <branch>`

Áp dụng lại các commit của nhánh hiện tại lên đầu nhánh chỉ định, tức là chuyển (hoặc gộp) một chuỗi commit sang commit gốc mới. Thường dùng để:

1. Giữ lịch sử dự án tuyến tính.
2. Tích hợp thay đổi từ nhánh này sang nhánh khác.
3. Cập nhật nhánh tính năng với thay đổi mới nhất từ nhánh chính.

```bash
git switch feature/login
git rebase main
```

### `git rebase --interactive <branch>`

Bắt đầu interactive rebase cho các commit từ `<branch>` tới HEAD. Bạn có thể sắp xếp lại, squash, sửa hoặc xoá commit để dọn dẹp lịch sử trước khi push. Dạng viết tắt: `git rebase -i <branch>`.

| Lệnh | Tác dụng |
|------|----------|
| `pick` | Giữ nguyên commit |
| `reword` | Giữ commit, sửa thông điệp |
| `edit` | Dừng lại để sửa nội dung commit |
| `squash` | Gộp vào commit trước, gộp cả thông điệp |
| `fixup` | Gộp vào commit trước, bỏ thông điệp |
| `drop` | Xoá commit |

### `git rebase --continue`

Tiếp tục rebase sau khi đã giải quyết xung đột (nhớ `git add` các file đã sửa).

### `git rebase --abort`

Huỷ rebase và quay về trạng thái nhánh ban đầu.

### `git fetch --rebase`

Bản gốc mô tả lệnh này là fetch từ remote rồi rebase thay đổi local.

> ⚠️ **Lưu ý:** `git fetch` **không có** tuỳ chọn `--rebase`. Lệnh đúng là:
>
> ```bash
> git pull --rebase
> # hoặc tách hai bước:
> git fetch origin
> git rebase origin/main
> ```

---

## Cherry-pick

Cherry-pick áp dụng thay đổi của một commit cụ thể từ nhánh này sang nhánh khác. Rất hữu ích khi chỉ muốn lấy vài thay đổi riêng lẻ (ví dụ một bản sửa lỗi) mà không merge cả nhánh, tránh kéo theo những thay đổi không mong muốn.

### `git cherry-pick <commit>`

Áp dụng thay đổi của một commit có sẵn vào nhánh hiện tại (tạo commit mới).

```bash
git switch release/1.2
git cherry-pick a1b2c3d
git cherry-pick a1b2c3d..f6e5d4c   # một loạt commit (không gồm a1b2c3d)
```

### `git cherry-pick --continue`

Tiếp tục cherry-pick sau khi giải quyết xung đột.

### `git cherry-pick --abort`

Huỷ quá trình cherry-pick.

### `git cherry-pick --no-commit <commit>`

Cherry-pick nhưng không tự tạo commit, để bạn sửa thêm trước khi commit. Dạng viết tắt: `git cherry-pick -n <commit>`.

---

## Patch

Patch là cách chuyển thay đổi giữa các repository hoặc giữa các nhánh trong cùng repository. Bạn tạo file patch (file văn bản mô tả khác biệt giữa commit hoặc nhánh), rồi áp dụng vào repo khác bằng `git apply` hoặc `git am` mà không cần merge trực tiếp. Hữu ích khi chia sẻ thay đổi cụ thể giữa các codebase.

### `git apply <patch_file>`

Áp dụng thay đổi từ file patch vào working directory (không tạo commit).

### `git apply --check`

Kiểm tra patch có áp dụng sạch được không (không thay đổi gì).

```bash
git apply --check fix.patch
```

### `git format-patch <since_commit>`

Tạo file patch cho từng commit kể từ commit chỉ định (không gồm commit đó). Mỗi file giữ cả thông tin tác giả và thông điệp.

```bash
git format-patch main          # các commit có trên nhánh hiện tại nhưng chưa có trên main
git format-patch -3            # 3 commit gần nhất
```

### `git am <patch_file>`

Áp dụng patch dạng mailbox (thường do `git format-patch` tạo ra) và tạo commit tương ứng, giữ nguyên tác giả và thông điệp.

### `git am --continue`

Tiếp tục áp dụng patch sau khi giải quyết xung đột.

### `git am --abort`

Huỷ quá trình áp dụng patch.

### `git diff > <file.patch>`

Tạo file patch từ các khác biệt hiện tại.

```bash
git diff > changes.patch
git apply changes.patch
```

---

[← Trước: Diff](10-diff.md) · [Mục lục](README.md) · [Tiếp: Cấu hình →](12-cau-hinh.md)
