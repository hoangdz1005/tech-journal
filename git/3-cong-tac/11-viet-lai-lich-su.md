# 11. Viết lại lịch sử: rebase, cherry-pick, amend

[← Lộ trình](../README.md) · Giai đoạn 3: Cộng tác phân tán · ⏱️ ~75 phút

## 🎯 Mục tiêu

- Hiểu rằng Git **không thể sửa** commit, chỉ tạo commit mới rồi di chuyển ref.
- Mô tả chính xác `cherry-pick` và `rebase` làm gì từng bước.
- Dùng thành thạo interactive rebase: reword, squash, fixup, edit, drop, reorder.
- Dùng `rebase --onto` cho các tình huống chuyển nhánh phức tạp.
- Biết **khi nào không được** viết lại lịch sử.

> Yêu cầu: [bài 02](../1-nen-tang/02-object-model.md) (object bất biến), [bài 08](../2-thao-tac-hang-ngay/08-merge-va-conflict.md) (three-way merge), [bài 10](10-remote-fetch-push.md) (push, force push).

## ❓ Vấn đề

Bạn có 7 commit "wip", "fix", "fix lại" trên nhánh feature và muốn gộp thành 2 commit sạch sẽ trước khi mở PR. Hoặc nhánh feature tách ra từ `main` hai tuần trước, và bạn muốn nó "như thể" vừa được tạo từ `main` mới nhất. Hoặc cần mang đúng một commit sửa bug từ `develop` sang `release`.

Tất cả đều là **viết lại lịch sử**. Nhưng bài 02 đã chỉ ra object là bất biến. Vậy "viết lại" thực chất là gì?

## 🧠 Khái niệm

### Nguyên lý chung: tạo mới rồi di chuyển ref

Mọi thao tác "sửa lịch sử" đều theo một khuôn:

1. Tạo **commit mới** (hash mới) với nội dung/cha/message mong muốn.
2. **Di chuyển nhánh** sang commit mới.
3. Commit cũ vẫn còn trong object database, chỉ không còn được nhánh trỏ tới (reflog vẫn nhớ).

```
Trước:  A ◄── B ◄── C          ◄── main
Amend:  A ◄── B ◄── C          (C vẫn còn, chỉ không còn nhánh nào trỏ tới)
                  ▲
                  └── C'       ◄── main
```

Vì hash của một commit phụ thuộc vào cha của nó (bài 02), **thay đổi một commit thì mọi commit phía sau cũng phải được tạo lại**, kể cả khi nội dung của chúng không đổi.

### `git commit --amend`

Tạo commit mới **thay thế** commit cuối: cùng cha với commit cũ, tree lấy từ index hiện tại, message cũ (hoặc message mới).

```bash
git commit --amend -m "Message mới"      # sửa message
git add quen.txt && git commit --amend --no-edit   # thêm file quên
git commit --amend --reset-author --no-edit        # cập nhật author và thời gian
```

### `git cherry-pick`: sao chép một commit

`git cherry-pick C` lấy **thay đổi mà C đưa vào** (so với cha của nó) rồi áp lên HEAD, tạo commit mới C' với cùng message và author.

```
         A ◄── B ◄── C ◄── D     ◄── develop
               ▲
               └── R              ◄── release (HEAD)

git cherry-pick C

               └── R ◄── C'       ◄── release
```

Bên trong, đó là một three-way merge với:

- **base** = `C^` (cha của C)
- **ours** = HEAD
- **theirs** = C

Vì vậy nó có thể conflict, và xử lý y như merge (bài 08), rồi `git cherry-pick --continue`.

> 💡 So sánh với `revert C` (bài 09): cũng three-way merge nhưng đảo base và theirs (base = C, theirs = C^). Cherry-pick "áp lại" thay đổi, revert "áp ngược".

Hay dùng:

```bash
git cherry-pick A..C          # các commit trong A..C (không gồm A)
git cherry-pick -x C          # thêm dòng "(cherry picked from commit ...)" vào message
git cherry-pick -n C          # áp thay đổi vào index/working tree, không commit
git cherry-pick -m 1 <merge>  # cherry-pick một merge commit, so với cha 1
```

### `git rebase`: cherry-pick hàng loạt rồi di chuyển nhánh

`git rebase main` khi đang ở `feature`:

