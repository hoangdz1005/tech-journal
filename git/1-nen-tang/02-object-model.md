# 02. Object model: blob, tree, commit, tag

[← Lộ trình](../README.md) · Giai đoạn 1: Nền tảng · ⏱️ ~60 phút

## 🎯 Mục tiêu

- Kể tên và giải thích 4 loại object của Git.
- Hiểu **content-addressable storage**: vì sao tên của object chính là hash nội dung của nó.
- Đọc được nội dung thô của bất kỳ object nào bằng `git cat-file`.
- Tự tạo một commit **không dùng `git add` hay `git commit`**.

## ❓ Vấn đề

Bài 01 nói Git lưu snapshot nhưng không lưu lại file trùng. Làm sao Git biết hai file giống nhau mà không phải so sánh từng byte với mọi file đã có? Và làm sao đảm bảo lịch sử không bị ai lén sửa?

Câu trả lời cho cả hai: **đặt tên mọi thứ bằng hash của chính nội dung đó**.

## 🧠 Khái niệm

### Git là một key-value store

Ở tầng thấp nhất, Git là một cơ sở dữ liệu key-value rất đơn giản:

- **Value**: một object (nội dung file, cây thư mục, commit...).
- **Key**: mã SHA-1 (40 ký tự hex) tính từ chính value đó.

```
key = SHA-1( "<loại> <kích thước>\0<nội dung>" )
```

Ví dụ với file có nội dung `hello\n` (6 byte):

```bash
printf 'blob 6\0hello\n' | shasum
# ce013625030ba8dba906f756967f9e9ca394464a
```

Ba tính chất quan trọng rút ra từ đây:

1. **Cùng nội dung → cùng hash**, trên mọi máy, mọi repo. Git tự động không lưu trùng.
2. **Nội dung thay đổi dù một byte → hash khác hoàn toàn**. Object **bất biến**: "sửa" một object nghĩa là tạo object mới.
3. **Hash là bằng chứng toàn vẹn**: nếu dữ liệu trên đĩa bị hỏng hoặc bị sửa, hash tính lại sẽ không khớp tên.

> 💡 Git đang chuyển dần sang SHA-256 (`git init --object-format=sha256`), nhưng SHA-1 vẫn là mặc định. Nguyên lý hoàn toàn giống nhau.

### Bốn loại object

```
                 ┌──────────────────────┐
                 │ tag  v1.0            │  (annotated tag, tuỳ chọn)
                 │ object → 3f2a...     │
                 └──────────┬───────────┘
                            ▼
                 ┌──────────────────────┐
                 │ commit 3f2a          │
                 │ tree   → 9c1e        │
                 │ parent → 871b        │──► commit cha
                 │ author, committer    │
                 │ message              │
                 └──────────┬───────────┘
                            ▼
                 ┌──────────────────────┐
                 │ tree 9c1e  (thư mục /)│
                 │ blob a.txt → ce01    │
                 │ tree src   → add4    │
                 └─────┬───────────┬────┘
                       ▼           ▼
              ┌────────────┐  ┌──────────────────┐
              │ blob ce01  │  │ tree add4 (src/) │
              │ "hello\n"  │  │ blob b.txt → 587b│
              └────────────┘  └─────────┬────────┘
                                        ▼
                                 ┌────────────┐
                                 │ blob 587b  │
                                 └────────────┘
```

| Object | Chứa gì | Tương đương |
|--------|---------|-------------|
| **blob** | Nội dung một file. **Không có tên file, không có quyền truy cập** | Nội dung file |
| **tree** | Danh sách mục: mode + loại + hash + tên. Mỗi mục trỏ tới blob (file) hoặc tree (thư mục con) | Một thư mục |
| **commit** | Hash của tree gốc, hash của commit cha (0, 1 hoặc nhiều), author, committer, message | Một snapshot kèm metadata |
| **tag** | Hash của object được gắn tag, tên tag, người tạo, message | Nhãn có chú thích (annotated tag) |

Những điểm cần để ý:

- **Blob không biết tên của nó.** Tên file nằm trong tree. Vì vậy đổi tên file không tạo blob mới, chỉ tạo tree mới.
- **Tree lồng nhau** giống hệt cây thư mục. Một commit chỉ trỏ tới **tree gốc**.
- **Commit trỏ tới cha.** Chuỗi con trỏ này tạo thành lịch sử (bài 05).
- Git **không lưu thư mục rỗng**, vì tree rỗng không có blob nào để chứa. Đó là lý do người ta hay đặt file `.gitkeep`.

### Vì sao sửa một file tạo ra cả chuỗi object mới

Giả sử bạn sửa `src/b.txt`:

```
blob b.txt mới  →  tree src/ mới (vì một mục đổi hash)
                →  tree gốc mới (vì mục src đổi hash)
                →  commit mới trỏ tới tree gốc mới
```

Còn `a.txt` không đổi, nên tree gốc mới **vẫn trỏ tới blob ce01 cũ**. Đó chính là cách snapshot "lưu toàn bộ" mà không tốn chỗ.

