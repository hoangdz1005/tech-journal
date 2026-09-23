# 05. Đồ thị commit và cú pháp revision

[← Lộ trình](../README.md) · Giai đoạn 1: Nền tảng · ⏱️ ~45 phút

## 🎯 Mục tiêu

- Hiểu lịch sử Git là một **đồ thị có hướng không chu trình** (DAG).
- Hiểu khái niệm **reachable** (truy cập được), nền tảng của `log`, `merge`, `push`, `gc`.
- Chỉ định chính xác bất kỳ commit nào: `~`, `^`, `@{}`, `:`.
- Dùng đúng khoảng `A..B` và `A...B`, và biết chúng mang nghĩa khác nhau trong `log` và `diff`.

## ❓ Vấn đề

"Cho mình xem những commit có trên nhánh feature mà chưa có trên main." "Commit trước commit trước nữa là gì?" "Hai nhánh này tách nhau từ đâu?" Để trả lời, cần hiểu hình dạng của lịch sử và có ngôn ngữ để chỉ vào từng điểm trên đó.

## 🧠 Khái niệm

### Lịch sử là một DAG

Mỗi commit trỏ tới 0, 1 hoặc nhiều commit cha:

- **0 cha**: root commit (commit đầu tiên). Một repo có thể có nhiều root.
- **1 cha**: commit thường.
- **2+ cha**: merge commit.

```
        A ◄── B ◄── C ◄──────── M ◄── N   ◄── main
                     ▲          │
                     └── D ◄── E ◄─┘          ◄── feature (đã merge)
```

Mũi tên luôn **chỉ về quá khứ** (con biết cha, cha không biết con). Không thể có chu trình, vì muốn trỏ tới một commit thì commit đó phải tồn tại trước (và hash của nó đã cố định).

Hệ quả: từ một commit, Git chỉ đi được **ngược về quá khứ**. Muốn tìm "con" của một commit, Git phải xuất phát từ các ref rồi đi ngược xuống.

### Reachable: khái niệm then chốt

Commit `X` **reachable** từ commit `Y` nếu đi theo con trỏ cha từ `Y` sẽ gặp `X` (tính cả `Y`).

- `git log main` = liệt kê mọi commit reachable từ `main`.
- "Nhánh `feature` đã được merge vào `main`" = đầu nhánh `feature` reachable từ `main`.
- "Commit bị mất" = không reachable từ **bất kỳ** ref nào (kể cả reflog). `git gc` chỉ xoá những commit như vậy (bài 15).
- Push fast-forward = commit cũ trên remote reachable từ commit mới (bài 10).

### Merge base

**Merge base** của hai commit là tổ tiên chung "gần nhất" của chúng. Đây là điểm hai nhánh tách ra, và là mốc để `merge` và `rebase` tính toán (bài 08, 11).

```
        A ◄── B ◄── C ◄── F       ◄── main
                    ▲
                    └── D ◄── E   ◄── feature

git merge-base main feature   →   C
```

## 🔬 Cú pháp chỉ định commit (revision)

Tham khảo đầy đủ: `git help revisions`. Các dạng hay dùng:

### Chỉ định một commit

| Cú pháp | Ý nghĩa | Ví dụ |
|---------|---------|-------|
| `<hash>` | Hash đầy đủ hoặc rút gọn (đủ để không trùng, tối thiểu 4 ký tự) | `871b580` |
| `<ref>` | Branch, tag, remote-tracking | `main`, `v1.0`, `origin/main` |
| `HEAD` hoặc `@` | Vị trí hiện tại | |
| `<rev>~<n>` | Đi ngược n bước theo **cha thứ nhất** | `HEAD~3` |
| `<rev>^<n>` | **Cha thứ n** (chỉ có ý nghĩa với merge commit) | `M^2` |
| `<rev>^` | = `<rev>^1` = `<rev>~1` | `HEAD^` |
| `<ref>@{<n>}` | Giá trị của ref ở n lần thay đổi trước (reflog) | `HEAD@{2}`, `main@{1}` |
| `<ref>@{<thời gian>}` | Giá trị của ref tại thời điểm đó (theo reflog **local**) | `main@{yesterday}` |
| `@{-<n>}` | Nhánh đã checkout n lần trước | `@{-1}` (giống `git switch -`) |
| `<nhánh>@{upstream}` / `@{u}` | Nhánh upstream (bài 10) | `@{u}` |
| `<rev>:<path>` | Blob/tree tại đường dẫn trong commit | `HEAD:src/app.js` |
| `:<path>` | Blob trong index | `:a.txt` |
| `<rev>^{tree}` | Tree của commit | `HEAD^{tree}` |
| `:/<chuỗi>` | Commit gần nhất có message khớp | `:/fix login` |

