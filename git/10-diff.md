# 10. Diff

[← Mục lục](README.md)

Diff cho biết sự khác nhau giữa các trạng thái trong repository: giữa working directory và staging area, giữa staging area và commit gần nhất, hoặc giữa hai commit/nhánh bất kỳ. Diff hiển thị thay đổi theo từng dòng, giúp bạn review trước khi commit, merge hay áp dụng thay đổi.

```
            git diff              git diff --cached
Working dir ─────────► Index ─────────────────► HEAD
     └───────────────── git diff HEAD ──────────────┘
```

### `git diff`

Hiển thị khác biệt giữa các trạng thái của repository. Không có tham số: so sánh working directory với index (các thay đổi chưa stage). Có thể truyền commit hoặc nhánh để so sánh chúng. Kết quả hiện theo từng dòng.

### `git diff --stat`

Tóm tắt thay đổi giữa working directory và index: file nào bị sửa, bao nhiêu dòng thêm/xoá.

### `git diff --stat <commit>`

Tóm tắt thay đổi giữa một commit và working directory.

### `git diff --stat <commit1> <commit2>`

Tóm tắt thay đổi giữa hai commit: file nào bị sửa và mức độ thay đổi.

### `git diff --stat <branch1> <branch2>`

Tóm tắt khác biệt giữa hai nhánh: file nào thay đổi và nhiều hay ít.

### `git diff --name-only <commit>`

Chỉ hiện tên các file thay đổi.

> ⚠️ **Lưu ý:** Với một commit, lệnh này so sánh commit đó với **working directory**, không phải "các file thay đổi trong commit". Muốn xem file thay đổi *trong* một commit:
>
> ```bash
> git show --name-only <commit>
> git diff --name-only <commit>~1 <commit>
> ```

### `git diff --cached`

Hiển thị khác biệt giữa index (đã stage) và commit gần nhất, tức là những gì sẽ vào commit tiếp theo. Tương đương `git diff --staged`.

### `git diff HEAD`

Hiển thị khác biệt giữa working directory (gồm cả phần đã stage và chưa stage) và commit gần nhất (HEAD).

### `git diff <branch1> <branch2>`

Hiển thị khác biệt giữa commit đầu của hai nhánh.

> 💡 `git diff branch1...branch2` (ba chấm) chỉ hiện thay đổi của `branch2` kể từ điểm tách nhánh với `branch1`, thường là thứ bạn muốn xem khi review một nhánh tính năng.

### `git difftool`

Mở công cụ diff bên ngoài để so sánh thay đổi.

### `git difftool <commit1> <commit2>`

Dùng công cụ diff để so sánh hai commit.

### `git difftool <branch1> <branch2>`

Dùng công cụ diff để so sánh hai nhánh.

### `git cherry <branch>`

So sánh commit của nhánh hiện tại với nhánh khác để tìm commit nào chưa được áp dụng sang nhánh kia.

> ⚠️ **Lưu ý:** Chính xác hơn, lệnh liệt kê các commit có ở nhánh hiện tại mà không có ở `<branch>`. Commit có dấu `+` là chưa có bản tương đương ở `<branch>`, dấu `-` là đã có (ví dụ đã được cherry-pick sang).
>
> ```bash
> git cherry -v main
> # + a1b2c3d Thêm API mới
> # - d4e5f6a Sửa lỗi (đã có trên main)
> ```

---

[← Trước: Lịch sử](09-lich-su.md) · [Mục lục](README.md) · [Tiếp: Rebase, Cherry-pick, Patch →](11-rebase-cherry-pick-patch.md)
