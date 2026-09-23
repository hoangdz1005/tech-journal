# 06. add, commit, status, diff

[← Lộ trình](../README.md) · Giai đoạn 2: Thao tác hằng ngày · ⏱️ ~45 phút

## 🎯 Mục tiêu

- Mô tả từng bước mà `git add` và `git commit` thực hiện bên trong.
- Phân biệt `add .`, `add -A`, `add -u`, `commit -a`.
- Dùng `add -p` để tạo commit gọn gàng.
- Đọc hiểu output của `git diff` và biết chọn đúng biến thể.

> Yêu cầu: đã học [bài 02](../1-nen-tang/02-object-model.md) và [bài 04](../1-nen-tang/04-index-ba-cay.md).

## ❓ Vấn đề

Đây là các lệnh bạn gõ nhiều nhất mỗi ngày. Dùng thì dễ, nhưng những câu hỏi như "sao file mới không vào commit dù đã `commit -a`?" hay "sao `git diff` trống trơn dù mình vừa sửa?" chỉ trả lời được khi biết chính xác chúng làm gì.

## 🔬 `git add`: working tree → index

Với mỗi file được chỉ định, `git add`:

1. Đọc nội dung file trên đĩa (áp dụng filter nếu có, như chuyển CRLF hay Git LFS, bài 16).
2. Tính hash, **ghi blob** vào `.git/objects` (nếu chưa có).
3. Ghi/cập nhật mục trong index: đường dẫn, mode, hash blob, stat.

Tương đương plumbing:

```bash
git hash-object -w a.txt                     # bước 1–2
git update-index --add a.txt                 # bước 3
```

Với file đã bị xoá trên đĩa, `add` **xoá mục đó khỏi index**.

### Các biến thể

| Lệnh | Thêm file mới | Cập nhật file sửa | Ghi nhận file xoá | Phạm vi |
|------|:---:|:---:|:---:|---|
| `git add <path>` | ✅ | ✅ | ✅ | `<path>` |
| `git add .` | ✅ | ✅ | ✅ | Thư mục hiện tại trở xuống |
| `git add -A` | ✅ | ✅ | ✅ | Toàn bộ repo |
| `git add -u` | ❌ | ✅ | ✅ | Chỉ file **đã tracked** |
| `git add -p` | ❌ | Từng hunk | | File tracked, chọn tương tác |
| `git add -N <file>` | Chỉ ghi "ý định thêm", chưa có nội dung | | | Để `git diff` và `add -p` thấy file mới |

> 💡 Trước Git 2.0, `git add .` **không** ghi nhận file bị xoá. Tài liệu cũ trên mạng có thể mô tả khác.

### `git add -p`: tạo commit có chủ đích

`add -p` chia thay đổi thành các **hunk** và hỏi từng hunk:

| Phím | Ý nghĩa |
|------|---------|
| `y` / `n` | Stage / bỏ qua hunk này |
| `s` | Chia hunk thành các hunk nhỏ hơn (nếu được) |
| `e` | Sửa tay hunk trước khi stage |
| `q` | Thoát |
| `?` | Trợ giúp |

Bên trong: Git **dựng ra một phiên bản file mới** gồm nội dung ở index cộng các hunk bạn chọn, hash nó thành blob, rồi ghi vào index. Phiên bản đó có thể **chưa từng tồn tại trên đĩa**.

Đây là kỹ năng giúp bạn sửa lung tung trong lúc code nhưng vẫn tạo được các commit tách bạch: "sửa bug", "refactor", "thêm test".

## 🔬 `git commit`: index → commit mới → di chuyển nhánh

Khi bạn chạy `git commit -m "msg"`:

1. Chạy hook `pre-commit` (nếu có). Hook thất bại thì dừng.
2. Chuẩn bị message và chạy hook `commit-msg` để kiểm tra nó.
3. **`write-tree`**: biến index thành cây tree object. Tree con không đổi sẽ được dùng lại.
4. **`commit-tree`**: tạo commit object với tree ở bước 3, `parent` = commit hiện tại của HEAD, author/committer lấy từ config và thời gian hiện tại.
5. **`update-ref`**: di chuyển nhánh mà HEAD trỏ tới sang commit mới, ghi một dòng vào reflog.
6. Chạy hook `post-commit`.

```bash
# Chính là:
TREE=$(git write-tree)
COMMIT=$(git commit-tree $TREE -p HEAD -m "msg")
git update-ref -m "commit: msg" HEAD $COMMIT
```

