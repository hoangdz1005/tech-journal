# 09. Hoàn tác: restore, reset, revert, reflog

[← Lộ trình](../README.md) · Giai đoạn 2: Thao tác hằng ngày · ⏱️ ~60 phút

## 🎯 Mục tiêu

- Mô tả `restore`, `reset`, `revert` bằng mô hình ba cây và con trỏ nhánh.
- Phân biệt ba chế độ `reset --soft`, `--mixed`, `--hard` một cách chính xác.
- Biết thao tác nào **không thể** hoàn tác, thao tác nào luôn cứu được.
- Dùng reflog để lấy lại commit "đã mất".

> Yêu cầu: [bài 03](../1-nen-tang/03-refs-branch-head.md) (ref), [bài 04](../1-nen-tang/04-index-ba-cay.md) (ba cây).

## ❓ Vấn đề

"Hoàn tác" có rất nhiều nghĩa: bỏ sửa trong file, bỏ stage, huỷ commit cuối, huỷ một commit đã push từ tuần trước... Git có các lệnh khác nhau cho từng nghĩa, và chọn sai có thể làm mất dữ liệu hoặc làm khổ đồng đội.

## 🧠 Khái niệm: ba lệnh, ba tầng

| Lệnh | Tác động | Có đổi lịch sử? |
|------|----------|:---:|
| `git restore` | Chép **file** giữa các cây (commit/index → index/working tree) | Không |
| `git reset` | **Di chuyển nhánh hiện tại**, và tuỳ chế độ, đồng bộ index/working tree | **Có** |
| `git revert` | Tạo **commit mới** đảo ngược một commit cũ | Không (chỉ thêm) |

### `git restore`: hoàn tác ở mức file

`restore` không bao giờ di chuyển HEAD hay nhánh. Nó chỉ chép nội dung file.

| Lệnh | Nguồn | Đích | Dùng khi |
|------|-------|------|----------|
| `git restore <f>` | Index | Working tree | Bỏ thay đổi chưa stage |
| `git restore --staged <f>` | HEAD | Index | Bỏ stage (giữ thay đổi trong file) |
| `git restore --staged --worktree <f>` | HEAD | Index + working tree | Bỏ hết thay đổi của file |
| `git restore --source=<c> <f>` | Commit `<c>` | Working tree | Lấy lại phiên bản cũ của file |
| `git restore -p <f>` | Index | Working tree | Bỏ từng hunk |

### `git reset`: di chuyển nhánh

Đây là lệnh quan trọng và hay bị hiểu sai nhất. `git reset <commit>` luôn làm **bước 1**, và tuỳ chế độ làm thêm bước 2, 3:

1. **Di chuyển nhánh mà HEAD trỏ tới** sang `<commit>`. (HEAD vẫn gắn với nhánh, chỉ nhánh di chuyển.)
2. Chép tree của `<commit>` vào **index**. (`--mixed`, `--hard`)
3. Chép tree của `<commit>` vào **working tree**. (`--hard`)

```
Trạng thái đầu: main ở C, có thêm thay đổi đã stage (S) và chưa stage (W)

   A ◄── B ◄── C   ◄── main ◄── HEAD
                   index = C + S
                   working = C + S + W

git reset --soft B     main → B     index = C + S       working = C + S + W
git reset --mixed B    main → B     index = B           working = C + S + W
git reset --hard B     main → B     index = B           working = B
```

| Chế độ | Nhánh | Index | Working tree | Kết quả thường gặp |
|--------|:---:|:---:|:---:|------|
| `--soft` | ✅ | | | Thay đổi của C thành "đã stage", sẵn sàng commit lại |
| `--mixed` (mặc định) | ✅ | ✅ | | Thay đổi của C thành "chưa stage" |
| `--hard` | ✅ | ✅ | ✅ | Mọi thay đổi sau B **biến mất** khỏi index và working tree |

Commit `C` không bị xoá. Nó chỉ không còn được `main` trỏ tới. Reflog vẫn nhớ nó (xem bên dưới).

#### `reset` với đường dẫn