### Merkle tree: vì sao lịch sử không thể lén sửa

Commit chứa hash của tree và hash của cha. Tree chứa hash của các blob. Vậy **hash của một commit phụ thuộc vào toàn bộ nội dung và toàn bộ lịch sử phía trước nó**. Sửa một byte trong một file ở commit cũ thì hash của commit đó thay đổi, kéo theo hash mọi commit phía sau thay đổi.

Hệ quả thực tế, sẽ gặp lại ở bài 11:

- Hai người có cùng hash commit đầu nhánh ⇒ họ có **chính xác** cùng lịch sử.
- "Sửa" một commit cũ (amend, rebase) luôn tạo ra **commit mới với hash mới**. Không có cách nào sửa tại chỗ.

## 🔬 Bên trong Git: object nằm ở đâu

Mỗi object được nén bằng zlib và lưu tại `.git/objects/<2 ký tự đầu>/<38 ký tự còn lại>`:

```
.git/objects/ce/013625030ba8dba906f756967f9e9ca394464a
```

Giải nén ra sẽ thấy đúng định dạng `"<loại> <kích thước>\0<nội dung>"`:

```bash
python3 -c "import zlib;print(zlib.decompress(open('.git/objects/ce/013625030ba8dba906f756967f9e9ca394464a','rb').read()))"
# b'blob 6\x00hello\n'
```

Dạng lưu từng file như vậy gọi là **loose object**. Về sau Git gom chúng vào **packfile** để tiết kiệm chỗ (bài 15), nhưng về mặt logic không có gì thay đổi.

### Porcelain và plumbing

Git có hai tầng lệnh:

- **Porcelain** (sứ, phần bạn nhìn thấy): `add`, `commit`, `switch`, `merge`... dành cho người dùng.
- **Plumbing** (đường ống bên dưới): `hash-object`, `cat-file`, `update-index`, `write-tree`, `commit-tree`, `update-ref`... là các bước nhỏ mà porcelain ghép lại.

Học plumbing một lần là cách nhanh nhất để hiểu porcelain làm gì.

| Lệnh plumbing | Việc làm |
|---------------|----------|
| `git hash-object [-w] <file>` | Tính hash của file; có `-w` thì ghi thành blob |
| `git cat-file -t <hash>` | Xem loại object |
| `git cat-file -s <hash>` | Xem kích thước |
| `git cat-file -p <hash>` | In nội dung object dạng dễ đọc |
| `git update-index --add --cacheinfo <mode>,<hash>,<path>` | Ghi một mục vào index |
| `git write-tree` | Tạo tree object từ index hiện tại |
| `git commit-tree <tree> [-p <cha>]` | Tạo commit object từ một tree |
| `git update-ref <ref> <hash>` | Cho một ref trỏ tới hash |

## 🧪 Thực hành: tự tạo commit bằng tay

Làm trong một repo **mới tinh**:

```bash
cd ~/git-lab && rm -rf manual && git init manual && cd manual

# Bước 1: tạo blob. -w = ghi vào object database
echo 'hello' | git hash-object -w --stdin
# ce013625030ba8dba906f756967f9e9ca394464a
# Hash này giống hệt trên máy mình và máy bạn!

git cat-file -t ce0136      # blob   (chỉ cần gõ vài ký tự đầu)
git cat-file -p ce0136      # hello

# Bước 2: đưa blob vào index với tên a.txt
git update-index --add --cacheinfo 100644,ce013625030ba8dba906f756967f9e9ca394464a,a.txt
git ls-files --stage
# 100644 ce013625030ba8dba906f756967f9e9ca394464a 0	a.txt

# Bước 3: biến index thành tree
git write-tree
# 2e81171448eb9f2ee3821e3d447aa6b2fe3ddba1
git cat-file -p 2e8117
# 100644 blob ce013625030ba8dba906f756967f9e9ca394464a	a.txt

# Bước 4: tạo commit từ tree
echo "Commit thủ công" | git commit-tree 2e8117
# <hash commit>, mỗi người sẽ ra một hash khác nhau. Vì sao?

# Bước 5: cho nhánh main trỏ tới commit đó
git update-ref refs/heads/main <hash commit>

git log --oneline
# be41f72 Commit thủ công
```

Bạn vừa làm đúng những gì `git add` + `git commit` làm. Giờ xem trạng thái:

```bash
git status --short
#  D a.txt
```

Git báo `a.txt` bị **xoá**, vì commit và index đều có `a.txt`, nhưng working tree thì không (ta chưa từng tạo file thật). Khôi phục từ index:

```bash
git restore a.txt
cat a.txt                  # hello
git status --short         # (trống): ba vùng đã khớp nhau
```

### Xem một commit thật

```bash
git cat-file -p HEAD
# tree 2e81171448eb9f2ee3821e3d447aa6b2fe3ddba1
# author An <an@example.com> 1767232800 +0700
# committer An <an@example.com> 1767232800 +0700
#
# Commit thủ công
```

