# 01. Git là gì: snapshot, phân tán, ba vùng

[← Lộ trình](../README.md) · Giai đoạn 1: Nền tảng · ⏱️ ~30 phút

## 🎯 Mục tiêu

- Giải thích được vì sao Git lưu **snapshot** chứ không lưu "danh sách thay đổi".
- Hiểu "phân tán" (distributed) nghĩa là gì trong thực tế.
- Phân biệt rõ **working tree**, **index** và **repository**.
- Biết thư mục `.git/` chứa những gì.

## ❓ Vấn đề

Không có hệ thống quản lý phiên bản, bạn sẽ có `bao-cao-final.docx`, `bao-cao-final-v2.docx`, `bao-cao-final-that-su.docx`. Mọi hệ thống quản lý phiên bản đều giải quyết ba nhu cầu:

1. **Quay lại** bất kỳ trạng thái nào trong quá khứ.
2. **Làm song song** nhiều hướng phát triển (nhánh) rồi gộp lại.
3. **Cộng tác**: nhiều người cùng sửa mà không đè lên nhau.

Git giải quyết cả ba bằng một thiết kế rất đơn giản. Hiểu thiết kế đó là mục tiêu của cả giai đoạn 1.

## 🧠 Khái niệm

### 1. Git lưu snapshot, không lưu delta

Nhiều hệ thống cũ (CVS, Subversion) lưu lịch sử dưới dạng **danh sách thay đổi** trên từng file:

```
Delta-based:     v1        v2        v3
  file A         A    →    Δ1   →    Δ2
  file B         B    ─────────→     Δ1
```

Git thì lưu **toàn bộ trạng thái dự án** tại mỗi commit, như chụp ảnh:

```
Snapshot-based:  commit 1   commit 2   commit 3
  file A         A1         A2         A3
  file B         B1         B1 (dùng   B2
                            lại, không
                            chép lại)
```

"Lưu toàn bộ" nghe có vẻ tốn chỗ, nhưng file không đổi thì Git **không lưu lại lần nữa**, chỉ trỏ tới nội dung đã có (bài 02 sẽ giải thích cơ chế). Còn chuyện nén thì để bài 15.

Hệ quả quan trọng:

- Một commit = **một trạng thái đầy đủ** của dự án, không phải "một bản vá".
- `git diff` và "commit này đổi gì" là thứ Git **tính ra** bằng cách so hai snapshot, không phải dữ liệu được lưu sẵn.
- Checkout một commit cũ rất nhanh, vì không cần "cộng dồn" các delta.

> 💡 Nhớ điểm này thật kỹ. Rất nhiều hiểu lầm về `rebase`, `cherry-pick`, `revert` xuất phát từ việc nghĩ commit là một bản vá.

### 2. Phân tán (distributed)

Với Git, **mỗi bản clone là một repository đầy đủ**: toàn bộ lịch sử, mọi nhánh, mọi commit. Không có máy nào "đặc biệt" về mặt kỹ thuật. GitHub hay GitLab chỉ là một bản sao mà cả nhóm **quy ước** là bản chung.

Hệ quả:

- Gần như mọi thao tác (commit, log, diff, branch, merge) chạy **offline** và rất nhanh.
- Chỉ bốn nhóm lệnh cần mạng: `clone`, `fetch`, `pull`, `push` (cùng `ls-remote`, `archive --remote`...).
- Commit của bạn chỉ nằm trên máy bạn cho tới khi bạn `push`.

### 3. Ba vùng

```
  Working tree            Index (staging area)          Repository (.git)
  ─────────────           ────────────────────          ─────────────────
  File thật trên đĩa,     Bản nháp cho commit tiếp      Lịch sử đã commit,
  bạn sửa trực tiếp       theo. Là một file nhị phân     không thể sửa
                          .git/index
        │                          │                           │
        └──── git add ────────────►│                           │
                                   └───── git commit ─────────►│
        ◄───────────────── git switch / git restore ───────────┘
```

| Vùng | Là gì | Nằm ở đâu |
|------|-------|-----------|
| **Working tree** | Các file bạn nhìn thấy và sửa trong editor | Thư mục dự án (trừ `.git/`) |
| **Index** | Danh sách file + nội dung sẽ vào commit tiếp theo | `.git/index` |
| **Repository** | Toàn bộ object (nội dung, cây thư mục, commit) và ref | `.git/objects/`, `.git/refs/`... |

Vì sao cần index ở giữa? Vì nó cho phép bạn **chọn lọc**: sửa 5 file nhưng chỉ commit 2 file, thậm chí chỉ commit một phần của một file (`git add -p`). Nhờ vậy mỗi commit có thể là một thay đổi logic trọn vẹn.

### 4. Trạng thái của một file

Từ góc nhìn Git, mỗi file trong working tree thuộc một trong các trạng thái:

```
 Untracked ──git add──► Staged ──git commit──► Unmodified
                          ▲                        │
                          │                     (bạn sửa file)
                          └──git add── Modified ◄──┘
```

- **Untracked**: Git chưa từng thấy file này (không có trong index).
- **Unmodified**: nội dung giống hệt bản trong commit hiện tại.
- **Modified**: đã sửa nhưng chưa `add`.
- **Staged**: đã `add`, sẽ nằm trong commit tiếp theo.