Để ý: **working tree không hề được đọc** trong quá trình này. Commit chỉ phụ thuộc vào index.

### `commit -a` không thêm file mới

`git commit -a` = `git add -u` + `git commit`. Vì `-u` chỉ xử lý file đã tracked, **file mới (untracked) không vào commit**. Đây là cái bẫy kinh điển.

### Các tuỳ chọn hay dùng

| Lệnh | Tác dụng |
|------|----------|
| `git commit` | Mở editor để viết message nhiều dòng |
| `git commit -v` | Mở editor kèm diff của những gì sắp commit, giúp viết message chính xác |
| `git commit --amend` | Thay commit cuối bằng commit mới (bài 11) |
| `git commit --allow-empty` | Commit không thay đổi gì (để kích hoạt CI, đánh dấu mốc) |
| `git commit --author="Tên <email>"` | Đặt author khác committer |
| `git commit --fixup=<c>` | Tạo commit "sửa cho `<c>`", dùng với `rebase --autosquash` (bài 11) |

### Author và committer

Mỗi commit có hai người:

- **Author**: người viết thay đổi.
- **Committer**: người tạo ra commit object này.

Thường là một người. Chúng khác nhau khi bạn cherry-pick, rebase hay áp patch của người khác: author giữ nguyên, committer là bạn, với thời gian mới.

```bash
git log --format='%h %an (%ad) | %cn (%cd)' --date=short
```

## 🔬 `git rm` và `git mv`

| Lệnh | Thực chất |
|------|-----------|
| `git rm f` | Xoá `f` khỏi working tree **và** khỏi index |
| `git rm --cached f` | Chỉ xoá khỏi index (file thành untracked, vẫn còn trên đĩa) |
| `git mv a b` | `mv a b` + `git rm --cached a` + `git add b` |

`git mv` chỉ là tiện ích. Tự `mv` rồi `git add -A` cho kết quả **giống hệt**, vì Git không lưu thông tin rename.

## 🔬 `git diff`: so hai cây, tính ra patch

Nhắc lại từ bài 04:

| Lệnh | So sánh |
|------|---------|
| `git diff` | Index → working tree |
| `git diff --staged` (= `--cached`) | HEAD → index |
| `git diff HEAD` | HEAD → working tree |
| `git diff A B` | Commit A → commit B |
| `git diff A...B` | merge-base(A, B) → B |
| `git diff A B -- path/` | Như trên, giới hạn trong `path/` |

### Đọc một diff

```diff
diff --git a/a.txt b/a.txt          ← file bên trái (a/) và bên phải (b/)
index 5799d3c..8c1384d 100644       ← hash blob trước..sau, mode
--- a/a.txt
+++ b/a.txt
@@ -10,4 +10,5 @@ function login() {  ← hunk: từ dòng 10 lấy 4 dòng (trái), từ dòng 10 lấy 5 dòng (phải)
   const user = find(id);            ← dòng ngữ cảnh (không đổi)
-  if (user) {                       ← chỉ có ở bên trái
+  if (user && user.active) {        ← chỉ có ở bên phải
+    log(user);
   return user;
```

Vì Git lưu snapshot, diff được **tính lúc hiển thị** bằng thuật toán so sánh dòng (mặc định Myers). Cùng hai snapshot, các thuật toán khác nhau (`--diff-algorithm=histogram`, `patience`) có thể cho ra diff khác nhau nhưng đều đúng.

### Tuỳ chọn hữu ích

| Tuỳ chọn | Tác dụng |
|----------|----------|
| `--stat` | Tóm tắt số dòng thêm/xoá mỗi file |
| `--name-only`, `--name-status` | Chỉ tên file (kèm A/M/D/R) |
| `--word-diff` | Diff theo từ, tốt cho văn bản, Markdown |
| `-w` | Bỏ qua khác biệt khoảng trắng |
| `-M` | Phát hiện đổi tên (đã bật mặc định ở `diff`, `log`, `status`) |
| `--color-moved` | Tô màu khác cho khối code chỉ bị di chuyển |
| `-U<n>` | Số dòng ngữ cảnh (mặc định 3) |

## 🧪 Thực hành