`git reset <commit> -- <path>` **không** di chuyển nhánh (không thể di chuyển nhánh cho một file). Nó chỉ chép `<path>` từ commit vào index, tương đương `git restore --staged --source=<commit> <path>`. `git reset <file>` (không có commit) là cách cũ để bỏ stage.

#### Công thức hay dùng

```bash
git reset --soft HEAD~1        # huỷ commit cuối, giữ thay đổi đã stage (để commit lại)
git reset HEAD~1               # huỷ commit cuối, thay đổi về trạng thái chưa stage
git reset --hard HEAD~1        # huỷ commit cuối và bỏ luôn thay đổi
git reset --soft HEAD~3        # gộp 3 commit cuối: reset rồi commit lại một lần
git reset --hard origin/main   # đưa nhánh local về giống hệt remote
git reset --hard ORIG_HEAD     # quay lại trước lần merge/rebase/reset vừa rồi
```

### `git revert`: hoàn tác bằng commit mới

`revert` tính **diff ngược** của một commit rồi áp lên HEAD, tạo thành commit mới:

```
   A ◄── B ◄── C ◄── D   ◄── main          C thêm dòng "x"

git revert C

   A ◄── B ◄── C ◄── D ◄── C'  ◄── main    C' xoá dòng "x"
```

Lịch sử chỉ được **thêm vào**, không bị viết lại, nên đây là cách **duy nhất** an toàn để hoàn tác commit **đã push lên nhánh dùng chung**.

Bên trong, `revert C` thực chất là một three-way merge với base = `C`, ours = `HEAD`, theirs = `C^`. Vì thế nó có thể **conflict** nếu các commit sau C đã sửa cùng chỗ. Xử lý như merge, rồi `git revert --continue`.

#### Revert một merge commit

Merge commit có hai cha. Revert cần biết "đảo ngược so với cha nào":

```bash
git revert -m 1 <merge-commit>    # giữ phía cha 1 (nhánh chính), bỏ những gì nhánh kia mang vào
```

⚠️ Sau khi revert một merge, nếu muốn merge lại nhánh đó, Git sẽ coi các commit cũ **đã được merge rồi** (chúng reachable). Bạn phải **revert cái revert** trước. Đây là bẫy nổi tiếng. Đọc thêm: `git help revert` và tài liệu *"How to revert a faulty merge"* (`Documentation/howto/revert-a-faulty-merge.txt` trong mã nguồn Git).

### `git clean`: xoá file untracked

`reset --hard` **không** xoá file untracked (vì chúng không có trong index, Git không quản lý). Dùng `clean`:

```bash
git clean -n        # xem trước (dry run). LUÔN chạy trước
git clean -f        # xoá file untracked
git clean -fd       # kèm thư mục untracked
git clean -fdx      # kèm cả file bị .gitignore (node_modules, build/...)
```

`clean` **không thể hoàn tác**: file untracked chưa từng vào object database.

## 🔬 Bên trong Git: reflog, lưới an toàn

Mỗi khi một ref thay đổi (commit, reset, switch, merge, rebase, amend...), Git ghi một dòng vào `.git/logs/<ref>`:

```bash
git reflog                 # reflog của HEAD
# a1b2c3d HEAD@{0}: reset: moving to HEAD~2
# 9f8e7d6 HEAD@{1}: commit: Thêm API đăng nhập
# 5c4b3a2 HEAD@{2}: commit: Thêm form đăng nhập
# 1234abc HEAD@{3}: checkout: moving from feature to main

git reflog show main       # reflog của riêng nhánh main
```

Hệ quả:

- **Mọi commit từng được một ref trỏ tới đều cứu được**, trong thời hạn reflog (mặc định 90 ngày; 30 ngày với commit đã unreachable).
- Reflog **chỉ ở local**, không được push hay clone.
- Xoá nhánh thì reflog của nhánh đó mất theo, nhưng reflog của **HEAD** vẫn còn (nếu bạn từng checkout nhánh đó).

### Mức độ an toàn của dữ liệu

