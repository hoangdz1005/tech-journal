# 03. Refs, branch và HEAD

[← Lộ trình](../README.md) · Giai đoạn 1: Nền tảng · ⏱️ ~45 phút

## 🎯 Mục tiêu

- Hiểu ref là gì và vì sao branch "rẻ" đến mức gần như miễn phí.
- Giải thích chính xác `git commit` thay đổi ref nào.
- Phân biệt HEAD **gắn với nhánh** (attached) và **detached HEAD**.
- Biết các ref đặc biệt: `ORIG_HEAD`, `FETCH_HEAD`, `MERGE_HEAD`.

## ❓ Vấn đề

Object database lưu mọi thứ bằng hash như `be41f720128b...`. Không ai nhớ được chuỗi đó. Hơn nữa commit chỉ trỏ **ngược** về cha, nên nếu không biết commit mới nhất thì bạn không thể đi được đến đâu. Cần một cơ chế để **đặt tên** cho commit và biết **đầu** của mỗi dòng lịch sử ở đâu.

## 🧠 Khái niệm

### Ref = tên trỏ tới một hash

Một **ref** (reference) chỉ là một file văn bản chứa một hash:

```bash
cat .git/refs/heads/main
# 871b580d6ab99dc67f10f038011bcb81de5c2216
```

Thế thôi. 40 ký tự hex và một ký tự xuống dòng: 41 byte.

| Loại ref | Nằm ở | Ý nghĩa |
|----------|-------|---------|
| Branch local | `refs/heads/<tên>` | Đầu một nhánh của bạn. **Tự di chuyển** khi commit |
| Tag | `refs/tags/<tên>` | Nhãn cố định, **không** tự di chuyển |
| Remote-tracking | `refs/remotes/<remote>/<tên>` | Bản ghi nhớ nhánh trên remote ở lần fetch gần nhất (bài 10) |
| Stash | `refs/stash` | Stash mới nhất (bài 13) |

### Branch chỉ là một con trỏ di động

Đây là câu quan trọng nhất của bài:

> **Branch không chứa commit. Branch là một con trỏ tới đúng một commit.**

"Các commit thuộc nhánh `feature`" thực chất có nghĩa là: các commit **đi ngược được** từ commit mà `feature` trỏ tới, theo con trỏ `parent`.

```
          A ◄── B ◄── C ◄── D      ◄── main
                       ▲
                       └── E ◄── F  ◄── feature
```

`A`, `B`, `C` vừa "thuộc" `main` vừa "thuộc" `feature`. Git không lưu thông tin "commit này sinh ra trên nhánh nào".

Vì vậy:

- **Tạo branch** = ghi một file 41 byte. Tức thời, không chép code.
- **Xoá branch** = xoá file đó. Commit **không** bị xoá (nhưng có thể thành "mồ côi", xem bên dưới).
- **Đổi tên branch** = đổi tên file.

### HEAD: "tôi đang ở đâu"

`HEAD` là một ref đặc biệt tại `.git/HEAD`. Bình thường nó là **symbolic ref**, tức trỏ tới **một ref khác** chứ không trỏ tới hash:

```bash
cat .git/HEAD
# ref: refs/heads/main
```

Chuỗi con trỏ: `HEAD → main → 871b580`.

### `git commit` di chuyển ref nào?

Khi commit, Git:

1. Tạo commit mới với `parent` = commit mà HEAD đang (gián tiếp) trỏ tới.
2. Cập nhật **nhánh mà HEAD trỏ tới** sang commit mới.
3. HEAD **không đổi** (vẫn là `ref: refs/heads/main`).

```
Trước:   HEAD → main → C          Sau:   HEAD → main → D
                                                       │
         A ◄── B ◄── C                   A ◄── B ◄── C ◄── D
```

Đó là lý do nhánh "tự chạy theo" commit mới. Chỉ **nhánh đang checkout** di chuyển, các nhánh khác đứng yên.

### Detached HEAD

Nếu bạn checkout **trực tiếp một commit** (hoặc một tag, hoặc một remote-tracking branch), HEAD sẽ chứa **hash** thay vì tên nhánh:

```bash
git switch --detach 871b580
cat .git/HEAD
# 871b580d6ab99dc67f10f038011bcb81de5c2216
```

