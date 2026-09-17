# 04. Remote, Fetch/Pull, Force Push

[← Mục lục](README.md)

## Remote

Remote là tham chiếu tới các repository từ xa, tức các phiên bản của dự án được lưu trên internet hoặc mạng nội bộ. Remote giúp nhiều người chia sẻ và đồng bộ thay đổi qua một repository trung tâm. Thao tác phổ biến: `git fetch` để lấy cập nhật, `git pull` để lấy và merge, `git push` để đẩy commit local lên remote. Ngoài ra còn có thêm, xoá, đổi tên remote và cấu hình URL.

### `git fetch`

Tải thay đổi từ remote nhưng **không** merge vào nhánh hiện tại.

### `git pull`

Tải thay đổi từ remote và merge ngay vào nhánh hiện tại.

### `git push`

Đẩy thay đổi của nhánh local lên remote.

### `git remote`

Liệt kê tên các remote đã cấu hình cho repository local.

### `git remote -v`

Hiển thị URL của các remote, gồm cả URL fetch và URL push.

```bash
git remote -v
# origin  git@github.com:user/repo.git (fetch)
# origin  git@github.com:user/repo.git (push)
```

### `git remote add <name> <url>`

Thêm remote mới với tên và URL chỉ định vào cấu hình của repo local.

### `git remote remove <name>` / `git remote rm <name>`

Xoá kết nối tới remote chỉ định khỏi cấu hình Git local.

### `git remote rename <old_name> <new_name>`

Đổi tên một remote đã có.

### `git remote set-url <name> <newurl>`

Đổi URL của một remote đã có.

```bash
git remote set-url origin git@github.com:user/new-repo.git
```

### `git fetch <remote>`

Lấy thay đổi mới nhất từ remote chỉ định, cập nhật các nhánh remote-tracking ở local mà không merge vào nhánh local.

### `git pull <remote>`

Lấy thay đổi từ remote chỉ định và merge vào nhánh hiện tại.

### `git remote update`

Fetch cập nhật cho tất cả remote của repository.

### `git push <remote> <branch>`

Đẩy nhánh chỉ định từ local lên remote.

### `git push <remote> --delete <branch>`

Xoá nhánh chỉ định trên remote.

### `git remote show <remote>`

Hiển thị thông tin chi tiết về remote: URL, cấu hình fetch/push và các nhánh đang theo dõi.

### `git ls-remote <repository>`

Liệt kê các tham chiếu (nhánh, tag) cùng commit ID của một remote. Cho phép xem nhánh và tag trên remote mà không cần clone.

### `git push origin <branch> --set-upstream`

Đẩy nhánh local lên remote `origin` và thiết lập theo dõi (upstream). Các lần `git push` / `git pull` sau sẽ mặc định làm việc với nhánh remote này.

```bash
git push -u origin feature/login   # dạng viết tắt
```

### `git remote add upstream <repository>`

Thêm remote tên `upstream` trỏ tới repository chỉ định. Thường dùng khi làm việc với fork: `upstream` là repo gốc, còn `origin` là bản fork của bạn.

### `git fetch upstream`

Lấy cập nhật từ remote `upstream`, cập nhật tham chiếu nhánh và tag của remote đó ở local mà không sửa working directory hay merge.

### `git pull upstream <branch>`

Lấy cập nhật từ nhánh chỉ định của `upstream` và merge vào nhánh hiện tại. Thường dùng để đồng bộ thay đổi từ repo gốc vào nhánh của mình.

```bash
git switch main
git pull upstream main
git push origin main
```

### `git push origin <branch>`

Đẩy nhánh local lên remote `origin`, để nhánh và các commit của nó có trên remote.

---

## Fetch và Pull

### `git fetch --all`

Lấy cập nhật từ **tất cả** remote đã cấu hình, gồm mọi nhánh và tag, mà không sửa các nhánh local.

### `git pull --rebase`

Lấy thay đổi từ remote rồi **rebase** các commit local lên trên nhánh remote đã cập nhật, thay vì merge. Lịch sử commit giữ được dạng tuyến tính và không sinh merge commit thừa.

```bash
git config --global pull.rebase true   # đặt làm mặc định cho git pull
```

---

## Force Push

### `git push --force`

Ép đẩy nhánh local lên remote kể cả khi đó không phải fast-forward, **ghi đè** nhánh remote bằng nhánh local. Cần thiết khi bạn đã viết lại lịch sử (ví dụ sau rebase) và muốn remote khớp với local. Tuy nhiên lệnh này có thể xoá mất thay đổi của người khác, nên phải dùng cẩn thận.

> 💡 **An toàn hơn:** dùng `git push --force-with-lease`. Lệnh này chỉ ghi đè nếu nhánh remote vẫn đúng như lần cuối bạn fetch, tránh vô tình xoá commit người khác vừa đẩy lên.

---

[← Trước: Merge & xung đột](03-merge-va-xung-dot.md) · [Mục lục](README.md) · [Tiếp: Sửa commit & Squash →](05-amend-va-squash.md)
