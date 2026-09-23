# 04. Index và mô hình ba cây

[← Lộ trình](../README.md) · Giai đoạn 1: Nền tảng · ⏱️ ~45 phút

## 🎯 Mục tiêu

- Biết chính xác index chứa gì.
- Nắm **mô hình ba cây** (HEAD, index, working tree), công cụ tư duy quan trọng nhất để hiểu `add`, `commit`, `diff`, `restore`, `reset`, `switch`.
- Đọc `git status` như hai phép so sánh.
- Hiểu vì sao `git status` chạy nhanh dù repo có hàng chục nghìn file.

## ❓ Vấn đề

Bài 02 cho thấy commit trỏ tới tree, và `git write-tree` tạo tree **từ index**. Vậy index là gì, nó khác working tree thế nào, và vì sao Git cần thêm một lớp ở giữa?

## 🧠 Khái niệm

### Index là một danh sách phẳng

Index (`.git/index`) là file nhị phân chứa **danh sách phẳng** mọi file sẽ có trong commit tiếp theo. Mỗi mục gồm:

- **Đường dẫn** đầy đủ (`src/b.txt`, không lồng thư mục như tree).
- **Mode** (`100644`, `100755`...).
- **Hash blob** của nội dung.
- **Stage number**: `0` bình thường; `1`, `2`, `3` khi có conflict (bài 08).
- **Thông tin stat** của file trên đĩa: mtime, ctime, size, inode... (xem phần "Bên trong").

```bash
git ls-files --stage
# 100644 ce013625030ba8dba906f756967f9e9ca394464a 0	a.txt
# 100644 587be6b4c3f93f93c489c0111bba5596147a26cb 0	src/b.txt
```

Điểm mấu chốt: **index không chứa nội dung file**, chỉ chứa **hash** của blob. Nội dung đã nằm trong object database từ lúc `git add`.

> 💡 Ngay sau một commit (khi chưa sửa gì), index **giống hệt** tree của commit đó, chỉ khác ở dạng phẳng. Index không phải "vùng chứa thay đổi", nó là **toàn bộ snapshot tiếp theo**. Một file không sửa gì vẫn có mặt trong index.

### Mô hình ba cây

Git luôn làm việc với ba "cây" (ba phiên bản của toàn bộ dự án):

| Cây | Là gì | Vai trò |
|-----|-------|---------|
| **HEAD** | Tree của commit mà HEAD trỏ tới | Snapshot đã commit gần nhất |
| **Index** | `.git/index` | Snapshot dự kiến cho commit tiếp theo |
| **Working tree** | File thật trên đĩa | Sân chơi của bạn |

```
    HEAD                    Index                  Working tree
┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│ a.txt  → v1   │      │ a.txt  → v2   │      │ a.txt  = v3   │
│ b.txt  → x    │      │ b.txt  → x    │      │ b.txt  = x    │
│               │      │               │      │ c.txt  = new  │
└───────────────┘      └───────────────┘      └───────────────┘
        ▲                      ▲                      ▲
        └──── git diff ────────┼──── git diff ────────┘
             --staged          │     (không tham số)
        ◄──────────────────────┴── git diff HEAD ────►
```

Mọi lệnh làm việc với file đều có thể mô tả bằng câu: **"chép nội dung từ cây nào sang cây nào"**.

| Lệnh | Chép từ | Sang |
|------|---------|------|
| `git add <f>` | Working tree | Index |
| `git commit` | Index | HEAD (tạo commit mới, di chuyển nhánh) |
| `git restore <f>` | Index | Working tree |
| `git restore --staged <f>` | HEAD | Index |
| `git restore --source=<c> --staged --worktree <f>` | Commit `<c>` | Index và working tree |
| `git reset --soft <c>` | (chỉ di chuyển nhánh) | HEAD |
| `git reset --mixed <c>` | Commit `<c>` | HEAD, index |
| `git reset --hard <c>` | Commit `<c>` | HEAD, index, working tree |
| `git switch <nhánh>` | Commit của nhánh | HEAD (đổi nhánh), index, working tree |