```
Trước:
        A ◄── B ◄── C ◄── F          ◄── main
                    ▲
                    └── D ◄── E      ◄── feature (HEAD)

Bước 1: Tìm các commit cần chuyển = main..feature = D, E
Bước 2: Lưu ORIG_HEAD = E, detach HEAD tại F (đầu của main)
Bước 3: cherry-pick D → D', rồi cherry-pick E → E'
Bước 4: Di chuyển feature sang E', gắn HEAD lại vào feature

Sau:
        A ◄── B ◄── C ◄── F          ◄── main
                    │     ▲
                    │     └── D' ◄── E'   ◄── feature (HEAD)
                    └── D ◄── E           (không còn nhánh nào trỏ tới, reflog và ORIG_HEAD vẫn nhớ)
```

Những điều rút ra:

- `D'` và `E'` là **commit mới**, hash khác, committer date mới. Author và message giữ nguyên.
- Mỗi commit được áp **lần lượt**, nên có thể phải giải conflict **nhiều lần** (mỗi commit một lần), khác với merge chỉ giải một lần.
- Commit nào mà thay đổi của nó **đã có sẵn** trong upstream (so theo *patch-id*) sẽ tự động bị bỏ qua. Nhờ đó rebase sau khi một commit đã được cherry-pick lên `main` không tạo ra trùng lặp.
- Merge commit trong đoạn được rebase sẽ bị **làm phẳng** (bỏ đi), trừ khi dùng `--rebase-merges`.

### ⚠️ ours/theirs bị đảo khi rebase

Trong lúc rebase, HEAD đang đứng trên **nhánh đích** (`main` và các commit đã áp xong), còn commit đang được áp là **commit của bạn**. Nên:

| | Merge (`git merge feature` khi ở main) | Rebase (`git rebase main` khi ở feature) |
|---|---|---|
| **ours** / HEAD / stage 2 | `main` | `main` + các commit đã áp xong |
| **theirs** / stage 3 | `feature` | **commit của bạn** đang được áp |

```
<<<<<<< HEAD
F                       ← phía main (ours khi rebase)
=======
D                       ← commit của bạn (theirs khi rebase)
>>>>>>> a53aeeb (D)
```

`git checkout --theirs file` khi rebase nghĩa là **lấy bản của bạn**. Ngược với trực giác, nên hãy nhớ kỹ.

### Điều khiển rebase

| Lệnh | Khi nào |
|------|---------|
| `git rebase --continue` | Đã giải conflict và `git add` |
| `git rebase --skip` | Bỏ qua commit đang áp |
| `git rebase --abort` | Huỷ toàn bộ, quay về ORIG_HEAD |
| `git reset --hard ORIG_HEAD` | Rebase xong mới thấy sai: quay về như trước rebase |

### Interactive rebase

`git rebase -i <base>` mở editor với danh sách commit trong `<base>..HEAD` (**cũ nhất ở trên**):

```
pick a53aeeb D: thêm form
pick 5efab4e E: sửa typo form
pick 7c1d2e3 G: thêm API
pick 9f8e7d6 H: wip
```

Bạn sửa danh sách, lưu và đóng editor. Git thực hiện từ trên xuống:

| Lệnh | Tác dụng |
|------|----------|
| `pick` | Giữ nguyên commit |
| `reword` | Giữ thay đổi, sửa message |
| `edit` | Dừng lại sau khi áp commit này để bạn sửa (amend, tách commit...) |
| `squash` | Gộp vào commit phía trên, **gộp cả message** |
| `fixup` | Gộp vào commit phía trên, **bỏ message** của commit này |
| `drop` (hoặc xoá dòng) | Bỏ commit |
| `exec <lệnh>` | Chạy lệnh shell (ví dụ chạy test sau mỗi commit) |
| `break` | Dừng tại đó |
| Đổi thứ tự dòng | Đổi thứ tự commit |

Ví dụ:

```
pick a53aeeb D: thêm form
fixup 5efab4e E: sửa typo form     ← gộp vào D, bỏ message
pick 7c1d2e3 G: thêm API
drop 9f8e7d6 H: wip
```

#### Tách một commit thành nhiều commit

Đánh dấu `edit`. Khi Git dừng lại:

```bash
git reset HEAD~          # bỏ commit, giữ thay đổi ở working tree
git add -p && git commit -m "phần 1"
git add -p && git commit -m "phần 2"
git rebase --continue
```

#### Quy trình fixup tự động

