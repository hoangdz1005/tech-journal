# 17. Worktree

[← Mục lục](README.md)

Worktree cho phép có **nhiều working directory** gắn với cùng một repository. Nhờ đó bạn làm việc trên nhiều nhánh cùng lúc mà không phải chuyển nhánh liên tục trong một thư mục. Mỗi tính năng, bản sửa lỗi hay thử nghiệm có môi trường riêng biệt, giúp làm việc hiệu quả và giảm rủi ro xung đột.

```
~/code/my-app            ← worktree chính (main)
~/code/my-app-hotfix     ← worktree cho hotfix/1.2.1
~/code/my-app-feature    ← worktree cho feature/login
```

### `git worktree add ../new-branch feature-branch`

Tạo worktree mới tại thư mục `../new-branch`, checkout nhánh `feature-branch`.

```bash
git worktree add ../my-app-hotfix hotfix/1.2.1
git worktree add -b feature/new ../my-app-new main   # tạo nhánh mới từ main
```

> 💡 Một nhánh chỉ có thể được checkout ở **một** worktree tại một thời điểm.

### `git worktree list`

Liệt kê tất cả worktree của repository, kèm đường dẫn và nhánh đang checkout.

### `git worktree remove <path>`

Xoá worktree tại đường dẫn chỉ định (xoá cả thư mục làm việc đó).

> ⚠️ **Lưu ý:** Nhánh đã checkout trong worktree **không bị xoá**, chỉ được "giải phóng" để checkout ở nơi khác. Worktree có thay đổi chưa commit sẽ không xoá được, trừ khi thêm `--force`.

### `git worktree prune`

Xoá thông tin về các worktree không còn tồn tại (ví dụ thư mục đã bị xoá thủ công), dọn danh sách worktree.

### `git worktree lock <path>`

Khoá worktree tại đường dẫn chỉ định để nó không bị prune. Hữu ích khi worktree nằm trên ổ di động hoặc ổ mạng không phải lúc nào cũng kết nối.

```bash
git worktree lock --reason "Nằm trên ổ USB" ../my-app-usb
```

### `git worktree unlock <path>`

Mở khoá worktree, cho phép prune khi cần.

---

[← Trước: Git Flow](16-git-flow.md) · [Mục lục](README.md) · [Tiếp: Subtree & Submodule →](18-subtree-va-submodule.md)
