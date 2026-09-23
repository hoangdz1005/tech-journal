# 14. Điều tra lịch sử: log, blame, pickaxe, bisect

[← Lộ trình](../README.md) · Giai đoạn 4: Nâng cao · ⏱️ ~50 phút

## 🎯 Mục tiêu

- Lọc `git log` theo tác giả, thời gian, đường dẫn, nội dung message.
- Tìm commit **thêm hoặc xoá** một đoạn code cụ thể (pickaxe `-S`, `-G`).
- Theo dõi lịch sử của **một hàm** hoặc một khoảng dòng (`log -L`).
- Dùng `blame` đúng cách, bỏ qua commit format/refactor.
- Dùng `bisect` để tìm commit gây bug bằng tìm kiếm nhị phân, kể cả tự động.

> Yêu cầu: [bài 05](../1-nen-tang/05-do-thi-commit.md) (revision, khoảng commit).

## ❓ Vấn đề

"Ai viết dòng này và tại sao?" "Hàm `calculateTax` bị xoá từ khi nào?" "Tuần trước còn chạy, giờ hỏng, mà có 200 commit ở giữa." Lịch sử Git trả lời được những câu này, nếu bạn biết cách hỏi.

## 🧠 `git log`: hỏi đúng câu

### Định dạng

```bash
git log --oneline --graph --decorate --all
git log --stat                         # kèm thống kê file
git log -p                             # kèm diff
git log --format='%h %ad %an %s' --date=short
git log --format='%h %<(12,trunc)%an %s'
```

Các placeholder hay dùng: `%H`/`%h` hash, `%an` author, `%ad` ngày author, `%cn`/`%cd` committer, `%s` tiêu đề, `%b` thân, `%d` ref, `%p` hash cha. Xem đủ ở `git help log` (mục PRETTY FORMATS).

### Lọc commit

| Tuỳ chọn | Lọc theo |
|----------|----------|
| `-n 5`, `-5` | 5 commit gần nhất |
| `--author="An"`, `--committer=` | Người viết / người commit (regex) |
| `--since="2 weeks ago"`, `--until=2026-01-01` | Thời gian |
| `--grep="login" -i` | Nội dung message |
| `-- path/` | Commit có chạm vào đường dẫn |
| `--follow -- file` | Theo một file qua các lần đổi tên (chỉ một file) |
| `--merges`, `--no-merges` | Chỉ / bỏ merge commit |
| `--first-parent` | Chỉ đi theo cha đầu (lịch sử "chính") |
| `--diff-filter=D` | Chỉ commit có **xoá** file (A thêm, M sửa, R đổi tên...) |
| `A..B`, `A...B` | Khoảng commit (bài 05) |

```bash
git log --oneline --diff-filter=D --name-only    # file nào đã bị xoá, ở commit nào
git log --oneline --since=monday --author="$(git config user.name)"   # báo cáo tuần
git shortlog -sn --no-merges                     # xếp hạng số commit theo người
```

### Pickaxe: tìm commit theo **nội dung thay đổi**

`--grep` tìm trong message. Pickaxe tìm trong **diff**:

| Tuỳ chọn | Tìm commit mà... |
|----------|------------------|
| `-S "chuỗi"` | **Số lần xuất hiện** của chuỗi thay đổi (tức là thêm hoặc xoá chuỗi đó) |
| `-S "regex" --pickaxe-regex` | Như trên, dùng regex |
| `-G "regex"` | Có dòng khớp regex trong phần **diff** (kể cả chỉ di chuyển hoặc sửa dòng chứa nó) |

```bash
git log -S "calculateTax" --oneline       # commit thêm/xoá calculateTax
git log -S "API_KEY" -p --all             # ai từng commit API_KEY, trên mọi nhánh
git log -G "timeout\s*=\s*\d+" --oneline  # mọi lần sửa dòng cấu hình timeout
```

Khác biệt: nếu một commit chỉ **sửa** dòng `x = calculateTax(a)` thành `x = calculateTax(b)`, số lần xuất hiện của `calculateTax` không đổi. Khi đó `-S` **không** tìm thấy, còn `-G` thì có.

### `log -L`: lịch sử của một hàm hoặc một khoảng dòng

```bash
git log -L 10,25:src/tax.js               # lịch sử dòng 10–25
git log -L :calculateTax:src/tax.js       # lịch sử hàm calculateTax
```

