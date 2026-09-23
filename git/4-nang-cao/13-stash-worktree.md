# 13. Stash và worktree

[← Lộ trình](../README.md) · Giai đoạn 4: Nâng cao · ⏱️ ~40 phút

## 🎯 Mục tiêu

- Hiểu stash thực chất là **các commit đặc biệt**, nhờ đó biết nó an toàn tới đâu.
- Dùng stash đúng cách, kể cả với file untracked và khi muốn giữ trạng thái staged.
- Dùng `worktree` để làm việc trên nhiều nhánh cùng lúc mà không cần stash hay clone lại.

> Yêu cầu: [bài 02](../1-nen-tang/02-object-model.md) (commit object), [bài 04](../1-nen-tang/04-index-ba-cay.md) (ba cây), [bài 07](../2-thao-tac-hang-ngay/07-branch-switch-checkout.md) (switch bị từ chối).

## ❓ Vấn đề

Bạn đang code dở thì có bug khẩn trên production. Code dở chưa đủ để commit, và `git switch` bị từ chối vì thay đổi local sẽ bị ghi đè. Bạn cần "cất tạm" công việc dở dang, hoặc có một thư mục thứ hai để xử lý song song.

## 🧠 Stash

### Dùng cơ bản

| Lệnh | Tác dụng |
|------|----------|
| `git stash` / `git stash push -m "msg"` | Cất thay đổi của file **tracked** (staged + unstaged), đưa index và working tree về HEAD |
| `git stash -u` | Cất cả file **untracked** |
| `git stash -a` | Cất cả file bị **ignore** |
| `git stash push -- <path>` | Chỉ cất một số file |
| `git stash -p` | Chọn từng hunk để cất |
| `git stash --keep-index` | Cất nhưng **giữ lại** phần đã stage trong index và working tree |
| `git stash list` | Danh sách stash (`stash@{0}` là mới nhất) |
| `git stash show -p stash@{1}` | Xem nội dung một stash |
| `git stash apply [stash@{n}]` | Áp stash, **giữ** stash trong danh sách |
| `git stash pop [stash@{n}]` | Áp stash rồi **xoá** khỏi danh sách (nếu áp thành công) |
| `git stash pop --index` | Khôi phục đúng cả trạng thái staged/unstaged |
| `git stash drop stash@{n}` | Xoá một stash |
| `git stash branch <tên> [stash@{n}]` | Tạo nhánh từ commit lúc stash, áp stash lên đó |

### 🔬 Bên trong: stash là commit

`git stash` tạo ra **2 hoặc 3 commit**:

```
            W  "On main: thử"             ← trạng thái working tree (file tracked)
          / | \
         /  |  U  "untracked files on main"   ← chỉ khi dùng -u / -a (cha thứ 3)
        /   I  "index on main"               ← trạng thái index (cha thứ 2)
       /   /
      H ◄─┘                                  ← HEAD lúc stash (cha thứ 1)
```

```bash
git cat-file -p stash@{0}
# tree 79eb940...
# parent a5d6a92...     ← H: HEAD lúc stash
# parent 5b7b290...     ← I: commit chứa index
# parent a57f475...     ← U: commit chứa file untracked
# ...
git show stash@{0}^2:a      # nội dung a trong index lúc stash
git show stash@{0}:a        # nội dung a trong working tree lúc stash
```

`refs/stash` trỏ tới stash mới nhất. "Danh sách stash" thực chất là **reflog của `refs/stash`**: `stash@{1}` chính là cú pháp reflog (bài 05).

Hệ quả:

- `stash apply` thực chất là một **three-way merge** (base = H, ours = HEAD hiện tại, theirs = W), nên có thể conflict. Khi conflict, `pop` sẽ **không** xoá stash.
- Mặc định `apply`/`pop` **không khôi phục trạng thái staged**. Mọi thứ thành unstaged. Dùng `--index` để khôi phục cả index từ commit `I`.
- Stash bị `drop` hay `clear` trở thành commit **unreachable**. Có thể cứu được:
  ```bash
  git fsck --unreachable | grep commit     # tìm commit treo
  git log --graph --oneline $(git fsck --no-reflog | awk '/dangling commit/ {print $3}')
  git stash apply <hash>
  ```

### Khi nào **không** nên dùng stash

Stash không gắn với nhánh, dễ quên, dễ chồng chất (`stash@{7}` là gì?). Nếu công việc dở cần giữ quá vài phút:

- Commit tạm lên nhánh: `git commit -m "WIP"`, lúc quay lại thì `git reset HEAD~` (hoặc amend).
- Hoặc dùng **worktree**.

## 🧠 Worktree

### Vấn đề của một working tree duy nhất

Một repo bình thường chỉ có **một** working tree + **một** index + **một** HEAD. Muốn xem hai nhánh cùng lúc (chạy app bản production trong khi đang code tính năng mới), bạn phải stash/commit rồi switch qua lại, hoặc clone thêm một bản (tốn dung lượng, phải fetch riêng).

### Worktree: nhiều working tree, một repository

```bash
git worktree add ../app-hotfix -b hotfix origin/main   # tạo thư mục mới trên nhánh mới
git worktree add ../app-review feature/x               # checkout nhánh có sẵn
git worktree list
git worktree remove ../app-hotfix
git worktree prune                                      # dọn thông tin worktree đã bị xoá tay
```

```
~/code/app/            ← worktree chính
  .git/                ← repository thật (objects, refs)
    worktrees/
      app-hotfix/      ← HEAD, index, reflog riêng của worktree phụ
        HEAD
        index
~/code/app-hotfix/     ← worktree phụ
  .git                 ← FILE (không phải thư mục): "gitdir: ~/code/app/.git/worktrees/app-hotfix"
```

