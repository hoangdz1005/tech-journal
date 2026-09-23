# 07. Branch, switch, checkout

[← Lộ trình](../README.md) · Giai đoạn 2: Thao tác hằng ngày · ⏱️ ~40 phút

## 🎯 Mục tiêu

- Hiểu chính xác `git switch` cập nhật những gì và theo thứ tự nào.
- Giải thích vì sao thay đổi chưa commit đôi khi "đi theo" bạn sang nhánh mới, đôi khi khiến Git từ chối chuyển nhánh.
- Biết vì sao `git checkout` được tách thành `switch` và `restore`.
- Quản lý nhánh: tạo, đổi tên, xoá an toàn, tìm nhánh chứa một commit.

> Yêu cầu: [bài 03](../1-nen-tang/03-refs-branch-head.md) (ref, HEAD) và [bài 04](../1-nen-tang/04-index-ba-cay.md) (ba cây).

## ❓ Vấn đề

Bạn đang sửa dở trên `main`, gõ `git switch feature` và... thay đổi vẫn còn nguyên trong working tree. Lần khác thì Git báo *"Your local changes would be overwritten"* và từ chối. Hành vi này không ngẫu nhiên, nó theo một quy tắc rất rõ ràng.

## 🧠 Khái niệm

### Tạo và quản lý nhánh

Nhắc lại bài 03: nhánh = file ref trỏ tới commit. Mọi lệnh quản lý nhánh chỉ thao tác trên các ref này.

| Lệnh | Tác dụng |
|------|----------|
| `git branch` | Liệt kê nhánh local (`*` là nhánh hiện tại) |
| `git branch -vv` | Kèm commit đầu nhánh và upstream, ahead/behind |
| `git branch -a` | Cả remote-tracking branch |
| `git branch <tên> [<điểm bắt đầu>]` | Tạo nhánh, **không** chuyển sang |
| `git branch -m <cũ> <mới>` | Đổi tên |
| `git branch -d <tên>` | Xoá, **chỉ khi** đầu nhánh đã reachable từ upstream của nó hoặc từ HEAD |
| `git branch -D <tên>` | Xoá bắt buộc |
| `git branch --merged` / `--no-merged` | Nhánh đã / chưa được merge vào HEAD |
| `git branch --contains <commit>` | Nhánh nào chứa commit này |

`-d` an toàn vì nó từ chối xoá nếu việc xoá khiến commit trở thành unreachable. `-D` bỏ qua kiểm tra đó.

### `git switch`: đổi HEAD và đồng bộ index + working tree

`git switch feature` làm ba việc:

1. **Cập nhật working tree và index** để khớp với tree của commit `feature`.
2. **Ghi `ref: refs/heads/feature` vào HEAD.**
3. Ghi một dòng vào reflog của HEAD (`checkout: moving from main to feature`).

Bước 1 **không** xoá sạch rồi chép lại toàn bộ. Git chỉ động vào những file **khác nhau** giữa tree hiện tại (HEAD cũ) và tree đích. Đây là chìa khoá để hiểu mọi hành vi còn lại.

### Quy tắc với thay đổi chưa commit

Với mỗi file, xét hai câu hỏi:

- **(1)** File này có khác nhau giữa commit hiện tại và commit đích không?
- **(2)** Bạn có thay đổi chưa commit (trong index hoặc working tree) ở file này không?

| (1) Khác giữa hai nhánh? | (2) Có thay đổi local? | Kết quả |
|:---:|:---:|---|
| Không | Không | Không động tới |
| Có | Không | Git cập nhật file theo nhánh đích |
| Không | Có | **Thay đổi đi theo bạn** sang nhánh mới, vì Git không cần động tới file này |
| Có | Có | **Từ chối chuyển nhánh**, vì cập nhật file sẽ đè mất thay đổi của bạn |

```
main:     a=1  b=1          feature:  a=1  b=2
Bạn sửa a=local (chưa commit)

git switch feature
  a: giống nhau giữa hai nhánh, có sửa local → giữ nguyên, đi theo   ✅
  b: khác nhau, không sửa local            → cập nhật thành 2          ✅

Nếu bạn sửa b=local:
  b: khác nhau, có sửa local               → TỪ CHỐI ❌
  "Your local changes to the following files would be overwritten by checkout: b"
```

