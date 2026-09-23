# 08. Merge và xử lý conflict

[← Lộ trình](../README.md) · Giai đoạn 2: Thao tác hằng ngày · ⏱️ ~60 phút

## 🎯 Mục tiêu

- Phân biệt **fast-forward** và **three-way merge**, biết khi nào xảy ra loại nào.
- Hiểu vai trò của **merge base** trong việc tự động gộp thay đổi.
- Biết conflict thực sự được lưu ở đâu (index stage 1/2/3) và vì sao.
- Xử lý conflict tự tin: đọc marker, dùng `ours`/`theirs`, huỷ merge.

> Yêu cầu: [bài 04](../1-nen-tang/04-index-ba-cay.md) (index), [bài 05](../1-nen-tang/05-do-thi-commit.md) (merge base).

## ❓ Vấn đề

Hai nhánh cùng phát triển từ một điểm. Làm sao gộp thay đổi của cả hai mà **không** cần người xem từng dòng? Và khi hai bên sửa cùng một chỗ, làm sao Git biết đó là xung đột chứ không phải một bên đã cố ý đổi?

## 🧠 Khái niệm

### Trường hợp 1: fast-forward

Nếu nhánh hiện tại **là tổ tiên** của nhánh cần merge (tức là nhánh hiện tại không có commit riêng nào), Git không cần gộp gì cả. Nó chỉ **di chuyển con trỏ** lên phía trước:

```
Trước:   A ◄── B            ◄── main (HEAD)
                ▲
                └── C ◄── D  ◄── feature

git merge feature

Sau:     A ◄── B ◄── C ◄── D  ◄── main (HEAD), feature
```

Không có commit mới nào được tạo. Lịch sử thẳng tắp, như thể mọi thứ được làm trực tiếp trên `main`.

| Tuỳ chọn | Hành vi |
|----------|---------|
| (mặc định) | Fast-forward nếu được, không thì three-way merge |
| `--ff-only` | Chỉ fast-forward. Không được thì báo lỗi |
| `--no-ff` | Luôn tạo merge commit, kể cả khi fast-forward được. Giữ dấu vết "đã có một nhánh ở đây" |

### Trường hợp 2: three-way merge

Khi cả hai nhánh đều có commit riêng, Git cần **ba** snapshot:

```
        A ◄── B ◄── C ◄── F          ◄── main (HEAD)  = "ours"
                    ▲
                    └── D ◄── E      ◄── feature      = "theirs"

        base = merge-base(main, feature) = C
```

Vì sao cần base? Hãy xem một dòng code:

| base (C) | ours (F) | theirs (E) | Kết luận |
|----------|----------|------------|----------|
| `x = 1` | `x = 1` | `x = 1` | Không ai đổi → `x = 1` |
| `x = 1` | `x = 2` | `x = 1` | Chỉ ours đổi → lấy `x = 2` |
| `x = 1` | `x = 1` | `x = 3` | Chỉ theirs đổi → lấy `x = 3` |
| `x = 1` | `x = 2` | `x = 2` | Cả hai đổi giống nhau → `x = 2` |
| `x = 1` | `x = 2` | `x = 3` | **Cả hai đổi khác nhau → CONFLICT** |

Nếu chỉ so ours với theirs (two-way), Git thấy `x = 2` khác `x = 3` nhưng **không biết** bên nào đã đổi. Base cho Git biết "trạng thái trước khi tách". Nhờ vậy phần lớn thay đổi được gộp tự động.

Kết quả là một **merge commit** có hai cha:

```
        A ◄── B ◄── C ◄── F ◄──── M   ◄── main (HEAD)
                    ▲             │
                    └── D ◄── E ◄─┘   ◄── feature
```

`M^1 = F` (nhánh bạn đứng), `M^2 = E` (nhánh được merge vào).

> 💡 Git so sánh **snapshot** chứ không phát lại từng commit. Merge chỉ nhìn C, F và E. Các commit D, B ở giữa không ảnh hưởng tới kết quả (trừ việc chúng nằm trong lịch sử).

### Conflict là gì, chính xác

Conflict xảy ra khi Git không tự quyết được:

| Loại | Ví dụ |
|------|-------|
| **Content** | Hai bên sửa cùng vùng dòng trong một file theo cách khác nhau |
| **Modify/delete** | Một bên sửa file, bên kia xoá file đó |
| **Rename/rename** | Hai bên đổi tên cùng file thành hai tên khác nhau |
| **Add/add** | Hai bên cùng tạo file mới trùng đường dẫn, nội dung khác |

Hai bên sửa **cùng file nhưng khác chỗ** (cách xa nhau) thì **không** conflict, Git gộp được.

## 🔬 Bên trong Git: conflict nằm trong index

Bình thường mỗi đường dẫn trong index có **một** mục với stage number `0`. Khi conflict, Git ghi **ba** mục cho cùng một đường dẫn:

```bash
git ls-files --stage
# 100644 a369a29... 1	app.js     ← stage 1: phiên bản base
# 100644 35c6d49... 2	app.js     ← stage 2: phiên bản ours (HEAD)
# 100644 4be0cff... 3	app.js     ← stage 3: phiên bản theirs (nhánh được merge)
```

Và working tree chứa phiên bản đã gộp **kèm marker** tại những chỗ conflict.

Toàn bộ trạng thái "đang merge" gồm:

| Thành phần | Nội dung |
|------------|----------|
| Index | Các mục stage 1/2/3 cho file conflict; stage 0 cho file đã gộp xong |
| Working tree | File có marker `<<<<<<<` `=======` `>>>>>>>` |
| `.git/MERGE_HEAD` | Hash commit đang merge vào (sẽ thành cha thứ hai) |
| `.git/MERGE_MSG` | Message mặc định cho merge commit |
| `.git/ORIG_HEAD` | HEAD trước khi merge |

**Không thể commit** khi index còn mục stage ≠ 0, vì `write-tree` không biết chọn phiên bản nào. Khi bạn `git add app.js`, Git **xoá ba mục stage 1/2/3** và ghi một mục stage 0 với nội dung hiện tại của file. Đó là ý nghĩa thật của câu "add để đánh dấu đã giải quyết".

Xem từng phiên bản:

```bash
git show :1:app.js      # base
git show :2:app.js      # ours
git show :3:app.js      # theirs
```

### Đọc marker

Mặc định:

```
<<<<<<< HEAD
  return "xin chao";          ← ours
=======
  return "hi";                ← theirs
>>>>>>> b
```

Nên bật kiểu `zdiff3` (Git ≥ 2.35) để thấy luôn **base**, giúp hiểu mỗi bên đã đổi gì:

```bash
git config --global merge.conflictStyle zdiff3
```

```
<<<<<<< ours
  return "xin chao";
||||||| base
  return "hello";             ← bản gốc: cả hai bên đều đã đổi "hello"
=======
  return "hi";
>>>>>>> theirs
```

### Công cụ xử lý conflict

| Lệnh | Tác dụng |
|------|----------|
| `git status` | Liệt kê file conflict (`UU`, `AA`, `DU`...) |
| `git diff` | Khi đang conflict: hiện **combined diff** (`diff --cc`) so với cả hai bên |
| `git checkout --ours <f>` / `--theirs <f>` | Lấy nguyên phiên bản stage 2 / stage 3 ra working tree |
| `git checkout --conflict=zdiff3 <f>` | Tạo lại marker (nếu lỡ sửa hỏng) |
| `git add <f>` | Đánh dấu đã giải quyết (gộp stage 1/2/3 thành stage 0) |
| `git merge --continue` | Tạo merge commit (tương đương `git commit`) |
| `git merge --abort` | Huỷ merge, về trạng thái trước khi merge |
| `git mergetool` | Mở công cụ merge đồ hoạ đã cấu hình |
| `git log --merge -p <f>` | Xem các commit ở hai bên đã chạm vào file conflict |

### "ours" và "theirs" khi merge

- **ours** = nhánh bạn đang đứng (HEAD).
- **theirs** = nhánh bạn chỉ định trong `git merge <nhánh>`.

⚠️ Khi **rebase**, hai khái niệm này bị **đảo ngược** (bài 11).

Còn một điều dễ nhầm: `-X ours` và `-s ours` khác nhau hoàn toàn.

| Lệnh | Ý nghĩa |
|------|---------|
| `git merge -X ours feature` | Merge bình thường. Chỉ **chỗ conflict** thì chọn bên ours. Thay đổi không conflict của feature vẫn được gộp |
| `git merge -s ours feature` | Tạo merge commit nhưng **bỏ toàn bộ** nội dung của feature. Kết quả y hệt HEAD |

### Chiến lược merge

Git ≥ 2.34 dùng chiến lược mặc định `ort` (trước đó là `recursive`). Khi có **nhiều** merge base (criss-cross merge), `ort`/`recursive` sẽ merge các base với nhau trước để tạo một base ảo. Bạn hiếm khi cần đổi chiến lược. `octopus` được dùng khi merge nhiều nhánh cùng lúc.

### `rerere`: nhớ cách giải conflict

Nếu phải giải cùng một conflict nhiều lần (rebase lặp lại, merge thử), bật:

```bash
git config --global rerere.enabled true
```

Git ghi lại cặp "conflict → cách giải" và tự áp dụng lần sau.

## 🧪 Thực hành