### 🔬 Chia sẻ gì, riêng gì

| Thành phần | Dùng chung | Riêng từng worktree |
|------------|:---:|:---:|
| Object database | ✅ | |
| Branch, tag, remote (`refs/`) | ✅ | |
| Cấu hình (`config`) | ✅ | |
| `HEAD` | | ✅ |
| Index | | ✅ |
| Working tree | | ✅ |
| Ref theo worktree (`refs/bisect`, `refs/worktree/*`...) | | ✅ |

Hệ quả:

- Commit ở worktree này thì worktree kia **thấy ngay** (chung object và ref), không cần fetch.
- Git **không cho** hai worktree cùng checkout một nhánh (`fatal: 'hotfix' is already checked out at ...`). Nếu cho, commit ở một bên sẽ di chuyển nhánh dưới chân bên kia, làm index của bên kia lệch với HEAD.
- Rất nhẹ: chỉ tạo thêm file cho working tree, không nhân bản lịch sử.

Ứng dụng: review PR mà không động tới code đang làm, chạy test dài trên một nhánh trong khi làm việc ở nhánh khác, so sánh hành vi hai phiên bản, chạy nhiều agent/tiến trình trên nhiều nhánh song song.

## 🧪 Thực hành

```bash
cd ~/git-lab && rm -rf st st-hotfix && git init st && cd st
echo 1 > a && git add a && git commit -qm init

# 1. Mổ xẻ stash
echo staged > a && git add a
echo wt > a
echo u > new
git status --short                     # MM a, ?? new
git stash push -u -m "thử"
git status --short                     # sạch
git log --oneline --graph --all        # thấy 3 commit của stash
git cat-file -p stash@{0}              # 3 dòng parent
git show stash@{0}^2:a                 # staged
git show stash@{0}:a                   # wt
git ls-tree stash@{0}^3                # new

# 2. pop mặc định và pop --index
git stash apply                        # không có --index
git status --short                     #  M a  ← mất trạng thái staged
git restore a && rm new
git stash pop --index
git status --short                     # MM a, ?? new  ← khôi phục đúng

# 3. Cứu stash đã drop
git stash -u && git stash drop        # working tree giờ sạch, stash "mất"
for h in $(git fsck --no-reflog | awk '/dangling commit/ {print $3}'); do
  git log -1 --format="%h %s" $h
done
# 21cffa7 WIP on main: a227807 init   ← stash vừa drop
# 2b2d6a0 On main: thử                ← stash "thử" đã pop ở bước 2 cũng thành dangling
git stash apply <hash của "WIP on main">
git status --short                     # thay đổi đã trở lại

# 4. Worktree
git add -A && git commit -qm "lưu lại"
git worktree add ../st-hotfix -b hotfix HEAD~1
cat ../st-hotfix/.git                  # gitdir: .../st/.git/worktrees/st-hotfix
git worktree list
git switch hotfix                      # fatal: already checked out
(cd ../st-hotfix && echo fix > a && git commit -qam "hotfix")
git log --oneline hotfix               # thấy ngay commit "hotfix" từ worktree chính
git worktree remove ../st-hotfix
```

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "`git stash` cất mọi thứ" | Mặc định chỉ cất file tracked. File mới cần `-u`. |
| "`stash pop` khôi phục y nguyên" | Mặc định mọi thay đổi thành unstaged. Cần `--index`. |
| "Stash gắn với nhánh" | Stash là commit trỏ tới HEAD lúc cất, áp được lên bất kỳ nhánh nào. |
| "Stash bị drop là mất" | Là commit unreachable, cứu được bằng `fsck` cho tới khi `gc` dọn. |
| "Worktree là một bản clone" | Worktree chia sẻ object và ref với repo chính. Chỉ HEAD, index, working tree là riêng. |

## ✅ Tự kiểm tra

<details>
<summary>1. <code>git stash -u</code> tạo ra những commit nào, và cha của commit chính là gì?</summary>

Ba commit: I (index), U (untracked), W (working tree). W có 3 cha: HEAD lúc stash, I và U.
</details>

<details>
<summary>2. <code>git stash pop</code> bị conflict. Stash còn trong danh sách không?</summary>

Còn. `pop` chỉ drop khi áp thành công. Giải conflict xong phải tự `git stash drop`.
</details>

<details>
<summary>3. Vì sao Git không cho hai worktree cùng checkout một nhánh?</summary>

Vì ref dùng chung. Commit ở worktree A di chuyển nhánh, khiến HEAD của worktree B đổi mà index và working tree của B không được cập nhật, làm trạng thái của B sai lệch.
</details>

## 🏋️ Bài tập

1. Dùng `git stash --keep-index` để chạy test **chỉ** trên phần đã stage (mô phỏng hook pre-commit), rồi khôi phục phần còn lại.
2. Stash trên `main`, chuyển sang nhánh khác mà file đó đã bị sửa, rồi `stash pop` để tạo conflict. Giải quyết và dọn stash.
3. Thiết lập hai worktree cho hai nhánh, chạy một lệnh dài (ví dụ `sleep 30`) ở một bên trong khi commit ở bên kia.

## 📎 Tra nhanh

- [Cheat sheet 06: Stash](../cheat-sheet/06-stash.md)
- [Cheat sheet 17: Worktree](../cheat-sheet/17-worktree.md)
- Pro Git: [7.3 Stashing and Cleaning](https://git-scm.com/book/en/v2/Git-Tools-Stashing-and-Cleaning)

---

[← Trước: Quy trình làm việc nhóm](../3-cong-tac/12-quy-trinh-nhom.md) · [Lộ trình](../README.md) · [Tiếp: Điều tra lịch sử →](14-dieu-tra-lich-su.md)