Commit đầu tiên không có dòng `parent`. Commit thường có 1 dòng `parent`, merge commit có 2 dòng trở lên.

Tree hash `2e8117...` giống trên mọi máy, vì nó chỉ phụ thuộc nội dung. Commit hash thì khác, vì nó chứa tên, email và **thời điểm** tạo commit.

### Xem một annotated tag

```bash
git tag -a v1 -m "phát hành"
git cat-file -t v1          # tag
git cat-file -p v1
# object be41f720128b04fa287c1ff4818378a5d2777565
# type commit
# tag v1
# tagger An <an@example.com> 1767232800 +0700
#
# phát hành
```

Lightweight tag (`git tag v1` không có `-a`) thì **không tạo object**, chỉ là một ref trỏ thẳng tới commit (bài 03, 17).

### Khám phá tree lồng nhau

```bash
mkdir src && echo x > src/b.txt
git add . && git commit -m "Thêm src"
git cat-file -p HEAD^{tree}
# 100644 blob ce0136...	a.txt
# 040000 tree add479...	src
git cat-file -p HEAD:src      # xem tree con, cú pháp <commit>:<đường dẫn>
git cat-file -p HEAD:src/b.txt
```

Các mode thường gặp trong tree: `100644` file thường, `100755` file thực thi, `120000` symlink, `040000` thư mục, `160000` submodule.

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "Đổi tên file thì Git lưu thêm một bản" | Blob giữ nguyên, chỉ tree thay đổi. |
| "Git theo dõi việc đổi tên (rename)" | Git **không lưu** thông tin rename. `git log --follow`, `git diff -M` **đoán** rename bằng cách so độ giống nhau của nội dung. |
| "Amend sửa commit cũ" | Amend tạo commit mới. Commit cũ vẫn còn nguyên trong object database. |
| "Hash commit là số thứ tự ngẫu nhiên" | Hash được tính hoàn toàn từ nội dung commit. Cùng nội dung thì cùng hash. |
| "Git lưu quyền file đầy đủ" | Git chỉ phân biệt file thường (644) và thực thi (755). |

## ✅ Tự kiểm tra

<details>
<summary>1. Hai file khác tên, khác thư mục nhưng cùng nội dung. Có bao nhiêu blob?</summary>

Một blob. Hai tree entry khác tên cùng trỏ tới một hash.
</details>

<details>
<summary>2. Bạn chỉ đổi tên <code>docs/a.md</code> thành <code>docs/b.md</code> rồi commit. Những object mới nào được tạo?</summary>

Tree `docs/` mới (mục đổi tên), tree gốc mới (vì hash của `docs` đổi) và commit mới. **Không** có blob mới.
</details>

<details>
<summary>3. Vì sao hai người clone cùng repo, cùng checkout một commit hash thì chắc chắn có cùng nội dung?</summary>

Commit hash phụ thuộc vào tree hash, tree hash phụ thuộc vào blob hash. Nếu nội dung khác thì hash phải khác. Đây là tính chất Merkle tree.
</details>

<details>
<summary>4. Sau <code>git add</code> một file 100MB rồi <code>git rm --cached</code>, repo có nặng thêm không?</summary>

Có, tạm thời. `add` đã ghi blob vào `.git/objects`. Blob đó không còn được gì tham chiếu tới, sẽ bị dọn khi `git gc` chạy và hết thời gian ân hạn (bài 15).
</details>

<details>
<summary>5. Commit hash của bạn và của mình khác nhau dù cùng tree. Nêu các trường làm nó khác.</summary>

Tên và email author/committer, timestamp và timezone của author/committer, message, danh sách parent.
</details>

## 🏋️ Bài tập

1. Dùng plumbing, tạo **commit thứ hai** có cha là commit thủ công ở trên (gợi ý: `git commit-tree <tree> -p <cha>`), chứa thêm file `b.txt`.
2. Tạo hai file cùng nội dung ở hai thư mục khác nhau, commit, rồi chứng minh bằng `git cat-file` rằng chỉ có một blob.
3. Chạy `find .git/objects -type f | wc -l` trước và sau khi: (a) sửa 1 file rồi commit, (b) chỉ đổi tên 1 file rồi commit. Giải thích số object tăng thêm.
4. Thử sửa tay một byte trong file object (sau khi `chmod u+w`), rồi chạy `git fsck`. Quan sát lỗi.

## 📎 Tra nhanh

- [Cheat sheet 13: Tham chiếu, tracking & index](../cheat-sheet/13-tham-chieu-va-index.md)
- [Cheat sheet 14: Toàn vẹn dữ liệu](../cheat-sheet/14-bao-tri-va-don-dep.md)
- Pro Git: [10.2 Git Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)

---

[← Trước: Git là gì](01-git-la-gi.md) · [Lộ trình](../README.md) · [Tiếp: Refs, branch và HEAD →](03-refs-branch-head.md)