Git theo dõi khoảng dòng đó qua các commit, kể cả khi nó bị dịch chuyển do code phía trên thay đổi. Mỗi commit hiện diff giới hạn trong khoảng đó.

### `git grep`: tìm trong code (ở bất kỳ commit nào)

```bash
git grep -n "TODO"                        # trong working tree (chỉ file tracked, rất nhanh)
git grep -n "TODO" v1.0                   # trong snapshot của tag v1.0
git grep -c "console.log" -- '*.js'       # đếm số lần mỗi file
```

## 🧠 `git blame`: dòng này từ đâu ra

```bash
git blame src/tax.js
git blame -L 40,60 src/tax.js
git blame -w          # bỏ qua thay đổi khoảng trắng
git blame -M          # phát hiện dòng bị di chuyển trong cùng file
git blame -C -C       # phát hiện dòng bị chép/di chuyển từ file khác
```

### 🔬 Blame hoạt động thế nào

Blame đi ngược lịch sử. Với mỗi dòng trong file hiện tại, Git tìm commit **gần nhất** mà diff của nó đã **tạo ra** dòng đó ở dạng hiện tại. Vì vậy:

- Blame chỉ cho biết **ai chạm vào dòng lần cuối**, không phải ai viết logic ban đầu.
- Một commit format lại code (prettier, đổi tab thành space) sẽ "chiếm" blame của cả file.

Cách khắc phục commit format:

```bash
echo "<hash commit format>" >> .git-blame-ignore-revs     # commit file này vào repo
git config blame.ignoreRevsFile .git-blame-ignore-revs
```

GitHub cũng tự đọc file `.git-blame-ignore-revs`.

Muốn đi tiếp về quá khứ khi blame dừng ở một commit "không liên quan": `git blame <hash>^ -- file` để blame từ trước commit đó. Các IDE có sẵn thao tác "blame previous revision".

## 🧠 `git bisect`: tìm commit gây bug bằng tìm kiếm nhị phân

### Ý tưởng

Bạn biết commit `good` (không có bug) và `bad` (có bug). Giữa chúng có N commit. Thay vì kiểm tra lần lượt, Git checkout **commit ở giữa** và hỏi bạn tốt hay xấu, rồi loại bỏ một nửa. Chỉ cần khoảng **log₂(N)** bước: 1000 commit chỉ mất khoảng 10 lần kiểm tra.

```
good                                   bad
 ●───●───●───●───●───●───●───●───●───●───●
                 ▲ kiểm tra: good
                         ●───●───●───●───●
                             ▲ kiểm tra: bad
                         ●───●
                         ▲ kiểm tra: bad  → đây là commit đầu tiên có bug
```

### Thủ công

```bash
git bisect start
git bisect bad                  # commit hiện tại có bug
git bisect good v1.2            # v1.2 còn tốt
# Git checkout một commit ở giữa (detached HEAD). Bạn test, rồi:
git bisect good                 # hoặc: git bisect bad
# ... lặp lại cho tới khi Git báo "<hash> is the first bad commit"
git bisect reset                # quay về nhánh ban đầu
```

| Lệnh | Tác dụng |
|------|----------|
| `git bisect skip` | Commit này không test được (không build được...) |
| `git bisect log` | Xem lại các bước đã chọn |
| `git bisect visualize` | Xem các commit còn nghi vấn |
| `git bisect start --term-old=fast --term-new=slow` | Đổi thuật ngữ khi không phải tìm "bug" (ví dụ tìm commit làm chậm) |

### Tự động: `bisect run`

Nếu có một lệnh trả về mã thoát **0 = good**, **1–127 (trừ 125) = bad**, **125 = skip**, Git tự chạy hết:

```bash
git bisect start HEAD v1.2
git bisect run npm test
git bisect run ./scripts/check-bug.sh
git bisect reset
```

Mẹo: viết một test nhỏ **chỉ** tái hiện bug, đặt nó **ngoài** repo (hoặc là file untracked) để nó không bị thay đổi khi Git checkout các commit cũ.

### 🔬 Bên trong

`bisect` lưu trạng thái trong `refs/bisect/bad`, `refs/bisect/good-<hash>` và `.git/BISECT_LOG`. Ở mỗi bước, Git chọn commit chia đôi **số commit còn nghi vấn** trong đồ thị (không chỉ theo thứ tự thời gian), nên nó vẫn hoạt động với lịch sử có merge.

