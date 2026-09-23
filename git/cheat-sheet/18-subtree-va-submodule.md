# 18. Subtree & Submodule

[← Mục lục](README.md)

## Subtree

Git subtree là cơ chế tích hợp một dự án con vào repository chính. Khác với submodule (coi dự án con là thực thể riêng có repository riêng), subtree đưa **trực tiếp nội dung** của repository khác vào một thư mục con của repo chính. Quy trình đơn giản hơn: không phải quản lý nhiều repository, và vẫn merge hoặc pull cập nhật từ dự án con được. Subtree là cách linh hoạt để quản lý phụ thuộc và làm việc với các codebase bên ngoài.

> ⚠️ `git subtree` thuộc nhóm lệnh *contrib*. Hầu hết bản cài đặt đã có sẵn, nhưng một số bản phân phối Linux cần cài thêm.

### `git subtree add --prefix=<dir> <repository> <branch>`

Thêm một repository vào dưới dạng subtree.

```bash
git subtree add --prefix=vendor/ui-kit https://github.com/org/ui-kit.git main --squash
```

### `git subtree merge --prefix=<dir> <branch>`

Merge một subtree.

### `git subtree pull --prefix=<dir> <repository> <branch>`

Kéo thay đổi mới từ repository của subtree về.

```bash
git subtree pull --prefix=vendor/ui-kit https://github.com/org/ui-kit.git main --squash
```

> 💡 Đẩy thay đổi ngược lên repo của subtree: `git subtree push --prefix=<dir> <repository> <branch>`.

---

## Submodule

Submodule là cách đưa và quản lý repository bên ngoài bên trong repository của bạn. Rất hữu ích để dùng lại mã giữa nhiều dự án, quản lý phụ thuộc hoặc tích hợp thư viện bên thứ ba. Repository chính giữ được sự gọn gàng, tách biệt, trong khi mọi thành phần cần thiết vẫn được đưa vào và quản lý phiên bản (repo chính lưu **commit cụ thể** của từng submodule).

### `git submodule init`

Khởi tạo submodule trong repository: thiết lập cấu hình cần thiết nhưng **chưa** clone chúng về.

### `git submodule update`

Clone và checkout submodule vào đúng đường dẫn. Thường chạy sau `git submodule init`.

```bash
git submodule update --init --recursive   # gộp init + update, gồm cả submodule lồng nhau
git clone --recurse-submodules <url>      # clone repo kèm submodule ngay từ đầu
```

### `git submodule add <repository> <path>`

Thêm submodule mới vào repository tại đường dẫn chỉ định, liên kết với repository chỉ định.

```bash
git submodule add https://github.com/org/shared-lib.git libs/shared
git commit -m "Thêm submodule shared-lib"
```

### `git submodule status`

Hiển thị trạng thái mọi submodule: commit hash, và submodule đang đồng bộ, đã bị sửa hay chưa khởi tạo.

| Tiền tố | Ý nghĩa |
|---------|---------|
| (dấu cách) | Đang checkout đúng commit được ghi nhận |
| `-` | Chưa khởi tạo |
| `+` | Commit đang checkout khác commit được ghi nhận |
| `U` | Có xung đột merge |

### `git submodule foreach <command>`

Chạy lệnh chỉ định trong từng submodule. Tiện cho thao tác hàng loạt.

```bash
git submodule foreach 'git switch main && git pull'
```

### `git submodule sync`

Đồng bộ URL submodule trong cấu hình local theo file `.gitmodules`, đảm bảo URL luôn cập nhật.

### `git submodule deinit <path>`

Huỷ đăng ký submodule chỉ định, xoá cấu hình của nó.

> ⚠️ **Lưu ý:** Bản gốc nói lệnh này không xoá working directory của submodule. Thực tế `deinit` **xoá nội dung** working tree của submodule (chỉ còn thư mục rỗng), nhưng dữ liệu repo trong `.git/modules` vẫn giữ lại. Muốn gỡ hẳn submodule khỏi dự án:
>
> ```bash
> git submodule deinit -f libs/shared
> git rm -f libs/shared
> rm -rf .git/modules/libs/shared
> ```

### `git submodule update --remote`

Fetch và cập nhật submodule lên commit mới nhất của nhánh được theo dõi trên remote của chúng.

### `git submodule set-url <path> <newurl>`

Đổi URL của submodule chỉ định.

### `git submodule absorbgitdirs`

Chuyển thư mục `.git` của submodule vào trong thư mục `.git/modules` của repo cha (superproject) để đơn giản hoá cấu trúc.

---

[← Trước: Worktree](17-worktree.md) · [Mục lục](README.md)
