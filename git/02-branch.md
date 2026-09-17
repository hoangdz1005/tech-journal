# 02. Branch & Checkout

[← Mục lục](README.md)

## Làm việc với Branch

Branch là các nhánh phát triển song song trong cùng một repository. Nhờ đó bạn có thể làm tính năng mới, sửa lỗi hay thử nghiệm mà không ảnh hưởng tới mã nguồn chính. Mỗi nhánh có lịch sử commit riêng, và thay đổi ở nhánh này không ảnh hưởng nhánh khác cho tới khi merge. Điều này giúp tổ chức công việc tốt hơn và cho phép nhiều người làm việc đồng thời mà không giẫm chân nhau.

Phần này gồm các lệnh tạo, chuyển, liệt kê, đổi tên, xoá nhánh, xem quan hệ giữa các nhánh và quản lý nhánh remote.

### `git branch <branch_name>`

Tạo nhánh mới (nhưng chưa chuyển sang nhánh đó).

### `git checkout <branch_name>`

Chuyển sang nhánh chỉ định và cập nhật working directory theo nhánh đó.

### `git branch`

Liệt kê các nhánh local. Nhánh hiện tại được đánh dấu `*`.

### `git branch -d <branch_name>`

Xoá một nhánh.

> ⚠️ **Lưu ý:** `-d` chỉ xoá được nhánh đã merge. Muốn xoá bắt buộc (mất các commit chưa merge) dùng `git branch -D <branch_name>`.

### `git push --delete <remote> <branch>`

Xoá một nhánh trên remote.

```bash
git push --delete origin feature/old
```

### `git branch -m <old_name> <new_name>`

Đổi tên nhánh.

```bash
git branch -m master main
git branch -m new-name        # đổi tên nhánh hiện tại
```

### `git checkout -b <new_branch>`

Tạo nhánh mới tên `<new_branch>` từ nhánh hiện tại và chuyển sang đó ngay.

### `git switch <branch>`

Chuyển working directory sang nhánh chỉ định. Đây là lệnh mới hơn, chuyên cho việc chuyển nhánh (tách khỏi `checkout`).

```bash
git switch main
git switch -c feature/login   # tạo và chuyển sang nhánh mới
git switch -                  # quay lại nhánh trước đó
```

### `git show-branch <branch>`

Tóm tắt lịch sử commit và quan hệ giữa các nhánh được chọn, cho thấy các nhánh tách ra từ đâu.

### `git show-branch --all`

Tương tự lệnh trên nhưng áp dụng cho tất cả các nhánh.

### `git branch -r`

Liệt kê các nhánh remote mà repo local đang biết (remote-tracking branches).

### `git branch -a`

Liệt kê mọi nhánh, cả local lẫn remote (những nhánh remote mà repo local đã biết).

### `git branch --merged`

Liệt kê các nhánh đã merge hoàn toàn vào nhánh hiện tại. Nếu không cần nữa có thể xoá an toàn.

### `git branch --no-merged`

Liệt kê các nhánh chưa merge hoàn toàn vào nhánh hiện tại, tức là còn thay đổi chưa được tích hợp.

---

## Checkout

`git checkout` là lệnh đa năng: chuyển giữa các nhánh, tag hoặc commit bằng cách cập nhật working directory và index theo đối tượng được chọn. Lệnh này còn dùng để tạo nhánh mới, khôi phục file từ một commit, hoặc tạo nhánh không có lịch sử với `--orphan`.

### `git checkout <commit>`

Cập nhật working directory và index theo commit chỉ định để xem hoặc thử nghiệm trạng thái repo tại thời điểm đó. Bạn sẽ ở trạng thái **"detached HEAD"**, tức là không nằm trên nhánh nào.

> 💡 Nếu muốn giữ lại commit tạo ra khi đang detached HEAD, hãy tạo nhánh: `git switch -c <new_branch>`.

### `git checkout -b <branch> <commit>`

Tạo nhánh mới bắt đầu từ commit chỉ định và chuyển sang nhánh đó, để làm tiếp từ điểm này trong lịch sử.

```bash
git checkout -b hotfix/1.2.1 v1.2.0
```

### `git checkout <commit> -- <file>`

Khôi phục file từ một commit cụ thể vào working directory, thay thế phiên bản hiện tại. Lịch sử commit không bị thay đổi.

> ⚠️ **Lưu ý:** Lệnh này cập nhật cả **index** lẫn working directory (file được stage luôn). Cách viết tương đương: `git restore --source=<commit> --staged --worktree <file>`.

### `git checkout --orphan <new_branch>`

Tạo nhánh mới không có lịch sử commit, như thể bắt đầu một repository mới. Thường dùng cho nhánh `gh-pages` hoặc để làm lại lịch sử từ đầu.

> ⚠️ **Lưu ý:** Sau lệnh này, các file của nhánh cũ vẫn còn trong working directory và index. Muốn bắt đầu thật sự "sạch" hãy chạy thêm `git rm -rf .`.

---

[← Trước: Lệnh cơ bản](01-co-ban.md) · [Mục lục](README.md) · [Tiếp: Merge & xung đột →](03-merge-va-xung-dot.md)