```bash
git commit --fixup=a53aeeb        # tạo commit "fixup! D: thêm form"
git rebase -i --autosquash main   # Git tự xếp fixup ngay dưới commit đích
git config --global rebase.autoSquash true   # luôn bật autosquash
```

### `rebase --onto`: cấy một đoạn commit sang chỗ khác

Dạng đầy đủ:

```
git rebase --onto <đích> <loại trừ> <nhánh>
```

Nghĩa là: lấy các commit trong `<loại trừ>..<nhánh>` rồi đặt lên trên `<đích>`.

Tình huống kinh điển: bạn tạo `sub` từ `feature` (chưa merge), nhưng lẽ ra phải tạo từ `main`:

```
        A ◄── B ◄── F                       ◄── main
              ▲
              └── D ◄── E ◄── G             ◄── feature
                              ▲
                              └── S1        ◄── sub

git rebase --onto main feature sub
→ lấy feature..sub = S1, đặt lên main

        A ◄── B ◄── F ◄── S1'               ◄── sub
```

Các dạng khác:

```bash
git rebase --onto HEAD~5 HEAD~3 HEAD   # xoá 2 commit HEAD~4, HEAD~3 khỏi giữa lịch sử
git rebase --update-refs main          # Git ≥ 2.38: cập nhật luôn các nhánh "xếp chồng" bên trên
```

### So sánh trước/sau khi rebase

```bash
git range-diff ORIG_HEAD~2..ORIG_HEAD HEAD~2..HEAD
git range-diff main@{1}...feature      # dạng ngắn, dùng reflog
```

`range-diff` so từng cặp commit cũ và mới. Rất hữu ích để review lại một nhánh sau khi rebase.

## 📏 Quy tắc vàng

> **Không viết lại commit đã được người khác dựa vào.**

Nếu commit đã push lên nhánh mà người khác pull về (`main`, `develop`, nhánh chung), việc rebase/amend rồi force push sẽ khiến lịch sử của họ lệch với server. Họ sẽ phải xử lý thủ công, dễ nhân đôi commit hoặc làm mất việc của nhau.

| Tình huống | Viết lại được không? |
|------------|:---:|
| Commit local chưa push | ✅ Thoải mái |
| Nhánh cá nhân đã push (chỉ bạn dùng) | ✅ Được, push bằng `--force-with-lease` |
| Nhánh PR đang được review | ⚠️ Tuỳ quy ước nhóm. Nhiều nhóm cho phép, và reviewer dùng `range-diff` |
| Nhánh dùng chung (`main`, `develop`, `release`) | ❌ Không. Dùng `revert` |

## 🧪 Thực hành

```bash
cd ~/git-lab && rm -rf rb && git init rb && cd rb
c() { echo "$1" > "$1.txt"; git add .; git commit -qm "$1"; }
c A; c B
git switch -qc feat; c D; c E
git switch -q main; c F
git log --oneline --graph --all

# 1. Rebase và quan sát hash mới
git rev-parse feat                       # ghi lại hash cũ của E
git switch -q feat && git rebase main
git log --oneline --graph --all          # thẳng hàng, D và E có hash mới
git reflog -4                            # rebase (pick): D, rebase (pick): E...
git log --oneline -1 ORIG_HEAD           # E cũ vẫn còn

# 2. Autosquash
echo sua >> D.txt && git commit -qam "fixup! D"
c G
git rebase -i --autosquash main          # xem editor: fixup đã nằm ngay dưới D. Lưu và đóng
git log --oneline                        # không còn "fixup! D"

# 3. Interactive: reword, drop, đổi thứ tự
git rebase -i main                       # thử đổi pick → reword cho G, drop E

# 4. --onto
git switch -qc sub; c S1
git switch -q feat; c H
git rebase --onto main feat~1 sub        # feat~1 là G, điểm sub tách ra
git log --oneline --graph --all

# 5. Conflict khi rebase: quan sát ours/theirs
git switch -q main && echo main > shared.txt && git add . && git commit -qm "main shared"
git switch -q feat && echo feat > shared.txt && git add . && git commit -qm "feat shared"
git rebase main
cat shared.txt                           # HEAD = main, phần dưới = commit của bạn
git checkout --theirs shared.txt && cat shared.txt   # "feat": bản của BẠN
git add shared.txt && git rebase --continue

# 6. Cherry-pick
git switch -q main
git cherry-pick -x sub
git log -1                               # có dòng "(cherry picked from commit ...)"
```

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "Rebase di chuyển commit" | Rebase **sao chép** commit thành commit mới. Commit cũ vẫn nằm đó. |
| "Amend sửa commit cuối" | Amend thay commit cuối bằng commit mới có hash khác. |
| "Rebase an toàn hơn merge" hoặc ngược lại | Cả hai đều an toàn với commit local. Rebase chỉ nguy hiểm khi viết lại commit người khác đã dùng. |
| "Cherry-pick sao chép cả commit" | Nó sao chép **thay đổi** (diff so với cha), rồi tạo commit mới trên snapshot của nhánh hiện tại. |
| "`--theirs` luôn là nhánh kia" | Khi rebase, `--theirs` là **commit của bạn**. |
| "Rebase xong không quay lại được" | `git reset --hard ORIG_HEAD` hoặc dùng reflog. |