Học thuộc bảng này thì không cần học thuộc lệnh nữa. Bài 06 và 09 đi sâu vào từng dòng.

### `git status` là hai phép so sánh

`git status` không đọc một "danh sách thay đổi" nào cả. Nó **tính** bằng cách so sánh:

1. **HEAD với Index** → "Changes to be committed" (cột trái trong `--short`).
2. **Index với Working tree** → "Changes not staged for commit" (cột phải).
3. File trong working tree nhưng **không có trong index** và không bị ignore → "Untracked files".

```
git status --short
 XY đường-dẫn
 │└─ Index vs Working tree
 └── HEAD  vs Index
```

| Mã | Ý nghĩa |
|----|---------|
| `M ` | Đã sửa và đã stage toàn bộ |
| ` M` | Đã sửa, chưa stage |
| `MM` | Đã stage một phiên bản, rồi sửa tiếp |
| `A ` | File mới, đã stage |
| `AM` | File mới đã stage, rồi sửa tiếp |
| `D ` | Đã stage việc xoá (`git rm`) |
| ` D` | Xoá trên đĩa nhưng chưa stage |
| `R ` | Đổi tên (Git **đoán** từ một cặp xoá + thêm có nội dung giống nhau) |
| `??` | Untracked |
| `UU` | Conflict, cả hai bên đều sửa (bài 08) |

## 🔬 Bên trong Git: vì sao `git status` nhanh

Để so index với working tree, cách ngây thơ là đọc và hash lại **mọi file**. Với repo lớn, việc này rất chậm. Git dùng **stat cache**: index lưu lại mtime, ctime, size, inode của mỗi file tại thời điểm hash.

```bash
git ls-files --debug a.txt
# a.txt
#   ctime: 1790158442:266458960
#   mtime: 1790158442:266458960
#   dev: 16777230	ino: 99196603
#   uid: 501	gid: 0
#   size: 11	flags: 0
```

Khi chạy `status`, Git gọi `stat()` cho từng file (rất rẻ). Thông tin stat khớp với index thì Git **coi như file không đổi**, không cần đọc nội dung. Chỉ những file stat khác mới bị hash lại.

Hệ quả thú vị:

- `touch a.txt` (đổi mtime, không đổi nội dung) khiến Git phải hash lại file đó. Kết quả vẫn là "không đổi", và `git status` sẽ tiện tay cập nhật stat mới vào index (*refresh index*).
- Đó là lý do đôi khi `git status`, một lệnh "chỉ đọc", lại ghi vào `.git/index`.
- Repo khổng lồ có thể bật thêm `core.fsmonitor` để hệ điều hành báo file nào đã đổi, khỏi phải `stat()` toàn bộ.

## 🧪 Thực hành

```bash
cd ~/git-lab && rm -rf trees && git init trees && cd trees
echo v1 > a.txt && echo x > b.txt
git add . && git commit -m "init"

# 1. Index = toàn bộ snapshot, không chỉ thay đổi
git ls-files --stage          # cả a.txt lẫn b.txt, dù không có gì thay đổi

# 2. Tạo ba phiên bản khác nhau của a.txt
echo v2 > a.txt && git add a.txt
echo v3 > a.txt
git status --short            # MM a.txt

# 3. Xem từng cây
git show HEAD:a.txt           # v1   (cây HEAD)
git show :a.txt               # v2   (cây index, cú pháp ":<path>")
cat a.txt                     # v3   (working tree)

# 4. Ba kiểu diff tương ứng ba cặp cây
git diff                      # index → working tree: v2 → v3
git diff --staged             # HEAD → index:         v1 → v2
git diff HEAD                 # HEAD → working tree:  v1 → v3

# 5. commit lấy từ index, không phải từ working tree
git commit -m "commit v2"
git show HEAD:a.txt           # v2
git status --short            #  M a.txt   (v3 vẫn chưa stage)

# 6. Chép ngược: index → working tree
git restore a.txt
cat a.txt                     # v2

# 7. Rename chỉ là suy luận
git mv b.txt c.txt
git status --short            # R  b.txt -> c.txt
git ls-files --stage          # chỉ thấy c.txt, không có thông tin "rename" nào
```

