# 03. Merge & xử lý xung đột

[← Mục lục](README.md)

## Merge

`git merge` gộp thay đổi từ một nhánh vào nhánh khác. Lệnh này kết hợp lịch sử của cả hai nhánh, thường tạo ra một commit mới chứa thay đổi từ hai phía. Nhờ đó nhiều hướng phát triển có thể được gom lại vào dự án chính. Nếu thay đổi ở hai bên chồng lên nhau sẽ phát sinh **xung đột** (conflict), cần xử lý thủ công.

### `git merge <branch>`

Tích hợp thay đổi từ nhánh chỉ định vào nhánh hiện tại, kết hợp lịch sử của hai nhánh.

```bash
git switch main
git merge feature/login
```

### `git merge --no-ff <branch>`

Merge nhánh chỉ định vào nhánh hiện tại và **luôn tạo merge commit**, kể cả khi có thể fast-forward. Giúp lịch sử thể hiện rõ một nhánh tính năng đã được gộp vào.

### `git merge --squash <branch>`

Gom tất cả thay đổi từ nhánh chỉ định thành một bộ thay đổi duy nhất trong nhánh hiện tại mà không gộp lịch sử của nhánh đó. Kết quả được stage sẵn để bạn tự commit và viết thông điệp.

```bash
git merge --squash feature/login
git commit -m "Thêm tính năng đăng nhập"
```

### `git merge --abort`

Huỷ quá trình merge đang dở (thường do xung đột), đưa working directory và index về trạng thái trước khi merge.

### `git merge -s ours <branch>` / `git merge --strategy=ours <branch>`

Merge bằng chiến lược **"ours"**: giữ nguyên nội dung nhánh hiện tại, bỏ toàn bộ thay đổi từ nhánh kia. Lịch sử hai nhánh được nối lại nhưng nội dung nhánh kia không được tích hợp.

### `git merge --strategy=theirs <branch>`

Bản gốc mô tả đây là merge theo chiến lược "theirs", tức ưu tiên thay đổi của nhánh được merge khi có xung đột, và ghi chú rằng "theirs" không phải chiến lược có sẵn.

> ⚠️ **Lưu ý:** Git **không có** chiến lược `-s theirs`, lệnh trên sẽ báo lỗi. Cách đúng là dùng *tuỳ chọn* của chiến lược mặc định:
>
> ```bash
> git merge -X theirs <branch>   # khi xung đột, ưu tiên phía nhánh được merge
> git merge -X ours <branch>     # khi xung đột, ưu tiên phía nhánh hiện tại
> ```
>
> Khác với `-s ours`, tuỳ chọn `-X` chỉ quyết định **những chỗ xung đột**. Các thay đổi không xung đột từ hai phía vẫn được gộp.

---

## Xử lý xung đột khi merge

Xung đột xảy ra khi thay đổi ở các nhánh hoặc commit chồng chéo, mâu thuẫn nhau khiến Git không tự merge được. Bạn cần xem xét và tự quyết định nội dung cuối cùng sao cho giữ đúng đóng góp của các bên. Xử lý xung đột tốt giúp mã nguồn nhất quán và làm việc nhóm suôn sẻ.

Quy trình thường gặp:

```bash
git merge feature/login        # báo CONFLICT
git status                     # xem file nào xung đột
# sửa các đoạn giữa <<<<<<< ======= >>>>>>> trong file
git add <file>                 # đánh dấu đã giải quyết
git commit                     # hoàn tất merge
```

### `git mergetool`

Mở công cụ merge để giải quyết xung đột phát sinh khi merge hoặc rebase. Có thể là giao diện đồ hoạ hoặc công cụ dạng văn bản, tuỳ cấu hình Git của bạn.

```bash
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'
git mergetool
```

### `git rerere`

**rerere** là viết tắt của *"reuse recorded resolution"* (dùng lại cách giải quyết đã ghi nhận). Khi bật, Git ghi lại cách bạn giải quyết xung đột. Nếu cùng xung đột đó xuất hiện lại trong lần merge hoặc rebase sau, Git tự áp dụng lại cách giải quyết cũ.

```bash
git config --global rerere.enabled true
```

---

[← Trước: Branch & Checkout](02-branch.md) · [Mục lục](README.md) · [Tiếp: Remote →](04-remote.md)