### `~` và `^` khác nhau thế nào

Với commit thường (1 cha), `~1` và `^1` như nhau. Khác biệt chỉ lộ ra ở merge commit:

```
          G   H   I   J
           \ /     \ /
            D   E   F
             \  |  / \
              \ | /   |
               \|/    |
                B     C
                 \   /
                  \ /
                   A          A (mới nhất) là merge commit có 2 cha: B (cha 1), C (cha 2)
                              B là merge commit có 3 cha: D, E, F

A^  = A^1 = A~1 = B       A^2 = C
A^^ = A~2 = D             A^^2 = B^2 = E       A^^3 = B^3 = F
A~3 = G                   A~2^2 = D^2 = H      A^^3^2 = F^2 = J
```

(Hình lấy từ `git help revisions`, trong đó commit mới nằm ở dưới.)

Quy tắc dễ nhớ: `~` = **đi lên theo chiều dọc** (luôn theo cha đầu). `^n` = **rẽ ngang** sang cha thứ n.

Cha thứ nhất của merge commit là **nhánh bạn đang đứng khi merge**. Nhờ đó `git log --first-parent main` cho thấy lịch sử "chính" của `main`, bỏ qua chi tiết bên trong các nhánh đã merge.

### Chỉ định một tập commit (dùng với `log`, `rev-list`)

| Cú pháp | Tập commit | Câu hỏi trả lời |
|---------|-----------|-----------------|
| `B` | Reachable từ B | Lịch sử của B |
| `^A B` hay `B --not A` | Reachable từ B, trừ reachable từ A | |
| `A..B` | = `^A B` | "B có gì mà A chưa có?" |
| `A...B` | Reachable từ A **hoặc** B, trừ reachable từ **cả hai** | "Hai bên khác nhau những gì?" |

```
        A ◄── B ◄── C ◄── F ◄── G       ◄── main
                    ▲
                    └── D ◄── E         ◄── feature

git log main..feature      →  D, E       (feature có, main chưa có)
git log feature..main      →  F, G
git log main...feature     →  D, E, F, G
git log --left-right --oneline main...feature
                           →  < G   > E   < F   > D    ("<" thuộc bên trái, ">" thuộc bên phải)
```

Ứng dụng hằng ngày:

```bash
git log origin/main..HEAD     # commit local chưa push
git log HEAD..origin/main     # commit trên remote chưa kéo về
git log main..                # bỏ trống một đầu = HEAD
```

### ⚠️ `..` và `...` trong `git diff` mang nghĩa khác

`git diff` so sánh **hai snapshot**, không làm việc với tập commit, nên hai ký hiệu bị "mượn" với nghĩa khác:

| Lệnh | Thực chất |
|------|-----------|
| `git diff A B` | So snapshot A với snapshot B |
| `git diff A..B` | **Giống hệt** `git diff A B` |
| `git diff A...B` | So `merge-base(A, B)` với B: "B đã thay đổi gì kể từ lúc tách khỏi A" |

`git diff main...feature` là đúng cái bạn thấy trong một Pull Request: chỉ các thay đổi của feature, không lẫn thay đổi mới của main.

## 🧪 Thực hành

