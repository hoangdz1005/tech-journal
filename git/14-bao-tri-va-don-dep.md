# 14. Toàn vẹn dữ liệu, dọn dẹp & đóng gói

[← Mục lục](README.md)

## Toàn vẹn dữ liệu

Git có các cơ chế đảm bảo dữ liệu trong repository chính xác và nhất quán. Mỗi đối tượng (commit, tree, blob) được định danh bằng mã băm mật mã (SHA-1 hoặc SHA-256). Mã băm vừa là định danh duy nhất, vừa đảm bảo mọi thay đổi nội dung đều sinh ra hash khác, nhờ đó phát hiện được dữ liệu hỏng hoặc bị can thiệp. Lệnh như `git fsck` kiểm tra tính liên kết và hợp lệ của các đối tượng, đảm bảo repository "khoẻ mạnh".

### `git fsck`

Kiểm tra tính liên kết và hợp lệ của các đối tượng trong cơ sở dữ liệu Git.

### `git fsck --unreachable`

Tìm các đối tượng không thể truy cập từ bất kỳ tham chiếu nào.

> 💡 Hữu ích để tìm lại commit/stash bị mất: `git fsck --lost-found`.

### `git prune`

Xoá các đối tượng không còn truy cập được.

> ⚠️ Thường không cần gọi trực tiếp, `git gc` đã tự chạy prune.

### `git gc`

Chạy quá trình thu gom rác (garbage collection).

Đây là quá trình bảo trì giúp dọn dẹp và tối ưu repository: xoá file không cần thiết, nén các phiên bản file để tiết kiệm dung lượng. `git gc` gom và xoá các đối tượng không truy cập được (commit mồ côi, blob không được tham chiếu), giúp repository gọn nhẹ và chạy nhanh. Chạy định kỳ giúp quản lý dung lượng tốt và giữ cấu trúc repository gọn gàng.

```bash
git gc
git gc --aggressive --prune=now   # tối ưu mạnh hơn (chậm)
```

---

## Dọn dẹp

Dọn dẹp là loại bỏ file, tham chiếu và nhánh không còn cần. Các việc như prune nhánh remote-tracking, xoá file untracked, xoá tham chiếu cũ giúp repository gọn gàng, cải thiện hiệu năng, giảm dung lượng và dễ làm việc hơn.

### `git fetch --prune`

Xoá các tham chiếu (nhánh remote-tracking) không còn tồn tại trên remote.

```bash
git config --global fetch.prune true   # luôn prune khi fetch
```

### `git remote prune <name>`

Xoá tất cả nhánh remote-tracking đã lỗi thời của remote chỉ định.

### `git fetch origin --prune`

Dọn các tham chiếu lỗi thời của remote `origin`.

### `git clean -f`

Xoá các file untracked (không được Git theo dõi) khỏi working directory.

### `git clean -fd`

Xoá cả file **và thư mục** untracked khỏi working directory.

### `git clean -i`

Vào chế độ tương tác để chọn file untracked cần xoá.

### `git clean -X`

Chỉ xoá các file **bị ignore** khỏi working directory.

> ⚠️ `X` viết hoa và viết thường khác nhau: `-X` chỉ xoá file bị ignore, còn `-x` xoá **cả** file untracked lẫn file bị ignore. Luôn thêm `-f` (hoặc chạy thử với `-n`).

---

## Xử lý file untracked

### `git clean`

Xoá file và thư mục untracked khỏi working directory. Kèm các cờ:

- `git clean -f`: xoá file untracked.
- `git clean -fd`: xoá file và thư mục untracked.
- `git clean -fx`: xoá file untracked, **kể cả** file bị `.gitignore` bỏ qua.
- `git clean -n`: chỉ hiển thị những gì sẽ bị xoá mà không xoá thật (dry run).

> ⚠️ **Lưu ý:** Bản gốc nói `git clean` không kèm cờ sẽ chỉ "hiển thị những gì sẽ bị xoá". Thực tế, với cấu hình mặc định (`clean.requireForce=true`), `git clean` không cờ sẽ **báo lỗi và từ chối chạy**. Muốn xem trước hãy dùng `git clean -n`.
>
> ```bash
> git clean -nd    # xem trước
> git clean -fd    # xoá thật
> ```

---

## Đóng gói (Archive)

`git archive` tạo file nén (`.tar`, `.zip`) chứa nội dung của một commit, nhánh hoặc tag. Hữu ích khi cần đóng gói "ảnh chụp" repository tại một thời điểm để phân phối hoặc sao lưu mà không kèm lịch sử Git.

### `git archive <format> <tree-ish>`

Tạo file lưu trữ chứa nội dung của tree-ish (commit, nhánh hoặc tag) theo định dạng chỉ định. Ví dụ:

- `git archive --format=tar HEAD`: tạo archive `.tar` của commit hiện tại (HEAD).
- `git archive --format=zip v1.0`: tạo archive `.zip` các file của tag `v1.0`.

> 💡 Mặc định kết quả được ghi ra stdout, nên cần chỉ định file đầu ra:
>
> ```bash
> git archive --format=zip -o release-v1.0.zip v1.0
> git archive --format=tar.gz --prefix=myapp/ -o myapp.tar.gz HEAD
> ```

---

[← Trước: Tham chiếu & index](13-tham-chieu-va-index.md) · [Mục lục](README.md) · [Tiếp: Tìm kiếm & Bisect →](15-tim-kiem-va-bisect.md)
