# 01. Lệnh cơ bản, Working Directory & Staging Area

[← Mục lục](README.md)

## Lệnh Git cơ bản

Đây là các lệnh nền tảng để tạo, sao chép và kiểm tra trạng thái repository. Nắm vững nhóm lệnh này là bước đầu tiên để dùng Git hiệu quả.

### `git help`

In ra hướng dẫn sử dụng Git và danh sách các lệnh thường dùng. Hữu ích khi cần xem nhanh cách dùng hoặc tìm lệnh.

Xem hướng dẫn chi tiết cho một lệnh cụ thể:

```bash
git help <command>
git help commit      # hướng dẫn của lệnh commit
git help git         # hướng dẫn của chính lệnh git
```

### `git version`

Hiển thị phiên bản Git đang cài. Dùng khi cần kiểm tra tính tương thích của một tính năng hoặc khi gỡ lỗi.

```bash
git version
# git version 2.47.0
```

### `git init`

Khởi tạo một repository Git mới trong thư mục hiện tại. Lệnh này tạo thư mục con `.git` chứa toàn bộ metadata của repo. Thường là lệnh đầu tiên khi bắt đầu quản lý một dự án bằng Git.

```bash
mkdir my-project && cd my-project
git init
```

### `git clone <repository_url>`

Tạo bản sao của một repository từ xa về máy. Toàn bộ file, các nhánh và lịch sử commit đều được tải về, bạn có thể làm việc ngay.

```bash
git clone https://github.com/user/repo.git
git clone https://github.com/user/repo.git my-folder   # clone vào thư mục tên khác
```

### `git status`

Hiển thị trạng thái hiện tại của working directory và staging area: file nào đã sửa, thêm mới hay bị xoá, và thay đổi nào đã được stage cho commit tiếp theo.

```bash
git status
git status -s   # dạng rút gọn
```

---

## Working Directory và Staging Area

**Working directory** là nơi bạn trực tiếp sửa, tạo, xoá file. Nó phản ánh trạng thái hiện tại của dự án. Các thay đổi ở đây chỉ nằm trên máy bạn và chưa thuộc lịch sử phiên bản.

**Staging area** (còn gọi là **index**) là vùng trung gian giữa working directory và repository. Tại đây bạn chọn lọc những thay đổi sẽ đưa vào commit tiếp theo. Nhờ vậy mỗi commit có thể chỉ gồm các thay đổi liên quan với nhau, giúp lịch sử rõ ràng, dễ theo dõi.

```
Working directory  --git add-->  Staging area  --git commit-->  Repository
```

### `git checkout .`

Huỷ toàn bộ thay đổi **chưa stage** trong working directory, đưa file về trạng thái trước khi sửa. Dùng để nhanh chóng bỏ các chỉnh sửa local.

> ⚠️ **Lưu ý:** Chính xác thì lệnh khôi phục file theo nội dung trong **index** (nếu chưa stage gì thì trùng với commit gần nhất). Thao tác này **không thể hoàn tác**. Từ Git 2.23 có lệnh tương đương, rõ nghĩa hơn: `git restore .`

### `git reset -p`

Chọn tương tác từng phần thay đổi (hunk) để **bỏ khỏi staging area**, giúp kiểm soát chi tiết phần nào giữ lại, phần nào bỏ.

> ⚠️ **Lưu ý:** Lệnh này chỉ tác động lên **index** (unstage), không sửa nội dung file trong working directory. Tương đương: `git restore --staged -p`.

### `git add <file>`

Thêm một file vào staging area để đưa vào commit tiếp theo. Cho phép chọn chính xác thay đổi nào sẽ vào lịch sử.

```bash
git add index.html
git add src/          # thêm cả thư mục
git add .             # thêm mọi thay đổi trong thư mục hiện tại
```

### `git add -p`

Stage thay đổi theo từng khối (hunk). Git hiển thị từng khối và hỏi bạn có muốn đưa vào index hay không, giúp review và chọn lọc trước khi commit.

Các phím thường dùng: `y` (stage), `n` (bỏ qua), `s` (chia nhỏ khối), `q` (thoát).

### `git add -i`

Vào chế độ add tương tác: một menu dạng văn bản cho phép stage từng thay đổi, cập nhật file, xem trạng thái...

### `git rm <file>`

Xoá file khỏi working directory và stage luôn việc xoá đó.

### `git rm --cached <file>`

Xoá file khỏi staging area (ngừng theo dõi) nhưng **giữ nguyên** file trong working directory. Thường dùng khi lỡ commit một file lẽ ra phải nằm trong `.gitignore`.

```bash
git rm --cached .env
echo ".env" >> .gitignore
```

### `git mv <old_path> <new_path>`

Di chuyển hoặc đổi tên file/thư mục trong repository. Thay đổi được stage tự động, sẵn sàng cho commit tiếp theo.

```bash
git mv README.txt README.md
```

### `git commit -m "message"`

Tạo commit mới từ các thay đổi đã stage, kèm thông điệp mô tả. Thông điệp nên giải thích ngắn gọn những gì đã thay đổi.

```bash
git commit -m "Thêm trang đăng nhập"
```

---

[← Mục lục](README.md) · [Tiếp: Branch & Checkout →](02-branch.md)