## ✅ Tự kiểm tra

<details>
<summary>1. Nhánh <code>feat</code> có 5 commit. Bạn sửa message commit thứ 2 (tính từ cũ nhất) bằng interactive rebase. Bao nhiêu commit có hash mới?</summary>

4 commit: commit thứ 2 và 3 commit phía sau nó, vì hash của mỗi commit phụ thuộc vào hash của cha.
</details>

<details>
<summary>2. Vì sao rebase có thể bắt bạn giải cùng một conflict nhiều lần, còn merge thì không?</summary>

Rebase áp từng commit một. Nếu nhiều commit cùng chạm vào vùng conflict, mỗi lần áp đều có thể conflict. Merge chỉ so ba snapshot cuối cùng một lần. (`rerere` giúp giảm việc lặp lại.)
</details>

<details>
<summary>3. Một commit đã được cherry-pick lên <code>main</code>. Sau đó bạn rebase nhánh chứa commit gốc lên <code>main</code>. Commit đó có bị nhân đôi không?</summary>

Thường là không. Rebase bỏ qua commit có patch-id trùng với một commit đã có trong upstream. (Nếu lúc cherry-pick có phải giải conflict thì patch có thể khác, khi đó có thể conflict hoặc tạo commit rỗng.)
</details>

<details>
<summary>4. Viết lệnh chuyển 3 commit cuối của nhánh hiện tại (đang nằm trên <code>develop</code>) sang nằm trên <code>release</code>.</summary>

`git rebase --onto release HEAD~3` (nhánh mặc định là nhánh hiện tại).
</details>

<details>
<summary>5. Đồng đội vừa force push nhánh chung mà bạn đã pull về. Bạn làm sao để đồng bộ nếu không có commit riêng?</summary>

`git fetch` rồi `git reset --hard origin/<nhánh>`. Nếu có commit riêng: `git rebase --onto origin/<nhánh> <đầu nhánh cũ trước khi bị force push> <nhánh>` (tìm đầu nhánh cũ trong reflog của `origin/<nhánh>`), hoặc đơn giản là `git pull --rebase`, vì Git thường tự dùng reflog đó để tìm đúng điểm tách (`--fork-point`).
</details>

## 🏋️ Bài tập

1. Tạo 6 commit lộn xộn ("wip", "fix typo", "thêm test", "fix test"...). Dùng một lần `rebase -i` để biến chúng thành 2 commit có ý nghĩa.
2. Dùng `edit` để tách một commit chứa hai thay đổi không liên quan thành hai commit.
3. Dùng `exec` trong interactive rebase để chạy một lệnh kiểm tra (ví dụ `test -f A.txt`) sau **mỗi** commit. Cố tình làm một commit fail và quan sát.
4. Tạo ba nhánh xếp chồng `a` ← `b` ← `c`. Sửa một commit trong `a` bằng `git rebase -i --update-refs` khi đang đứng ở `c`. Kiểm tra `a` và `b` đều được cập nhật.

## 📎 Tra nhanh

- [Cheat sheet 05: Sửa commit & Squash](../cheat-sheet/05-amend-va-squash.md)
- [Cheat sheet 11: Rebase, Cherry-pick, Patch](../cheat-sheet/11-rebase-cherry-pick-patch.md)
- Pro Git: [3.6 Rebasing](https://git-scm.com/book/en/v2/Git-Branching-Rebasing), [7.6 Rewriting History](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History)

---

[← Trước: Remote, fetch, push](10-remote-fetch-push.md) · [Lộ trình](../README.md) · [Tiếp: Quy trình làm việc nhóm →](12-quy-trinh-nhom.md)
