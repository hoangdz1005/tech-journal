# 13. Tham chiếu, tracking & index

[← Mục lục](README.md)

## Tham chiếu (References)

Tham chiếu (refs) là con trỏ tới các commit hoặc đối tượng trong repository: branch, tag, và các ref đặc biệt như `HEAD` (trỏ tới commit đang checkout). Git dùng ref để theo dõi cấu trúc và lịch sử repository, giúp đặt tên và làm việc với các commit dễ dàng hơn thay vì phải nhớ hash.

```
.git/refs/heads/main        → nhánh local
.git/refs/tags/v1.0.0       → tag
.git/refs/remotes/origin/*  → nhánh remote-tracking
.git/HEAD                   → ref: refs/heads/main
```

### `git show-ref --heads`

Liệt kê tham chiếu của tất cả head (các nhánh).

> ⚠️ **Lưu ý:** Từ Git 2.46, `--heads` được coi là deprecated, nên dùng `git show-ref --branches`.

### `git show-ref --tags`

Liệt kê tham chiếu của tất cả tag.

---

## Tracking

Tracking là việc theo dõi và quản lý các file trong repository. `git ls-files` liệt kê những file đang được Git theo dõi, còn `git ls-tree` hiển thị nội dung một tree object của nhánh chỉ định (cấu trúc thư mục và file tại thời điểm đó). Hai lệnh này giúp biết file nào đang thuộc repository và được tổ chức ra sao.

### `git ls-files`

Liệt kê tất cả file đang được theo dõi.

```bash
git ls-files
git ls-files --others --exclude-standard   # file untracked (không tính file bị ignore)
git ls-files --ignored --exclude-standard --others  # file bị ignore
```

### `git ls-tree <branch>`

Liệt kê nội dung một tree object.

```bash
git ls-tree main
git ls-tree -r --name-only main   # liệt kê đệ quy, chỉ tên file
```

---

## Thao tác với index

Thao tác với index là quản lý staging area, nơi chuẩn bị thay đổi trước khi commit. Ví dụ đánh dấu file là "assume unchanged" để tạm thời bỏ qua thay đổi, hoặc bỏ đánh dấu để theo dõi lại. Lệnh `git update-index` cho phép kiểm soát file nào có trong commit tiếp theo, linh hoạt cho các nhu cầu đặc thù.

### `git update-index --assume-unchanged <file>`

Đánh dấu file là "assume unchanged": Git sẽ không kiểm tra thay đổi của file này.

### `git update-index --no-assume-unchanged <file>`

Bỏ đánh dấu "assume unchanged".

> ⚠️ **Lưu ý:** `--assume-unchanged` vốn là tuỳ chọn **tối ưu hiệu năng**, Git có thể tự bỏ cờ này (ví dụ khi checkout). Nếu mục đích là giữ thay đổi local của một file cấu hình mà không commit, nên dùng:
>
> ```bash
> git update-index --skip-worktree <file>
> git update-index --no-skip-worktree <file>
> git ls-files -v | grep '^S'   # liệt kê file đang skip-worktree
> ```

---

[← Trước: Cấu hình](12-cau-hinh.md) · [Mục lục](README.md) · [Tiếp: Bảo trì & dọn dẹp →](14-bao-tri-va-don-dep.md)