```bash
cd ~/git-lab && rm -rf daily && git init daily && cd daily
printf 'a\nb\nc\nd\ne\nf\ng\nh\ni\nj\n' > f.txt
git add f.txt && git commit -qm "init"

# 1. commit -a bỏ sót file mới
echo new > new.txt
echo k >> f.txt
git commit -am "thử -a"
git status --short              # ?? new.txt  ← vẫn untracked!
git show --stat HEAD            # chỉ có f.txt

# 2. add -p: sửa hai chỗ xa nhau, chỉ commit một chỗ
sed -i.bak 's/^b$/B/; s/^i$/I/' f.txt && rm f.txt.bak
git diff                        # 2 hunk
git add -p f.txt                # trả lời y cho hunk đầu, n cho hunk sau
git diff --staged               # chỉ có b → B
git diff                        # chỉ có i → I
git show :f.txt | head -3       # index có B nhưng vẫn là i thường
git commit -qm "Viết hoa b"

# 3. commit không đọc working tree: chứng minh bằng plumbing
git add f.txt
git write-tree                  # tree từ index
git commit -qm "Viết hoa i"
git rev-parse HEAD^{tree}       # trùng với kết quả write-tree ở trên

# 4. rename chỉ là suy luận
mv new.txt renamed.txt
git add -A
git status --short              # hiện: A  renamed.txt, vì new.txt chưa từng tracked
git commit -qm "thêm renamed"
mv renamed.txt r2.txt && git add -A
git status --short              # R  renamed.txt -> r2.txt

# 5. Diff đa dạng
git diff HEAD~3 HEAD --stat
git diff HEAD~3 HEAD --name-status
git diff HEAD~3 HEAD --word-diff
```

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "`commit -a` commit mọi thứ" | Chỉ file **đã tracked**. File mới phải `add` trước. |
| "Commit lấy nội dung file trên đĩa" | Commit lấy từ **index**. |
| "Mỗi commit lưu một diff" | Mỗi commit trỏ tới một tree đầy đủ. Diff được tính khi xem. |
| "`git mv` cần thiết để Git biết đổi tên" | `mv` + `add -A` cho kết quả y hệt. Rename luôn được phát hiện sau, dựa trên độ giống nội dung. |
| "Diff của một commit là duy nhất" | Thuật toán diff khác nhau có thể cho ra các cách trình bày khác nhau của cùng một thay đổi. |

## ✅ Tự kiểm tra

<details>
<summary>1. Liệt kê các object mới được tạo khi commit thay đổi của một file <code>src/app/main.js</code> (repo đã có sẵn nhiều commit).</summary>

1 blob (nội dung mới, tạo lúc `add`), 3 tree (`src/app/`, `src/`, gốc) và 1 commit.
</details>

<details>
<summary>2. <code>git diff</code> trống nhưng <code>git status</code> vẫn báo có thay đổi. Vì sao?</summary>

Thay đổi đã được stage. `git diff` chỉ so index với working tree. Dùng `git diff --staged`. Cũng có thể là file untracked, `git diff` không hiện file untracked.
</details>

<details>
<summary>3. Khi nào author và committer của một commit khác nhau?</summary>

Khi commit được tạo lại từ thay đổi của người khác: cherry-pick, rebase, `git am` (áp patch), hoặc dùng `--author`.
</details>

<details>
<summary>4. Hook <code>pre-commit</code> báo lỗi thì commit object có được tạo không?</summary>

Không. `pre-commit` chạy trước `write-tree`/`commit-tree`. Tuy nhiên blob đã được tạo từ lúc `add`.
</details>

## 🏋️ Bài tập

1. Sửa 3 chỗ trong 2 file cho 2 mục đích khác nhau. Dùng `add -p` để tạo đúng 2 commit, mỗi commit một mục đích. Kiểm tra bằng `git show`.
2. Dùng `git add -N` với một file mới, rồi `git diff` và `git add -p` trên file đó. So sánh với khi không dùng `-N`.
3. Bật `git config diff.algorithm histogram` và so sánh diff của một lần refactor lớn với thuật toán mặc định.

## 📎 Tra nhanh

- [Cheat sheet 01: Lệnh cơ bản](../cheat-sheet/01-co-ban.md)
- [Cheat sheet 10: Diff](../cheat-sheet/10-diff.md)
- Pro Git: [2.2 Recording Changes](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository)

---

[← Trước: Đồ thị commit](../1-nen-tang/05-do-thi-commit.md) · [Lộ trình](../README.md) · [Tiếp: Branch, switch, checkout →](07-branch-switch-checkout.md)