Tương tự với **file untracked**: nếu nhánh đích có một file tracked cùng đường dẫn với file untracked của bạn, Git từ chối (*"untracked working tree files would be overwritten"*).

Cách xử lý khi bị từ chối:

- Commit thay đổi (có thể commit tạm, sau đó amend).
- `git stash` rồi `git stash pop` sau khi chuyển (bài 13).
- `git switch --merge feature`: thử merge thay đổi local vào phiên bản của nhánh đích (có thể gây conflict).
- `git switch --discard-changes feature` (hoặc `-f`): **bỏ** thay đổi local. Nguy hiểm.

### Vì sao có `switch` và `restore`?

`git checkout` làm hai việc rất khác nhau tuỳ tham số:

| Lệnh cũ | Việc thực sự làm | Lệnh mới (Git ≥ 2.23) |
|---------|------------------|-----------------------|
| `git checkout feature` | Đổi nhánh (đổi HEAD) | `git switch feature` |
| `git checkout -b feat` | Tạo và đổi nhánh | `git switch -c feat` |
| `git checkout <commit>` | Detached HEAD | `git switch --detach <commit>` |
| `git checkout -- file` | Chép file từ **index** ra working tree. HEAD không đổi | `git restore file` |
| `git checkout <commit> -- file` | Chép file từ commit vào **index và** working tree | `git restore --source=<commit> --staged --worktree file` |

Điểm nguy hiểm: `git checkout foo` là đổi nhánh nếu có nhánh `foo`, nhưng là **ghi đè file** `foo` nếu có file tên `foo`. `switch` và `restore` tách bạch hai việc này. Lộ trình này dùng lệnh mới, nhưng bạn vẫn cần đọc hiểu `checkout` vì nó còn rất phổ biến.

### Một vài dạng switch đặc biệt

```bash
git switch -                       # quay lại nhánh trước (= @{-1})
git switch -c feat origin/feat     # tạo nhánh local từ remote-tracking, tự đặt upstream
git switch feat                    # nếu chỉ có origin/feat, Git tự tạo nhánh local (--guess)
git switch --detach v1.0           # xem code ở một tag
git switch --orphan gh-pages       # nhánh mới KHÔNG có lịch sử, bắt đầu từ thư mục trống
```

## 🔬 Bên trong Git

```
git switch feature
  │
  ├─ Đọc tree của HEAD hiện tại (T_cũ) và tree của feature (T_mới)
  ├─ So T_cũ với T_mới → danh sách file cần thay đổi
  ├─ Kiểm tra từng file trong danh sách với index + working tree
  │     └─ có thay đổi local → huỷ toàn bộ, không đổi gì
  ├─ Cập nhật index và working tree cho các file trong danh sách
  ├─ HEAD ← "ref: refs/heads/feature"
  └─ Reflog HEAD: "checkout: moving from main to feature"
```

Để ý: kiểm tra xảy ra **trước khi** thay đổi bất cứ thứ gì. Nếu bị từ chối, repo vẫn y như cũ.

Chuyển nhánh nhanh hay chậm phụ thuộc vào **số file khác nhau** giữa hai nhánh, không phụ thuộc vào kích thước repo.

## 🧪 Thực hành