```
Attached:  HEAD → main → C          Detached:  HEAD ──────► B
                                               main ──────► C
```

Detached HEAD **không phải lỗi**. Nó hữu ích để xem code cũ, chạy thử, bisect. Nhưng hãy cẩn thận khi commit trong trạng thái này:

```
          A ◄── B ◄── C   ◄── main
                 ▲
                 └── X ◄── Y  ◄── HEAD   (không có nhánh nào trỏ tới)
```

Khi bạn `switch` sang nhánh khác, không còn gì trỏ tới `Y`. Commit `X`, `Y` trở thành **unreachable** (không truy cập được). Chúng vẫn nằm trong object database và cứu được qua reflog (bài 09), nhưng sẽ bị `git gc` dọn sau một thời gian (bài 15).

Muốn giữ lại thì tạo nhánh **trước khi rời đi**:

```bash
git switch -c thu-nghiem     # nhánh mới trỏ tới Y, HEAD gắn vào nhánh này
```

### Ref đặc biệt khác

| Ref | Ghi bởi | Ý nghĩa |
|-----|---------|---------|
| `ORIG_HEAD` | `reset`, `merge`, `rebase` | Vị trí HEAD **trước** thao tác nguy hiểm. `git reset --hard ORIG_HEAD` để quay lại |
| `FETCH_HEAD` | `fetch` | Những gì vừa fetch về |
| `MERGE_HEAD` | `merge` khi đang có conflict | Commit đang được merge vào |
| `CHERRY_PICK_HEAD`, `REVERT_HEAD` | `cherry-pick`, `revert` | Commit đang được xử lý |

Các file này nằm ngay trong `.git/`.

### `packed-refs`

Repo có hàng nghìn tag thì hàng nghìn file nhỏ sẽ chậm. Git gom ref vào một file `.git/packed-refs`:

```
# pack-refs with: peeled fully-peeled sorted
871b580d6ab99dc67f10f038011bcb81de5c2216 refs/heads/main
3b65457803c7f353d42371c6c5ad80d8d7894668 refs/tags/v1
```

Vì vậy **đừng đọc/ghi ref bằng cách thao tác file trực tiếp** trong script. Dùng `git rev-parse`, `git update-ref`, `git for-each-ref`. Git bản mới còn có backend **reftable** thay cho file, càng không nên phụ thuộc vào cấu trúc thư mục.

## 🔬 Bên trong Git: các lệnh làm gì với ref

| Lệnh | Tác động lên ref |
|------|------------------|
| `git branch feat` | Tạo `refs/heads/feat` trỏ tới commit của HEAD |
| `git branch -d feat` | Xoá `refs/heads/feat` |
| `git switch feat` | Ghi `ref: refs/heads/feat` vào HEAD (và cập nhật index + working tree, bài 07) |
| `git commit` | Tạo commit, di chuyển nhánh mà HEAD trỏ tới |
| `git reset <commit>` | Di chuyển nhánh mà HEAD trỏ tới sang `<commit>` (bài 09) |
| `git tag v1` | Tạo `refs/tags/v1` trỏ tới commit |
| `git fetch` | Cập nhật `refs/remotes/origin/*` |

Để ý: `reset` và `commit` đều **di chuyển nhánh hiện tại**. `switch` thì đổi **HEAD trỏ tới nhánh nào**. Phân biệt được hai việc này là hiểu được một nửa bài 09.

## 🧪 Thực hành

