# 05. Sửa commit & Squash

[← Mục lục](README.md)

## Sửa commit (Amend)

Amend cho phép chỉnh sửa commit gần nhất, thường để sửa nội dung hoặc thông điệp. Lệnh `git commit --amend` mở commit trong trình soạn thảo mặc định để bạn chỉnh. Cách này tiện để sửa lỗi nhỏ hoặc thêm thay đổi bị quên mà không phải tạo commit mới, giúp lịch sử gọn và chính xác hơn.

> ⚠️ Amend tạo ra commit **mới** (hash khác). Nếu commit cũ đã push, bạn sẽ phải force push. Tránh làm vậy trên nhánh dùng chung.

### `git commit --amend`

Sửa commit gần nhất, gộp thêm các thay đổi đang được stage.

```bash
git add forgotten-file.js
git commit --amend --no-edit   # giữ nguyên thông điệp cũ
```

### `git commit --amend -m "new message"`

Đổi thông điệp của commit gần nhất.

### `git commit --fixup=HEAD`

Tạo commit mới dùng để sửa cho commit gần nhất (HEAD). Thông điệp của commit này có tiền tố `fixup!`. Khi chạy interactive rebase, commit này sẽ được tự động gộp vào commit cần sửa.

```bash
git commit --fixup=a1b2c3d
git rebase -i --autosquash a1b2c3d~1
```

---

## Squash

Squash là gộp nhiều commit thành một. Thường làm trước khi merge vào nhánh chính để lịch sử ngắn gọn, dễ đọc. Squash được thực hiện qua interactive rebase (`git rebase -i`), nơi bạn có thể gộp, sắp xếp lại hoặc sửa các commit. Nhờ đó những thay đổi lặt vặt được gom lại và lịch sử phát triển rõ ràng hơn.

### `git rebase -i HEAD~<n>`

Squash commit bằng interactive rebase, áp dụng cho `n` commit gần nhất.

```bash
git rebase -i HEAD~3
```

Trình soạn thảo sẽ hiện ra:

```
pick a1b2c3d Thêm form đăng nhập
pick d4e5f6a Sửa lỗi chính tả
pick 7b8c9d0 Sửa validate
```

Đổi `pick` thành `squash` (hoặc `s`) cho các commit muốn gộp vào commit phía trên:

```
pick a1b2c3d Thêm form đăng nhập
squash d4e5f6a Sửa lỗi chính tả
squash 7b8c9d0 Sửa validate
```

Dùng `fixup` (`f`) thay cho `squash` nếu muốn bỏ thông điệp của commit được gộp.

---

[← Trước: Remote](04-remote.md) · [Mục lục](README.md) · [Tiếp: Stash →](06-stash.md)