| Dữ liệu | Nếu bị ghi đè hoặc xoá | Cứu được? |
|---------|------------------------|:---:|
| Commit (kể cả bị reset, amend, rebase) | Còn trong object DB + reflog | ✅ (reflog) |
| Thay đổi **đã stage** nhưng chưa commit | Blob còn trong object DB, nhưng không có tên file | ⚠️ khó (`git fsck --lost-found`) |
| Thay đổi **chưa stage** | Chưa từng vào Git | ❌ |
| File untracked bị `clean` | Chưa từng vào Git | ❌ |
| Stash bị `drop` | Commit unreachable | ⚠️ (`git fsck --unreachable`) |

**Bài học:** commit sớm, commit thường xuyên (kể cả commit tạm). Commit gần như không bao giờ mất. Thứ chưa commit thì có.

## 🧭 Chọn lệnh nào?

```
Muốn hoàn tác gì?
│
├─ Thay đổi trong file, chưa stage ─────────────► git restore <f>
├─ Đã stage, muốn bỏ stage ─────────────────────► git restore --staged <f>
├─ Lấy lại phiên bản cũ của một file ───────────► git restore --source=<c> <f>
│
├─ Commit CHƯA push
│   ├─ Sửa message / thêm file vào commit cuối ─► git commit --amend (bài 11)
│   ├─ Bỏ commit, giữ thay đổi ─────────────────► git reset [--soft] HEAD~1
│   └─ Bỏ commit và thay đổi ───────────────────► git reset --hard HEAD~1
│
├─ Commit ĐÃ push lên nhánh chung ──────────────► git revert <c>
│
└─ "Tôi làm hỏng rồi, muốn quay lại lúc nãy" ───► git reflog → git reset --hard HEAD@{n}
```

## 🧪 Thực hành

```bash
cd ~/git-lab && rm -rf undo && git init undo && cd undo
for i in 1 2 3; do echo $i > f$i; git add .; git commit -qm "C$i"; done
git log --oneline

# 1. Ba chế độ reset
echo staged > f3 && git add f3 && echo unstaged > f1

git reset --soft HEAD~1
git log --oneline               # C3 biến mất khỏi log
git status --short
#  M f1      ← working tree không bị động tới
# A  f3      ← index vẫn giữ f3 (bản "staged"), nhưng HEAD giờ là C2 chưa có f3
git reset --hard ORIG_HEAD      # quay về như cũ (bỏ cả thay đổi local)

echo unstaged > f1
git reset --mixed HEAD~1
git status --short
#  M f1      ← working tree không bị động tới
# ?? f3      ← index bị đặt lại theo C2 (không có f3), file trên đĩa thành untracked
git reset --hard ORIG_HEAD

git reset --hard HEAD~1
git status --short              # sạch, f3 không còn
ls                              # f1 f2

# 2. reset --hard không xoá file untracked
echo rac > rac.txt
git reset --hard
ls                              # rac.txt vẫn còn
git clean -n && git clean -f

# 3. Cứu commit C3 bằng reflog
git reflog | head -5            # lưu ý: cả "reset --hard" ở bước 2 cũng được ghi lại
git log -g --format='%h %gd %gs | %s' | head -8   # reflog kèm message của commit
git reset --hard <hash của C3>  # đừng đoán HEAD@{n}, hãy đọc reflog để chọn đúng
git log --oneline               # C3 đã trở lại

# 4. revert
git revert --no-edit HEAD~1     # đảo ngược C2
git log --oneline               # C1 C2 C3 Revert "C2"
ls                              # f2 đã bị xoá bởi commit revert

# 5. Cứu một nhánh bị xoá bằng -D
git switch -qc tam && echo x > x && git add x && git commit -qm "X quan trọng"
git switch -q main && git branch -D tam
git reflog | grep "X quan trọng"
git branch tam <hash>           # tạo lại nhánh
```