```bash
cd ~/git-lab && rm -rf mg && git init mg && cd mg
printf 'function greet() {\n  return "hello";\n}\n' > app.js
git add . && git commit -qm base

# 1. Fast-forward
git switch -qc ff && echo "// ff" >> app.js && git commit -qam "ff"
git switch -q main && git merge ff          # "Fast-forward"
git log --oneline --graph                   # thẳng, không có merge commit

# 2. Tạo conflict
git switch -qc a && sed -i.bak 's/hello/xin chao/' app.js && rm app.js.bak && git commit -qam "a: xin chao"
git switch -q main
git switch -qc b && sed -i.bak 's/hello/hi/' app.js && rm app.js.bak && git commit -qam "b: hi"
git switch -q a
git merge b                                 # CONFLICT (content)

# 3. Quan sát trạng thái merge
git status --short                          # UU app.js
git ls-files --stage                        # 3 mục: stage 1, 2, 3
git show :1:app.js                          # base: hello
git show :2:app.js                          # ours: xin chao
git show :3:app.js                          # theirs: hi
cat .git/MERGE_HEAD                         # hash commit đầu nhánh b
cat app.js                                  # có marker

# 4. Xem kèm base
git checkout --conflict=zdiff3 app.js && cat app.js

# 5. Giải quyết: viết lại nội dung, rồi add
printf 'function greet() {\n  return "xin chao / hi";\n}\n// ff\n' > app.js
git add app.js
git ls-files --stage                        # chỉ còn 1 mục stage 0
git commit -qm "Merge b vào a"
git cat-file -p HEAD | head -3              # 2 dòng parent

# 6. Thử lại rồi huỷ
git switch -q main && git merge a --no-commit --no-ff
git merge --abort
git status --short                          # sạch
```

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "Merge phát lại từng commit của nhánh kia" | Merge chỉ so ba snapshot: base, ours, theirs. |
| "Sửa cùng file là conflict" | Chỉ conflict khi sửa cùng **vùng** theo cách khác nhau. |
| "`git add` khi conflict là stage bình thường" | `add` thay 3 mục stage 1/2/3 bằng 1 mục stage 0. Đó là cách Git biết conflict đã được giải. |
| "Không có conflict nghĩa là merge đúng" | Git chỉ gộp **văn bản**. Hai thay đổi có thể gộp sạch nhưng phá logic của nhau (ví dụ một bên đổi tên hàm, bên kia thêm lời gọi tên cũ). Luôn build và chạy test sau merge. |
| "`-X ours` = bỏ thay đổi của nhánh kia" | `-X ours` chỉ ưu tiên ours ở chỗ conflict. Bỏ hoàn toàn là `-s ours`. |

## ✅ Tự kiểm tra

<details>
<summary>1. <code>main</code> có 0 commit riêng kể từ khi <code>feature</code> tách ra. <code>git merge feature</code> tạo mấy commit mới?</summary>

0 commit mới, đây là fast-forward. Dùng `--no-ff` nếu muốn có merge commit.
</details>

<details>
<summary>2. Vì sao merge cần merge base, không chỉ hai đầu nhánh?</summary>

Base cho biết trạng thái trước khi tách. Nhờ đó Git biết bên nào đã đổi một dòng. Chỉ một bên đổi thì lấy bên đó, cả hai đổi khác nhau mới là conflict.
</details>

<details>
<summary>3. Đang merge có conflict, <code>git ls-files --stage</code> hiện 2 mục stage 2 và stage 3 cho <code>x.txt</code> nhưng không có stage 1. Điều đó nghĩa là gì?</summary>

Base không có file `x.txt`. Cả hai nhánh cùng thêm mới file này với nội dung khác nhau (add/add conflict).
</details>

<details>
<summary>4. Lỡ sửa hỏng file conflict và mất marker. Làm sao lấy lại?</summary>

`git checkout --conflict=merge <file>` (hoặc `=zdiff3`). Được vì index vẫn còn giữ ba phiên bản stage 1/2/3.
</details>

<details>
<summary>5. Merge xong, test fail. Muốn quay về trước khi merge (chưa push). Làm gì?</summary>

`git reset --hard ORIG_HEAD` (hoặc `HEAD~1` / `HEAD^1`). Xem bài 09.
</details>

## 🏋️ Bài tập

1. Tạo một conflict **modify/delete**. Quan sát `git status` và `git ls-files --stage`. Giải quyết theo hai cách: giữ file, và xoá file (`git rm`).
2. Hai nhánh sửa hai hàm khác nhau trong cùng một file. Xác nhận merge không conflict.
3. Tạo một "conflict ngữ nghĩa": nhánh A đổi tên hàm `foo` thành `bar`, nhánh B thêm một lời gọi `foo()` ở file khác. Merge sạch sẽ, nhưng code hỏng. Rút ra bài học gì về CI?
4. Bật `rerere`, tạo conflict, giải quyết, `reset --hard` về trước merge, merge lại. Quan sát thông báo `Resolved ... using previous resolution`.

## 📎 Tra nhanh

- [Cheat sheet 03: Merge & xử lý xung đột](../cheat-sheet/03-merge-va-xung-dot.md)
- Pro Git: [3.2 Basic Merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging), [7.8 Advanced Merging](https://git-scm.com/book/en/v2/Git-Tools-Advanced-Merging)

---

[← Trước: Branch, switch, checkout](07-branch-switch-checkout.md) · [Lộ trình](../README.md) · [Tiếp: Hoàn tác →](09-hoan-tac.md)
