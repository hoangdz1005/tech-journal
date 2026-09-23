# 12. Cấu hình, bảo mật, alias, attributes

[← Mục lục](README.md)

## Cấu hình

Cấu hình Git là thiết lập các tuỳ chọn điều khiển cách Git hoạt động: tên và email người dùng, trình soạn thảo mặc định, alias cho lệnh hay dùng, file ignore toàn cục... Cấu hình có thể áp dụng ở ba cấp:

| Cấp | Tuỳ chọn | Phạm vi | Vị trí file |
|-----|----------|---------|-------------|
| System | `--system` | Mọi người dùng trên máy | `/etc/gitconfig` |
| Global | `--global` | Mọi repo của người dùng hiện tại | `~/.gitconfig` |
| Local | `--local` (mặc định) | Một repository | `.git/config` |

Cấp hẹp hơn ghi đè cấp rộng hơn (local > global > system).

### `git config --global user.name "Your Name"`

Đặt tên người dùng ở cấp global.

### `git config --global user.email "your_email@example.com"`

Đặt email người dùng ở cấp global.

### `git config --global core.editor <editor>`

Đặt trình soạn thảo mặc định.

```bash
git config --global core.editor "code --wait"
git config --global core.editor vim
```

### `git config --global core.excludesfile <file>`

Đặt file ignore toàn cục (áp dụng cho mọi repository).

```bash
git config --global core.excludesfile ~/.gitignore_global
```

### `git config --list`

Liệt kê tất cả cấu hình.

### `git config --list --show-origin`

Liệt kê tất cả cấu hình kèm file nguồn của từng giá trị.

### `git config <key>`

Lấy giá trị của một khoá.

### `git config --get <key>`

Lấy giá trị của một khoá cấu hình.

```bash
git config --get user.email
```

### `git config --unset <key>`

Xoá một khoá cấu hình.

### `git config --global --unset <key>`

Xoá một khoá cấu hình ở cấp global.

---

## Bảo mật (ký GPG)

Git dùng GNU Privacy Guard (GPG) để ký commit và tag, đảm bảo tính xác thực và toàn vẹn. Khi cấu hình khoá GPG và bật ký tự động, mọi người có thể xác minh commit/tag được tạo bởi người đáng tin cậy, chống giả mạo và đảm bảo lịch sử repository không bị can thiệp.

### `git config --global user.signingKey <key>`

Cấu hình khoá GPG dùng để ký commit và tag.

```bash
gpg --list-secret-keys --keyid-format=long   # tìm ID khoá
git config --global user.signingKey 3AA5C34371567BD2
```

### `git config --global commit.gpgSign true`

Tự động ký GPG cho mọi commit.

> 💡 Tương tự cho tag: `git config --global tag.gpgSign true`. Kiểm tra chữ ký bằng `git log --show-signature`.

---

## Alias

Alias là lối tắt tự đặt cho các lệnh Git dài. Cấu hình alias giúp gõ ít hơn, nhanh hơn và ít sai hơn. Ví dụ `git st` thay cho `git status`, `git co` thay cho `git checkout`. Alias có thể đặt global cho mọi repository hoặc local cho từng dự án.

### `git config --global alias.ci commit`

Đặt `git ci` làm alias cho `git commit`.

### `git config --global alias.st status`

Đặt `git st` làm alias cho `git status`.

### `git config --global alias.co checkout`

Đặt `git co` làm alias cho `git checkout`.

### `git config --global alias.br branch`

Đặt `git br` làm alias cho `git branch`.

### `git config --global alias.graph "log --graph --all --oneline --decorate"`

Tạo alias `git graph` để xem đồ thị lịch sử chi tiết.

---

## Attributes

Git attributes là các thiết lập quy định cách Git xử lý file hoặc đường dẫn cụ thể. Chúng được khai báo trong file `.gitattributes` và điều khiển mã hoá văn bản, chuẩn hoá ký tự xuống dòng, chiến lược merge, thuật toán diff... Nhờ đó Git hoạt động nhất quán trên mọi môi trường và giữa mọi người. Ví dụ, đánh dấu file là binary để Git không cố merge, hoặc chỉ định diff driver riêng để so sánh dễ hiểu hơn.

```gitattributes
# .gitattributes
* text=auto
*.sh text eol=lf
*.png binary
package-lock.json -diff merge=ours
```

### `git check-attr <attribute> -- <file>`

Hiển thị giá trị của một attribute cho file chỉ định theo cấu hình `.gitattributes`. Giúp hiểu Git đang xử lý file thế nào về mã hoá văn bản, merge hay diff.

```bash
git check-attr text eol -- scripts/build.sh
git check-attr -a -- image.png   # tất cả attribute
```

---

[← Trước: Rebase, Cherry-pick, Patch](11-rebase-cherry-pick-patch.md) · [Mục lục](README.md) · [Tiếp: Tham chiếu & index →](13-tham-chieu-va-index.md)
