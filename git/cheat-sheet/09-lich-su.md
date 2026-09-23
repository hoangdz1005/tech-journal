# 09. Lịch sử: Log, ngày tương đối, Blame

[← Mục lục](README.md)

## Xem lịch sử (Log)

Lịch sử Git là bản ghi mọi thay đổi của repository theo thời gian: một chuỗi commit, mỗi commit là ảnh chụp dự án tại một thời điểm. Lịch sử cho biết ai đã thay đổi gì, khi nào và vì sao, giúp hiểu quá trình phát triển mã nguồn, cộng tác hiệu quả và hỗ trợ gỡ lỗi. `git log` là công cụ chính để duyệt lịch sử này.

### `git log`

Hiển thị lịch sử commit.

### `git log --oneline`

Hiển thị mỗi commit trên một dòng.

### `git log --graph`

Vẽ lịch sử commit dạng đồ thị (thấy rõ nhánh và merge).

```bash
git log --oneline --graph --all --decorate
```

### `git log --stat`

Hiển thị lịch sử kèm thống kê file thay đổi.

### `git log --pretty=format:"%h %s"`

Định dạng đầu ra theo mẫu chỉ định.

### `git log --pretty=format:"%h - %an, %ar : %s"`

Định dạng dễ đọc hơn: hash ngắn, tác giả, thời gian tương đối, tiêu đề.

| Placeholder | Ý nghĩa |
|-------------|---------|
| `%H` / `%h` | Hash đầy đủ / rút gọn |
| `%an` / `%ae` | Tên / email tác giả |
| `%ad` / `%ar` | Ngày / thời gian tương đối |
| `%s` | Tiêu đề commit |

### `git log --author=<author>`

Chỉ hiện commit của tác giả chỉ định.

### `git log --before=<date>`

Chỉ hiện commit trước ngày chỉ định.

### `git log --after=<date>` / `git log --since=<date>`

Chỉ hiện commit sau ngày chỉ định.

```bash
git log --since="2 weeks ago" --author="Nguyen"
git log --after=2026-01-01 --before=2026-02-01
```

### `git log --cherry-pick`

Bỏ qua các commit có nội dung tương đương giữa hai nhánh.

> ⚠️ **Lưu ý:** Tuỳ chọn này chỉ có tác dụng khi dùng với khoảng đối xứng (symmetric difference), ví dụ:
>
> ```bash
> git log --cherry-pick --left-right main...feature
> ```

### `git log --follow <file>`

Hiển thị lịch sử commit của một file, kể cả qua các lần đổi tên.

### `git log --show-signature`

Hiển thị thông tin chữ ký GPG của các commit.

### `git shortlog`

Tóm tắt `git log` theo từng tác giả.

### `git shortlog -sn`

Tóm tắt theo tác giả kèm số lượng commit, sắp xếp giảm dần.

### `git log --simplify-by-decoration`

Chỉ hiện các commit được tag hoặc branch trỏ tới.

### `git log --no-merges`

Bỏ qua merge commit.

### `git whatchanged`

Liệt kê commit cùng các file thay đổi, định dạng giống log.

> ⚠️ **Lưu ý:** Lệnh này đã cũ và không còn được khuyến khích. Dùng `git log --raw` thay thế.

### `git diff-tree --pretty --name-only --root <commit>`

Hiển thị thông tin commit cùng danh sách file thay đổi trong commit đó. `--root` giúp lệnh hoạt động cả với commit đầu tiên.

### `git log --first-parent`

Chỉ đi theo parent đầu tiên của merge commit, tức là chỉ thấy commit của nhánh hiện tại, bỏ các commit được merge vào từ nhánh khác.

---

## Ngày tương đối

Git cho phép tham chiếu các thời điểm trong lịch sử bằng biểu thức thời gian dễ đọc. Ví dụ `main@{1.week.ago}` hay `@{3.days.ago}` cho phép xem trạng thái nhánh hoặc thay đổi trong một khoảng thời gian tính tới hiện tại. Bạn có thể dùng "yesterday", "2 weeks ago" hay ngày cụ thể mà không cần nhớ hash commit.

> ⚠️ **Lưu ý:** Cú pháp `<ref>@{<thời gian>}` đọc dữ liệu từ **reflog local**. Nó cho biết nhánh *trên máy bạn* trỏ vào đâu tại thời điểm đó, nên không hoạt động với khoảng thời gian cũ hơn reflog hoặc trên repo vừa clone. Muốn lọc commit theo ngày tạo, dùng `git log --since/--until`.

### `git show main@{1.week.ago}`

Xem trạng thái nhánh `main` một tuần trước.

### `git diff @{3.days.ago}`

Xem những gì đã thay đổi trong 3 ngày qua.

### `git checkout main@{2.weeks.ago}`

Checkout repository như trạng thái 2 tuần trước (ở chế độ detached HEAD).

### `git log @{1.month.ago}..HEAD`

Xem log các commit từ 1 tháng trước tới hiện tại.

### Các ví dụ khác

```
@{2024-06-01}
@{yesterday}
@{"1 week 2 days ago"}
```

---

## Blame

`git blame` cho biết lần sửa gần nhất của từng dòng trong file: commit nào, tác giả nào, lúc nào. Rất hữu ích để theo dõi lịch sử file, hiểu vì sao mã được viết như vậy và tìm nguồn gốc lỗi.

### `git blame <file>`

Hiển thị lần sửa gần nhất cho mỗi dòng của file.

### `git blame <file> -L <start>,<end>`

Chỉ blame trong khoảng dòng chỉ định.

```bash
git blame -L 10,25 src/app.js
```

### `git blame <file> <commit>`

Hiển thị thông tin blame tính tới commit chỉ định.

> ⚠️ **Lưu ý:** Cú pháp chuẩn là revision đứng **trước** file:
>
> ```bash
> git blame <commit> -- <file>
> ```

### `git blame <file> -C -C`

Hiển thị ai sửa từng dòng gần nhất, có phát hiện dòng bị **sao chép/di chuyển**.

> ⚠️ **Lưu ý:** Mô tả chính xác theo tài liệu Git:
>
> - `-M`: phát hiện dòng bị di chuyển/sao chép **trong cùng file**.
> - `-C`: thêm phát hiện dòng đến từ **các file khác bị sửa trong cùng commit**.
> - `-C -C`: tìm thêm trong các file **không bị sửa** ở commit tạo ra file.
> - `-C -C -C`: tìm trong **mọi commit**.

### `git blame <file> --reverse`

Blame theo chiều ngược: cho biết mỗi dòng **tồn tại tới** revision nào (lần cuối còn xuất hiện trước khi bị sửa hoặc xoá).

> ⚠️ **Lưu ý:** `--reverse` cần một khoảng revision: `git blame --reverse <start>..<end> -- <file>`.

### `git blame <file> --first-parent`

Hiển thị ai sửa từng dòng gần nhất, chỉ đi theo parent đầu tiên khi gặp merge commit.

---

[← Trước: Hoàn tác & Reflog](08-hoan-tac-va-reflog.md) · [Mục lục](README.md) · [Tiếp: Diff →](10-diff.md)