Bisect hiệu quả nhất khi **mỗi commit đều build được** và nhỏ. Đó là một lý do thực tế cho commit nguyên tử (bài 12).

## 🧪 Thực hành

```bash
cd ~/git-lab && rm -rf inv && git init inv && cd inv
for i in $(seq 1 20); do
  if [ $i -ge 13 ]; then echo "value=bad$i" > app.cfg; else echo "value=ok$i" > app.cfg; fi
  echo "line $i" >> notes.txt
  git add . && git commit -qm "Commit $i"
done

# 1. Pickaxe
git log -S "line 7" --oneline               # Commit 7 (dòng được thêm vào)
git log -G "ok1[0-9]" --oneline             # các commit mà diff có dòng khớp

# 2. log -L và blame
git log -L 1,1:notes.txt --oneline
git blame -L 5,6 notes.txt

# 3. Bisect thủ công: commit nào bắt đầu có "bad"?
git bisect start
git bisect bad HEAD
git bisect good HEAD~19
cat app.cfg                                 # tự đánh giá, rồi gõ git bisect good/bad
# ... lặp lại
git bisect reset

# 4. Bisect tự động
git bisect start HEAD HEAD~19
git bisect run sh -c '! grep -q bad app.cfg'   # 0 nếu không có "bad"
# → <hash> is the first bad commit ... Commit 13
git bisect reset
```

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "`blame` cho biết ai viết code" | Nó cho biết ai **chạm vào dòng lần cuối**. Commit format có thể che mất tác giả thật. |
| "`--grep` tìm trong code" | `--grep` tìm trong message. Tìm trong diff dùng `-S`/`-G`, tìm trong snapshot dùng `git grep`. |
| "`-S` và `-G` như nhau" | `-S` so **số lần xuất hiện**, `-G` khớp **dòng trong diff**. |
| "Bisect cần lịch sử thẳng" | Bisect hoạt động trên đồ thị, kể cả có merge. |
| "`log -- file` theo được file đã đổi tên" | Cần `--follow`, và chỉ với một file. |

## ✅ Tự kiểm tra

<details>
<summary>1. Tìm commit đã xoá hàm <code>legacyAuth</code>. Lệnh nào?</summary>

`git log -S "legacyAuth" --oneline` (commit gần nhất trong kết quả thường là commit xoá; thêm `-p` để xác nhận).
</details>

<details>
<summary>2. Có 500 commit giữa good và bad. Bisect cần khoảng bao nhiêu bước?</summary>

Khoảng 9 bước (log₂ 500 ≈ 9).
</details>

<details>
<summary>3. Script cho <code>bisect run</code> nên trả về mã gì khi commit không build được?</summary>

125, để Git `skip` commit đó.
</details>

<details>
<summary>4. Nhóm vừa chạy prettier cho toàn bộ repo. Làm sao để blame không hiện commit đó?</summary>

Thêm hash vào `.git-blame-ignore-revs` và cấu hình `blame.ignoreRevsFile` (GitHub tự hỗ trợ).
</details>

## 🏋️ Bài tập

1. Trên một repo mã nguồn mở bạn quan tâm, tìm commit đầu tiên thêm một hàm quan trọng và đọc message của nó.
2. Viết một script `check.sh` trả về 0/1/125 và dùng `bisect run` trên một lịch sử có vài commit không build được.
3. Dùng `git log --format` và `shortlog` để tạo "báo cáo tuần": số commit, file thay đổi nhiều nhất (gợi ý: `--name-only` + `sort | uniq -c | sort -rn`).

## 📎 Tra nhanh

- [Cheat sheet 09: Lịch sử, Log, Blame](../cheat-sheet/09-lich-su.md)
- [Cheat sheet 15: Tìm kiếm & Bisect](../cheat-sheet/15-tim-kiem-va-bisect.md)
- Pro Git: [7.10 Debugging with Git](https://git-scm.com/book/en/v2/Git-Tools-Debugging-with-Git), [2.3 Viewing the Commit History](https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History)

---

[← Trước: Stash và worktree](13-stash-worktree.md) · [Lộ trình](../README.md) · [Tiếp: Lưu trữ và gc →](15-luu-tru-va-gc.md)
