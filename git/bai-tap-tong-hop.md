# Bài tập tổng hợp

[← Lộ trình](README.md)

Các tình huống dưới đây mô phỏng sự cố thực tế. Hãy tự giải **trước khi** mở gợi ý. Với mỗi bài, cố gắng trả lời thêm: *lệnh của mình đã thay đổi object, ref, index hay working tree?*

Chuẩn bị một repo nháp cho mỗi bài:

```bash
cd ~/git-lab && rm -rf cap && git init cap && cd cap
c() { echo "$1" > "$1.txt"; git add .; git commit -qm "$1"; }
```

---

## Mức 1: Nền tảng

### 1. Commit nhầm nhánh

Bạn đã tạo 3 commit trên `main`, nhưng đáng lẽ chúng phải ở nhánh `feature/x` (chưa tồn tại). `main` chưa được push.

```bash
c A; c B; c C; c D      # giả sử B, C, D là 3 commit nhầm
```

<details>
<summary>Gợi ý</summary>

Branch chỉ là con trỏ (bài 03). Bạn cần một con trỏ mới ở D, và đưa `main` về A.
</details>

<details>
<summary>Lời giải</summary>

```bash
git branch feature/x          # con trỏ mới tại D
git reset --hard HEAD~3       # main về A
git switch feature/x
```

Không commit nào được tạo hay chép lại. Chỉ có hai ref thay đổi.
</details>

### 2. Quên một file trong commit cuối

Bạn vừa commit nhưng quên `config.yml`, và message có lỗi chính tả. Chưa push.

<details>
<summary>Lời giải</summary>

```bash
git add config.yml
git commit --amend -m "Message đúng"
```

Commit cũ vẫn còn trong object database (xem `git reflog`). `main` trỏ tới commit mới có cùng cha.
</details>

### 3. Stage một phần

Bạn sửa một file ở hai chỗ: một chỗ sửa bug, một chỗ thêm log debug. Chỉ commit phần sửa bug.

<details>
<summary>Lời giải</summary>

`git add -p <file>` → `y` cho hunk sửa bug, `n` cho hunk debug → `git diff --staged` để kiểm tra → `git commit`. Sau đó `git restore -p <file>` để bỏ phần debug nếu muốn.
</details>

### 4. Giải thích output

Không chạy lệnh, hãy dự đoán `git status --short`:

```bash
echo 1 > f && git add f && git commit -qm f1
echo 2 > f && git add f
echo 3 > f
git rm --cached -q f 2>/dev/null || git restore --staged f
```

<details>
<summary>Lời giải</summary>

`git rm --cached` với file có nội dung staged khác cả HEAD lẫn working tree sẽ **bị từ chối** (Git bảo vệ bạn khỏi mất dữ liệu), nên nhánh `||` chạy `git restore --staged f`: index lấy lại bản `1` từ HEAD. Working tree vẫn là `3`. Kết quả: ` M f`. Bản `2` đã stage giờ chỉ còn là một blob treo.
</details>

---

## Mức 2: Tích hợp

### 5. Cập nhật nhánh feature với main

`feature` tách từ `main` 2 tuần trước. `main` đã có 30 commit mới. Bạn muốn lịch sử thẳng trước khi mở PR, và nhánh chưa ai khác dùng.

<details>
<summary>Lời giải</summary>

```bash
git fetch
git switch feature
git rebase origin/main          # giải conflict theo từng commit nếu có
git push --force-with-lease --force-if-includes
```

Nhớ: trong lúc rebase, `--theirs` là commit **của bạn**.
</details>

### 6. Push bị từ chối

`git push` báo `! [rejected] main -> main (fetch first)`. Giải thích nguyên nhân bằng khái niệm reachable và xử lý mà không tạo merge commit.

<details>
<summary>Lời giải</summary>

Server có commit mà local chưa có. Nếu chấp nhận push, các commit đó sẽ không còn reachable từ `main` trên server. Xử lý: `git pull --rebase` (= fetch + rebase lên `origin/main`) rồi `git push`.
</details>

### 7. Mang một bản sửa lỗi sang nhánh release

Commit `abc123` trên `main` sửa một bug bảo mật. Cần đưa nó vào `release/2.3` mà không mang theo gì khác.

<details>
<summary>Lời giải</summary>

```bash
git switch release/2.3
git cherry-pick -x abc123
```

`-x` ghi lại nguồn gốc trong message. Cherry-pick là three-way merge với base = `abc123^`.
</details>

### 8. Hoàn tác một tính năng đã merge

PR `feature/payment` đã được merge vào `main` (merge commit `M`) và deploy, nhưng gây lỗi. Nhiều người đã pull `main`.