Một file có thể **vừa staged vừa modified**: bạn `add` rồi lại sửa tiếp. Khi đó index giữ bản lúc `add`, working tree giữ bản mới nhất. Bài 04 sẽ giải thích chính xác.

## 🔬 Bên trong Git: thư mục `.git/`

`git init` chỉ làm một việc: tạo thư mục `.git/`. **Toàn bộ repository nằm trong đó.** Xoá `.git/` thì dự án trở lại thành một thư mục bình thường, mất hết lịch sử.

```
.git/
├── HEAD            # "tôi đang ở đâu": thường là "ref: refs/heads/main"
├── config          # cấu hình riêng của repo (remote, branch upstream...)
├── description     # chỉ dùng cho GitWeb, bỏ qua
├── hooks/          # script chạy tự động (bài 16)
├── info/exclude    # giống .gitignore nhưng không commit
├── objects/        # object database: toàn bộ nội dung (bài 02)
├── refs/
│   ├── heads/      # branch local (bài 03)
│   └── tags/       # tag
├── index           # staging area (chỉ xuất hiện sau lần add đầu tiên)
└── logs/           # reflog (bài 09)
```

## 🧪 Thực hành

```bash
mkdir -p ~/git-lab && cd ~/git-lab
rm -rf demo && git init demo && cd demo

# 1. Repo mới chỉ là một thư mục .git
ls -A                       # .git
cat .git/HEAD               # ref: refs/heads/main  (hoặc master)
ls .git/refs/heads          # rỗng! nhánh main chưa tồn tại vì chưa có commit
find .git/objects -type f   # rỗng

# 2. Tạo file: file ở trạng thái untracked
echo "hello" > a.txt
git status --short          # ?? a.txt

# 3. add: file được ghi vào object database và index
git add a.txt
git status --short          # A  a.txt
find .git/objects -type f   # xuất hiện một object!
ls .git/index               # file index đã được tạo

# 4. Sửa tiếp: file vừa staged vừa modified
echo "world" >> a.txt
git status --short          # AM a.txt

# 5. commit
git add a.txt
git commit -m "Commit đầu tiên"
ls .git/refs/heads          # main: giờ nhánh mới tồn tại
cat .git/refs/heads/main    # một mã hash 40 ký tự
```

Câu hỏi để suy nghĩ: ở bước 3, **chưa commit** nhưng Git đã ghi object vào `.git/objects`. Vì sao? Bài 02 sẽ trả lời.

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "Git lưu diff của từng commit" | Git lưu snapshot. Diff được tính khi cần. |
| "GitHub là Git" | GitHub là dịch vụ lưu trữ repo Git, cộng thêm PR, issue... Git chạy độc lập hoàn toàn. |
| "Commit là đưa code lên server" | Commit chỉ ghi vào `.git/` trên máy bạn. `push` mới gửi lên remote. |
| "`git add` chỉ đánh dấu file" | `add` **chép nội dung** file vào object database ngay lúc đó. Sửa file sau khi add thì bản đã add không đổi. |
| "Branch là một bản sao của code" | Branch chỉ là một con trỏ tới commit (bài 03). |

## ✅ Tự kiểm tra

<details>
<summary>1. Nếu 1000 commit đều không sửa file <code>logo.png</code>, Git lưu bao nhiêu bản của file đó?</summary>

Một bản. Các snapshot đều trỏ tới cùng một nội dung. Bài 02 giải thích vì sao Git biết hai nội dung giống nhau.
</details>

<details>
<summary>2. Bạn <code>git add a.txt</code>, sửa tiếp a.txt, rồi <code>git commit</code>. Commit chứa phiên bản nào?</summary>

Phiên bản tại thời điểm `add`. Commit được tạo từ **index**, không phải từ working tree.
</details>

<details>
<summary>3. Mất mạng thì những lệnh nào không chạy được?</summary>

Chỉ các lệnh trao đổi với remote: `clone`, `fetch`, `pull`, `push`, `ls-remote`. Commit, branch, merge, log, diff đều chạy bình thường.
</details>

<details>
<summary>4. Vì sao ngay sau <code>git init</code>, <code>.git/refs/heads/</code> rỗng dù HEAD trỏ tới <code>main</code>?</summary>

Branch là con trỏ tới một commit. Chưa có commit nào thì không có gì để trỏ. HEAD đang trỏ tới một nhánh "sắp được tạo" (Git gọi là *unborn branch*). Commit đầu tiên sẽ tạo file `refs/heads/main`.
</details>

## 🏋️ Bài tập

1. Tạo repo, tạo 3 file, chỉ commit 2 file. Dùng `git status` để xác nhận file còn lại vẫn untracked.
2. Tạo tình huống để `git status --short` hiện `MM`. Giải thích từng chữ M.
3. Copy nguyên thư mục repo sang chỗ khác (`cp -r demo demo2`). `demo2` có phải là một repo đầy đủ không? Kiểm tra bằng `git log`.

## 📎 Tra nhanh

- [Cheat sheet 01: Lệnh cơ bản, Working Directory & Staging Area](../cheat-sheet/01-co-ban.md)
- Pro Git: [1.3 What is Git?](https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F)

---

[← Lộ trình](../README.md) · [Tiếp: Object model →](02-object-model.md)