```bash
cd ~/git-lab && rm -rf dag && git init dag && cd dag
c() { git commit --allow-empty -q -m "$1"; }   # hàm tạo commit rỗng cho nhanh

c A; c B; c C
git switch -q -c feature
c D; c E
git switch -q main
c F; c G

git log --oneline --graph --all

# 1. Merge base
git merge-base main feature                 # hash của C
git log -1 --format=%s $(git merge-base main feature)   # C

# 2. Khoảng
git log --oneline main..feature             # E D
git log --oneline feature..main             # G F
git log --oneline --left-right main...feature

# 3. Merge để có commit nhiều cha
git merge -q --no-ff feature -m "M"
git log -1 --format='%s | cha: %p' HEAD     # M | cha: <G> <E>
git log -1 --format=%s HEAD^1               # G
git log -1 --format=%s HEAD^2               # E
git log -1 --format=%s HEAD~2               # F
git log -1 --format=%s HEAD^2~1             # D

# 4. first-parent
git log --oneline --first-parent            # M G F C B A (không có D, E)

# 5. Kiểm tra "đã merge chưa"
git merge-base --is-ancestor feature main && echo "feature đã merge vào main"
git branch --merged main                    # các nhánh có đầu reachable từ main

# 6. Chuyển revision thành hash
git rev-parse HEAD~2 HEAD^2 HEAD^{tree}
git rev-parse --abbrev-ref HEAD             # tên nhánh hiện tại
```

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "`HEAD~2` và `HEAD^2` giống nhau" | `~2` = ông (theo cha đầu). `^2` = cha thứ hai (chỉ tồn tại ở merge commit). |
| "`git log A..B` là các commit nằm giữa A và B" | Là các commit reachable từ B nhưng **không** reachable từ A. A không nhất thiết là tổ tiên của B. |
| "`git diff A..B` chỉ ra thay đổi của các commit trong `A..B`" | Nó chỉ so snapshot A với snapshot B. Muốn "thay đổi của B kể từ khi tách" thì dùng `A...B`. |
| "`main@{yesterday}` là main trên server hôm qua" | Nó đọc reflog **local**: giá trị main **trên máy bạn** lúc hôm qua. |
| "Git biết commit nào là con của commit nào" | Commit chỉ trỏ tới cha. Tìm con phải đi từ các ref xuống. |

## ✅ Tự kiểm tra

<details>
<summary>1. Viết lệnh liệt kê các commit bạn đã commit local nhưng chưa push lên <code>origin/main</code>.</summary>

`git log origin/main..HEAD` (hoặc `git log @{u}..`).
</details>

<details>
<summary>2. <code>M</code> là merge commit tạo ra khi đứng ở <code>main</code> và chạy <code>git merge feature</code>. <code>M^1</code> và <code>M^2</code> là gì?</summary>

`M^1` = đầu nhánh `main` trước khi merge. `M^2` = đầu nhánh `feature` lúc merge.
</details>

<details>
<summary>3. Trong Pull Request từ <code>feature</code> vào <code>main</code>, diff hiển thị tương đương lệnh nào?</summary>

`git diff main...feature`: so merge base với đầu `feature`.
</details>

<details>
<summary>4. Nhánh <code>old</code> bị xoá nhưng mọi commit của nó đã được merge vào <code>main</code>. Commit của <code>old</code> có bị <code>gc</code> xoá không?</summary>

Không. Chúng vẫn reachable từ `main` (qua cha thứ hai của merge commit).
</details>

## 🏋️ Bài tập

1. Tạo một lịch sử có ít nhất 2 merge commit. Viết 5 biểu thức revision khác nhau cùng trỏ tới một commit, kiểm chứng bằng `git rev-parse`.
2. So sánh output của `git diff main..feature` và `git diff main...feature` sau khi `main` có thêm commit mới. Giải thích vì sao khác.
3. Dùng `git rev-list --count main..feature` và `git rev-list --count feature..main` để in ra "feature đi trước N commit, đi sau M commit". Đó chính là cách `git status` tính "ahead/behind".

## 📎 Tra nhanh

- [Cheat sheet 09: Lịch sử](../cheat-sheet/09-lich-su.md)
- [Cheat sheet 10: Diff](../cheat-sheet/10-diff.md)
- `git help revisions`, `git help gitrevisions`
- Pro Git: [7.1 Revision Selection](https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection)

---

[← Trước: Index và ba cây](04-index-ba-cay.md) · [Lộ trình](../README.md) · [Tiếp: add, commit, status, diff →](../2-thao-tac-hang-ngay/06-add-commit-status-diff.md)