```bash
cd ~/git-lab && rm -rf refs && git init refs && cd refs
echo 1 > f && git add f && git commit -m "A"
echo 2 > f && git commit -am "B"

# 1. Branch là file
cat .git/HEAD
cat .git/refs/heads/main
git rev-parse main              # cách "chuẩn" để đọc ref

# 2. Tạo branch bằng tay, không dùng git branch
git rev-parse HEAD > .git/refs/heads/tay
git branch                      # "tay" xuất hiện!
# (Chỉ để hiểu. Trong thực tế dùng git branch hoặc git update-ref.)

# 3. Commit chỉ di chuyển nhánh hiện tại
git switch tay
echo 3 > f && git commit -am "C"
git log --oneline --graph --all
# * xxxxxxx (HEAD -> tay) C
# * yyyyyyy (main) B
# * zzzzzzz A

# 4. Detached HEAD
git switch --detach main~1      # về commit A
cat .git/HEAD                   # hash, không có "ref:"
echo 4 > f && git commit -am "Mồ côi"
git log --oneline --graph --all # thấy commit "Mồ côi" với HEAD
git switch main                 # Git cảnh báo sẽ bỏ lại 1 commit
git log --oneline --graph --all # commit "Mồ côi" biến mất khỏi đồ thị

# 5. Nhưng nó vẫn còn!
git reflog | head -3            # tìm hash của "Mồ côi"
git branch cuu-ho <hash>        # gắn lại một nhánh
git log --oneline --graph --all

# 6. Liệt kê mọi ref
git for-each-ref
git show-ref
```

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "Xoá nhánh là xoá code" | Chỉ xoá con trỏ. Commit vẫn còn, reflog vẫn nhớ (mặc định tối thiểu 30 ngày). |
| "Commit biết nó thuộc nhánh nào" | Commit không chứa tên nhánh. "Thuộc nhánh" = đi ngược được từ đầu nhánh. |
| "HEAD là commit mới nhất" | HEAD là **vị trí hiện tại của bạn**, có thể là commit rất cũ. |
| "Detached HEAD là trạng thái lỗi cần sửa" | Đó là trạng thái hợp lệ, chỉ cần nhớ tạo nhánh nếu muốn giữ commit. |
| "Tag và branch khác nhau về bản chất" | Cùng là ref. Khác ở chỗ branch tự di chuyển khi commit, tag thì không. |

## ✅ Tự kiểm tra

<details>
<summary>1. Đang ở <code>main</code>, chạy <code>git branch feat</code> rồi <code>git commit</code>. Nhánh nào di chuyển?</summary>

`main`. `git branch` chỉ tạo nhánh, **không** chuyển HEAD sang nhánh đó. Muốn tạo và chuyển luôn thì dùng `git switch -c feat`.
</details>

<details>
<summary>2. Nội dung <code>.git/HEAD</code> khi đang detached là gì?</summary>

Một hash 40 ký tự, không có tiền tố `ref:`.
</details>

<details>
<summary>3. <code>git checkout v1.0</code> (v1.0 là tag) đưa bạn vào trạng thái nào? Vì sao?</summary>

Detached HEAD. HEAD chỉ gắn (attached) được vào `refs/heads/*`. Nếu cho HEAD gắn vào tag thì commit sẽ di chuyển tag, trái với ý nghĩa của tag.
</details>

<details>
<summary>4. Repo có 10.000 commit. Tạo một nhánh mới tốn bao nhiêu dung lượng?</summary>

Khoảng 41 byte (cộng một dòng reflog). Không phụ thuộc số commit hay kích thước code.
</details>

## 🏋️ Bài tập

1. Dùng `git update-ref` (thay vì `git branch`) để tạo nhánh `test` trỏ tới commit đầu tiên. Dùng `git symbolic-ref HEAD refs/heads/test` để "chuyển nhánh" mà không đụng tới working tree. Chạy `git status` và giải thích kết quả.
2. Chạy `git pack-refs --all`, rồi xem `.git/refs/heads/` và `.git/packed-refs`. Branch còn hoạt động không?
3. Vẽ đồ thị (trên giấy) sau chuỗi lệnh sau, rồi kiểm chứng bằng `git log --graph --all`:
   ```bash
   git switch -c a; git commit --allow-empty -m a1
   git switch main; git commit --allow-empty -m m1
   git switch a; git commit --allow-empty -m a2
   ```

## 📎 Tra nhanh

- [Cheat sheet 02: Branch & Checkout](../cheat-sheet/02-branch.md)
- [Cheat sheet 13: Tham chiếu](../cheat-sheet/13-tham-chieu-va-index.md)
- Pro Git: [3.1 Branches in a Nutshell](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell), [10.3 Git References](https://git-scm.com/book/en/v2/Git-Internals-Git-References)

---

[← Trước: Object model](02-object-model.md) · [Lộ trình](../README.md) · [Tiếp: Index và ba cây →](04-index-ba-cay.md)