```bash
cd ~/git-lab && rm -rf sw && git init sw && cd sw
echo 1 > a && echo 1 > b && git add . && git commit -qm init
git switch -qc feat && echo 2 > b && git commit -qam "feat sửa b"
git switch -q main

# 1. Thay đổi ở file giống nhau giữa hai nhánh: đi theo
echo local > a
git switch feat                   # Switched... và "M  a"
git status --short                #  M a
git switch -q main && git restore a

# 2. Thay đổi ở file khác nhau giữa hai nhánh: bị từ chối
echo local > b
git switch feat                   # error: Your local changes ... would be overwritten
git status --short                # vẫn ở main, không có gì bị đổi

# 3. Thử --merge
git switch --merge feat           # Git cố gộp "local" vào b=2 → conflict
git status --short                # UU b
git switch -q --discard-changes main   # bỏ hết, quay về main sạch

# 4. File untracked bị chặn
git switch -q feat && echo c > c && git add c && git commit -qm "thêm c"
git switch -q main
echo "của tôi" > c                # untracked trên main
git switch feat                   # error: untracked working tree files would be overwritten
rm c

# 5. Xoá nhánh an toàn
git branch -d feat                # error: not fully merged
git branch --no-merged            # feat
git merge -q feat && git branch -d feat   # giờ thì xoá được

# 6. Nhánh mồ côi
git switch --orphan docs
ls; git status --short            # trống: switch --orphan xoá mọi file tracked khỏi index và đĩa
echo "# Docs" > README.md && git add . && git commit -qm "docs root"
git log --oneline --graph --all   # hai lịch sử tách rời
```

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "Thay đổi chưa commit thuộc về nhánh" | Thay đổi chưa commit nằm ở index/working tree, không thuộc nhánh nào. Chúng đi theo bạn khi Git không cần động tới file đó. |
| "Chuyển nhánh sẽ ghi lại toàn bộ thư mục" | Chỉ file khác nhau giữa hai commit mới bị động tới. |
| "`git branch feat` chuyển sang feat" | Chỉ tạo nhánh. Dùng `switch -c` để tạo và chuyển. |
| "`git checkout -- file` lấy file từ commit cuối" | Nó lấy từ **index**. Nếu đã stage thì lấy bản đã stage. |
| "`branch -d` không bao giờ mất dữ liệu" | Đúng với commit, nhưng upstream của nhánh cũng có thể là tiêu chí "đã merge". Luôn đọc thông báo. |

## ✅ Tự kiểm tra

<details>
<summary>1. Bạn sửa <code>config.js</code> trên <code>main</code>. <code>config.js</code> giống nhau ở <code>main</code> và <code>feature</code>. Chuyển sang <code>feature</code> rồi commit. Thay đổi đó nằm ở nhánh nào?</summary>

`feature`. Thay đổi đi theo bạn và được commit lên nhánh đang checkout.
</details>

<details>
<summary>2. Bị từ chối chuyển nhánh. Git có sửa file nào trước khi báo lỗi không?</summary>

Không. Git kiểm tra toàn bộ trước, rồi mới thay đổi.
</details>

<details>
<summary>3. Viết lệnh thay cho <code>git checkout main -- package.json</code>, và nói lệnh đó thay đổi những cây nào.</summary>

`git restore --source=main --staged --worktree package.json`. Nó chép bản ở `main` vào index và working tree. HEAD không đổi.
</details>

<details>
<summary>4. <code>git switch --orphan x</code> và <code>git checkout --orphan x</code> khác nhau thế nào?</summary>

Cả hai đều trỏ HEAD tới nhánh `x` chưa tồn tại (commit tiếp theo sẽ là root commit). `switch --orphan` còn **xoá sạch** index và file tracked, cho bạn bắt đầu từ thư mục trống. `checkout --orphan` giữ nguyên index và working tree, nên `git status` sẽ thấy mọi file là "file mới đã stage" (vì HEAD rỗng).
</details>

## 🏋️ Bài tập

1. Tạo tình huống thay đổi đã **stage** (không chỉ sửa) ở một file giống nhau giữa hai nhánh, rồi switch. Trạng thái staged có được giữ không?
2. Tìm mọi nhánh chứa một commit sửa bug cụ thể bằng `git branch --contains`.
3. Viết alias `git cleanup` xoá mọi nhánh local đã merge vào `main` (trừ `main`). Gợi ý: `git branch --merged main`.

## 📎 Tra nhanh

- [Cheat sheet 02: Branch & Checkout](../cheat-sheet/02-branch.md)
- Pro Git: [3.2 Basic Branching and Merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging), [3.3 Branch Management](https://git-scm.com/book/en/v2/Git-Branching-Branch-Management)

---

[← Trước: add, commit, status, diff](06-add-commit-status-diff.md) · [Lộ trình](../README.md) · [Tiếp: Merge và conflict →](08-merge-va-conflict.md)