Ở bước 1, đọc kỹ `git status` sau mỗi lần reset và giải thích từng dòng bằng bảng ba chế độ.

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "`reset` xoá commit" | `reset` di chuyển nhánh. Commit vẫn còn và cứu được qua reflog. |
| "`reset --hard` dọn sạch thư mục" | Nó không đụng tới file untracked. Dùng `git clean`. |
| "`--mixed` làm thay đổi thành untracked" | Thay đổi thành **unstaged**. Chỉ những file chưa có ở commit đích mới thành untracked. |
| "`revert` quay repo về trạng thái của commit đó" | `revert C` chỉ đảo ngược **thay đổi của riêng C**. Muốn đưa file về trạng thái cũ thì dùng `restore --source`. |
| "`git revert` cho merge commit giống commit thường" | Phải chọn cha bằng `-m`, và muốn merge lại thì phải revert cái revert trước. |

## ✅ Tự kiểm tra

<details>
<summary>1. Đang ở <code>main</code>, chạy <code>git reset --hard HEAD~2</code>. Những ref nào thay đổi?</summary>

Nhánh `main` (trỏ về 2 commit trước), `ORIG_HEAD` (lưu vị trí cũ). HEAD vẫn là `ref: refs/heads/main`. Reflog của HEAD và `main` có thêm một dòng.
</details>

<details>
<summary>2. Commit cuối chưa push chứa một file mật khẩu. Làm sao bỏ file đó mà giữ các thay đổi khác của commit?</summary>

`git rm --cached secret.txt` → thêm vào `.gitignore` → `git commit --amend`. Hoặc `git reset HEAD~1` rồi commit lại không có file đó. (Nếu **đã push**, phải đổi mật khẩu ngay: coi như đã lộ.)
</details>

<details>
<summary>3. Vì sao không nên dùng <code>reset</code> để hoàn tác commit đã push lên <code>main</code> dùng chung?</summary>

`reset` viết lại lịch sử: phải force push, và đồng đội đã có commit cũ sẽ bị lệch lịch sử. `revert` chỉ thêm commit mới nên an toàn.
</details>

<details>
<summary>4. Lỡ <code>git reset --hard</code> khi đang có thay đổi đã <code>add</code> nhưng chưa commit. Còn hi vọng không?</summary>

Có một chút. Blob đã được ghi lúc `add`. `git fsck --lost-found` sẽ đưa các blob "treo" (dangling) vào `.git/lost-found/other/`, nhưng không kèm tên file. Thay đổi chưa `add` thì mất hẳn.
</details>

<details>
<summary>5. <code>git reset HEAD~1</code> và <code>git reset HEAD~1 -- file.txt</code> khác nhau thế nào?</summary>

Lệnh đầu di chuyển nhánh về HEAD~1 và đặt lại index. Lệnh sau không di chuyển nhánh, chỉ chép `file.txt` từ HEAD~1 vào index.
</details>

## 🏋️ Bài tập

1. Gộp 4 commit cuối thành một chỉ bằng `reset --soft` và `commit`. Kiểm tra `git diff ORIG_HEAD HEAD` rỗng.
2. Tạo một merge commit, revert nó bằng `-m 1`, rồi thử merge lại nhánh đó. Quan sát điều gì xảy ra, và sửa bằng cách revert cái revert.
3. Dùng `git fsck --lost-found` để cứu nội dung một file đã `add` rồi bị `reset --hard`.
4. Viết lại cây quyết định "Chọn lệnh nào" theo cách của bạn, rồi so với bài.

## 📎 Tra nhanh

- [Cheat sheet 08: Hoàn tác thay đổi & Reflog](../cheat-sheet/08-hoan-tac-va-reflog.md)
- [Cheat sheet 14: Dọn dẹp](../cheat-sheet/14-bao-tri-va-don-dep.md)
- Pro Git: [7.7 Reset Demystified](https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified), [2.4 Undoing Things](https://git-scm.com/book/en/v2/Git-Basics-Undoing-Things)

---

[← Trước: Merge và conflict](08-merge-va-conflict.md) · [Lộ trình](../README.md) · [Tiếp: Remote, fetch, push →](../3-cong-tac/10-remote-fetch-push.md)