<details>
<summary>Lời giải</summary>

```bash
git revert -m 1 M
git push
```

Không dùng `reset` trên nhánh chung. Sau khi sửa xong tính năng, muốn merge lại thì phải `git revert <commit revert>` trước, vì các commit cũ của nhánh vẫn đang reachable từ `main`.
</details>

---

## Mức 3: Cứu hộ

### 9. Lỡ `reset --hard`

Bạn chạy `git reset --hard HEAD~5` trên nhánh cá nhân rồi nhận ra 5 commit đó rất quan trọng.

<details>
<summary>Lời giải</summary>

```bash
git reflog                       # tìm dòng ngay trước "reset: moving to HEAD~5"
git reset --hard HEAD@{1}        # hoặc ORIG_HEAD, nếu chưa có thao tác nào khác ghi đè nó
```
</details>

### 10. Xoá nhầm nhánh chưa merge

`git branch -D experiment`, rồi phát hiện nhánh này chưa được push.

<details>
<summary>Lời giải</summary>

Reflog của nhánh bị xoá mất theo, nhưng reflog của HEAD vẫn còn nếu bạn từng checkout nhánh đó:

```bash
git reflog | grep experiment     # "checkout: moving from experiment to main"
# hash ở dòng đó (hoặc dòng commit cuối trên experiment) là đầu nhánh
git branch experiment <hash>
```

Output của lệnh `branch -D` cũng in ra hash: `Deleted branch experiment (was 1a2b3c4)`.
</details>

### 11. Rebase hỏng giữa chừng

Đang interactive rebase 10 commit, tới commit thứ 6 thì conflict rối tung và bạn không còn chắc mình đã làm gì.

<details>
<summary>Lời giải</summary>

`git rebase --abort` đưa mọi thứ về trước khi rebase. Nếu đã lỡ `--continue` tới cuối: `git reset --hard ORIG_HEAD`, hoặc tìm trong reflog dòng `rebase (start)` và reset về dòng ngay trước đó.
</details>

### 12. Commit nhầm secret đã push

File `.env` chứa API key đã được push lên `main` từ 3 commit trước.

<details>
<summary>Lời giải</summary>

1. **Rotate API key ngay.** Đây là bước bắt buộc và quan trọng nhất.
2. Thêm `.env` vào `.gitignore`, `git rm --cached .env`, commit.
3. Nếu chính sách yêu cầu xoá khỏi lịch sử: `git filter-repo --invert-paths --path .env`, phối hợp với nhóm để force push và mọi người clone lại, liên hệ nền tảng để xoá cache.

Chỉ xoá khỏi lịch sử **không** đủ, vì bản clone, fork, log CI có thể đã có key.
</details>

---

## Mức 4: Điều tra

### 13. Commit nào gây bug?

Tính năng hoạt động ở tag `v3.0`, hỏng ở `HEAD`. Có khoảng 400 commit ở giữa, và có sẵn lệnh `npm test -- checkout.spec` để kiểm tra.

<details>
<summary>Lời giải</summary>

```bash
git bisect start HEAD v3.0
git bisect run npm test -- checkout.spec
git bisect reset
```

Khoảng 9 bước (log₂ 400). Commit nào không build được thì script trả về 125.
</details>

### 14. Ai xoá hàm này?

Hàm `applyDiscount` biến mất, không ai nhớ khi nào.

<details>
<summary>Lời giải</summary>

`git log -S "applyDiscount" --oneline -p`. Commit gần nhất trong danh sách là commit xoá. Đọc message để biết lý do.
</details>

---

## Thử thách cuối: tự làm lại `git commit`

Không dùng `git add` hay `git commit`, hãy tạo một commit **thứ hai** (có cha là HEAD) gồm một file mới `hello.txt`, rồi cập nhật nhánh hiện tại. Kết quả `git log` và `git status` phải giống như khi dùng lệnh thông thường.

<details>
<summary>Lời giải</summary>

```bash
echo "xin chào" > hello.txt
BLOB=$(git hash-object -w hello.txt)
git update-index --add --cacheinfo 100644,$BLOB,hello.txt
TREE=$(git write-tree)
COMMIT=$(echo "Thêm hello" | git commit-tree $TREE -p HEAD)
git update-ref -m "commit: Thêm hello" HEAD $COMMIT
git log --oneline -2
git status          # sạch
```

`update-ref HEAD` khi HEAD là symbolic ref sẽ cập nhật nhánh mà HEAD trỏ tới, đúng như `git commit`.
</details>

Nếu làm được bài này mà không cần xem lời giải, bạn đã thực sự hiểu Git. 🎉

---

[← Lộ trình](README.md)