Thử thêm: sửa `a.txt` rồi chạy `git add -p a.txt` để stage chỉ **một phần** file. Sau đó xem `git show :a.txt` để thấy index giữ một phiên bản "lai" chưa từng tồn tại trên đĩa.

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "Staging area chỉ chứa file đã thay đổi" | Index chứa **mọi** file tracked. Nó là snapshot đầy đủ cho commit tiếp theo. |
| "`git add` đánh dấu file để commit sau" | `add` hash nội dung **ngay lúc đó** thành blob và ghi hash vào index. |
| "`git diff` cho thấy mọi thay đổi chưa commit" | `git diff` không tham số chỉ so index với working tree. Thay đổi đã stage cần `git diff --staged`. |
| "File tracked = file đã từng commit" | File tracked = file **có trong index**. File vừa `add` lần đầu đã là tracked dù chưa commit. |
| "`git status` chỉ đọc" | Nó có thể refresh stat cache trong index. |

## ✅ Tự kiểm tra

<details>
<summary>1. Ngay sau <code>git commit</code>, nói gì về quan hệ giữa HEAD và index?</summary>

Chúng chứa cùng một snapshot. `git diff --staged` rỗng. Nếu `write-tree` ngay lúc đó sẽ ra đúng tree của HEAD.
</details>

<details>
<summary>2. <code>git status --short</code> hiện <code>AM new.txt</code>. Commit ngay bây giờ thì <code>new.txt</code> trong commit có nội dung gì?</summary>

Nội dung tại thời điểm `git add`, không phải nội dung hiện tại trên đĩa.
</details>

<details>
<summary>3. File nằm trong <code>.gitignore</code> nhưng đã có trong index. Git có theo dõi thay đổi của nó không?</summary>

Có. `.gitignore` chỉ ảnh hưởng tới file **untracked**. File đã có trong index thì vẫn tracked. Muốn ngừng theo dõi: `git rm --cached <file>`.
</details>

<details>
<summary>4. Viết lệnh xem nội dung <code>config.yml</code> đang nằm trong index.</summary>

`git show :config.yml` (hoặc `git cat-file -p :config.yml`).
</details>

<details>
<summary>5. Làm sao hiện diff của <em>mọi</em> thay đổi chưa commit, cả staged lẫn unstaged?</summary>

`git diff HEAD`: so HEAD với working tree.
</details>

## 🏋️ Bài tập

1. Tạo tình huống mà `git status --short` hiện lần lượt: `A `, `AM`, `M `, `MM`, ` D`, `D `. Với mỗi trạng thái, ghi ra nội dung của cả ba cây.
2. Dùng `git add -p` stage một nửa thay đổi trong một file. Chứng minh index đang giữ một phiên bản không tồn tại ở HEAD hay working tree.
3. Chạy `touch` lên một file tracked, rồi `git ls-files --debug <file>` trước và sau `git status`. Giải thích sự khác biệt.

## 📎 Tra nhanh

- [Cheat sheet 01: Working Directory & Staging Area](../cheat-sheet/01-co-ban.md)
- [Cheat sheet 10: Diff](../cheat-sheet/10-diff.md)
- Pro Git: [7.7 Reset Demystified](https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified) (phần "The Three Trees")

---

[← Trước: Refs, branch và HEAD](03-refs-branch-head.md) · [Lộ trình](../README.md) · [Tiếp: Đồ thị commit →](05-do-thi-commit.md)
