# 08. Hoàn tác thay đổi & Reflog

[← Mục lục](README.md)

## Hoàn tác thay đổi

Git có nhiều cách để hoàn tác thay đổi:

- `git revert` tạo commit mới đảo ngược thay đổi của một commit trước đó, **giữ nguyên** lịch sử.
- `git reset` di chuyển HEAD về commit chỉ định. Tuỳ tuỳ chọn (`--soft`, `--mixed`, `--hard`) mà staging area và working directory có bị cập nhật theo hay không.
- `git checkout` (hoặc `git restore`) dùng để bỏ thay đổi trong working directory, đưa file về trạng thái của commit gần nhất.

Nhờ các công cụ này bạn có thể sửa sai linh hoạt, giữ repository chính xác và gọn gàng.

### `git checkout -- <file>`

Bỏ thay đổi **chưa stage** của file chỉ định trong working directory, đưa file về trạng thái commit gần nhất.

```bash
git restore <file>   # cách viết tương đương, mới hơn
```

### `git revert <commit>`

Tạo commit mới hoàn tác thay đổi của commit chỉ định, giữ nguyên lịch sử. Đây là cách an toàn để hoàn tác trên nhánh đã push và dùng chung.

```bash
git revert a1b2c3d
git revert HEAD      # hoàn tác commit gần nhất
```

### `git revert -n <commit>`

Hoàn tác một commit nhưng **không** tự tạo commit. Thay đổi được để sẵn trong index và working directory, bạn có thể gom nhiều revert rồi commit một lần.

### `git reset`

Đặt lại HEAD hiện tại về trạng thái chỉ định. Tuỳ tuỳ chọn (`--soft`, `--mixed`, `--hard`) mà staging area và working directory có thể được cập nhật theo. Chạy không tham số tương đương `git reset --mixed HEAD`, tức là bỏ stage tất cả.

### `git reset --soft <commit>`

Chuyển HEAD về commit chỉ định, **giữ nguyên** index và working directory. Mọi thay đổi sau commit đó vẫn ở trạng thái đã stage, sẵn sàng commit lại. Hữu ích khi muốn huỷ commit nhưng vẫn giữ thay đổi.

```bash
git reset --soft HEAD~1   # huỷ commit gần nhất, giữ thay đổi đã stage
```

### `git reset --mixed <commit>`

Chuyển HEAD về commit chỉ định và cập nhật index theo commit đó nhưng **giữ nguyên** working directory. Các thay đổi sau commit đó vẫn còn nhưng không được stage. Đây là chế độ mặc định của `git reset`.

> ⚠️ **Lưu ý:** Bản gốc ghi thay đổi trở thành "untracked", nhưng chính xác là chúng thành **unstaged** (chưa stage). File vẫn được Git theo dõi, trừ file chỉ mới được thêm sau commit đích.

### `git reset --hard <commit>`

Chuyển HEAD về commit chỉ định và cập nhật cả index lẫn working directory theo commit đó, **bỏ toàn bộ thay đổi** sau commit đó.

> ⚠️ **Lưu ý:** `--hard` **không xoá file untracked** (muốn xoá phải dùng `git clean`). Thay đổi chưa commit bị mất sẽ không lấy lại được. Commit bị "bỏ" thì vẫn có thể tìm lại qua `git reflog`.

| Chế độ | HEAD | Index (staging) | Working directory |
|--------|------|-----------------|-------------------|
| `--soft` | Di chuyển | Giữ nguyên | Giữ nguyên |
| `--mixed` (mặc định) | Di chuyển | Đặt lại | Giữ nguyên |
| `--hard` | Di chuyển | Đặt lại | Đặt lại |

---

## Reflog

Reflog ghi lại mọi thay đổi của HEAD và đầu các nhánh trong repository local: commit, checkout, merge, reset... Nhờ lịch sử này, bạn có thể theo dõi các thao tác gần đây và **khôi phục commit bị mất**, kể cả khi chúng không còn thuộc lịch sử nhánh nào. Đây là công cụ rất giá trị để gỡ lỗi và sửa sai.

### `git reflog`

Hiển thị nhật ký thay đổi của HEAD và đầu các nhánh (commit, checkout, merge, reset), giúp khôi phục commit bị mất hoặc theo dõi các thay đổi gần đây.

```bash
git reflog
# a1b2c3d HEAD@{0}: reset: moving to HEAD~2
# 9f8e7d6 HEAD@{1}: commit: Thêm API đăng nhập
# ...

# Khôi phục sau khi lỡ reset --hard:
git reset --hard HEAD@{1}
```

### `git reflog show <ref>`

Hiển thị reflog của tham chiếu chỉ định (ví dụ một nhánh), gồm các lần cập nhật cùng thông điệp và thời gian.

```bash
git reflog show main
```

> 💡 Reflog chỉ tồn tại **ở local** và các mục cũ sẽ hết hạn (mặc định 90 ngày, 30 ngày với commit không còn truy cập được).

---

[← Trước: Tag](07-tag.md) · [Mục lục](README.md) · [Tiếp: Lịch sử →](09-lich-su.md)
