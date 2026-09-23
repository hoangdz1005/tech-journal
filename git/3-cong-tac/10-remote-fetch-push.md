# 10. Remote, fetch, pull, push

[← Lộ trình](../README.md) · Giai đoạn 3: Cộng tác phân tán · ⏱️ ~60 phút

## 🎯 Mục tiêu

- Hiểu remote và **remote-tracking branch** thực chất là gì.
- Mô tả chính xác `clone`, `fetch`, `pull`, `push` làm gì với object và ref.
- Đọc hiểu **refspec** và cấu hình **upstream**.
- Hiểu vì sao push bị từ chối, và dùng force push một cách an toàn.

> Yêu cầu: [bài 03](../1-nen-tang/03-refs-branch-head.md) (ref), [bài 05](../1-nen-tang/05-do-thi-commit.md) (reachable), [bài 08](../2-thao-tac-hang-ngay/08-merge-va-conflict.md) (merge).

## ❓ Vấn đề

Bài 01 nói mỗi bản clone là một repo đầy đủ và độc lập. Vậy làm sao các repo này trao đổi commit? Và "nhánh `main` của tôi", "`origin/main`", "nhánh `main` trên GitHub" là ba thứ khác nhau ra sao?

## 🧠 Khái niệm

### Remote chỉ là một tên viết tắt cho URL

```bash
git remote -v
# origin  git@github.com:team/app.git (fetch)
# origin  git@github.com:team/app.git (push)
```

Trong `.git/config`:

```ini
[remote "origin"]
    url = git@github.com:team/app.git
    fetch = +refs/heads/*:refs/remotes/origin/*
```

`origin` không có gì đặc biệt, chỉ là tên mặc định mà `clone` đặt. Một repo có thể có nhiều remote (ví dụ `origin` là fork của bạn, `upstream` là repo gốc).

### Ba "main" khác nhau

```
   Máy bạn                                           Server (origin)
┌────────────────────────────────────┐            ┌──────────────────┐
│ refs/heads/main          (main)    │            │ refs/heads/main  │
│   └ nhánh của bạn, bạn commit lên  │            │   └ nhánh thật   │
│                                    │   fetch    │     trên server  │
│ refs/remotes/origin/main           │ ◄───────── │                  │
│   (origin/main)                    │            │                  │
│   └ bản ghi nhớ: "lần fetch cuối,  │            │                  │
│     main trên server ở đây"        │            │                  │
└────────────────────────────────────┘            └──────────────────┘
```

| Tên | Là gì | Ai cập nhật |
|-----|-------|-------------|
| `main` | Nhánh local của bạn | Bạn (commit, merge, reset...) |
| `origin/main` | **Remote-tracking branch**: ảnh chụp nhánh `main` của server ở lần giao tiếp gần nhất | **Chỉ** `fetch`/`pull`/`push` |
| `main` trên server | Nhánh thật trên server | Những ai push lên |

Điểm mấu chốt: **`origin/main` có thể đã cũ.** Nó không tự cập nhật. `git status` báo *"Your branch is up to date with 'origin/main'"* chỉ có nghĩa là bạn khớp với **lần fetch gần nhất**, không phải với server lúc này.

Bạn không thể commit lên `origin/main`: `git switch origin/main` sẽ đưa bạn vào detached HEAD.

### Upstream (tracking)

Nhánh local có thể được cấu hình để "theo dõi" một remote-tracking branch:

```ini
[branch "main"]
    remote = origin
    merge = refs/heads/main
```

Nhờ đó:

- `git pull` / `git push` không cần tham số.
- `git status` tính được **ahead/behind** (bằng `git rev-list --count` hai chiều, như bài 05).
- Dùng được `@{u}` / `@{upstream}`.

```bash
git branch -vv                       # xem upstream của mọi nhánh
git branch -u origin/feature         # đặt upstream cho nhánh hiện tại
git push -u origin feature           # push và đặt upstream luôn
```

## 🔬 Bên trong Git: từng lệnh

### `git clone <url>`

Tương đương:

```bash
git init <thư-mục>
git remote add origin <url>
git fetch origin                     # tải mọi object, tạo refs/remotes/origin/*
git switch <nhánh mặc định của remote>   # tạo nhánh local có upstream là origin/<nhánh>
```

Chỉ **một** nhánh local được tạo. Các nhánh khác chỉ tồn tại dưới dạng `origin/*` cho tới khi bạn `switch` sang (Git tự tạo nhánh local theo dõi nó).

### `git fetch`

1. Hỏi server: "các ref của bạn đang trỏ tới đâu?"
2. Tính ra các object mà server có nhưng bạn chưa có (dựa trên các commit hai bên cùng có).
3. Tải về các object đó (dạng packfile, bài 15).
4. Cập nhật remote-tracking branch theo **refspec**.
5. Ghi `.git/FETCH_HEAD`.

**`fetch` không bao giờ động tới nhánh local, index hay working tree.** Nó luôn an toàn, chạy lúc nào cũng được.

### Refspec

Refspec cho biết ánh xạ **ref nguồn → ref đích**:

```
+refs/heads/*:refs/remotes/origin/*
│└──── nguồn ───┘ └──────── đích ──────┘
└─ "+" = cho phép cập nhật kể cả khi không phải fast-forward
```

Đọc là: "mọi nhánh `refs/heads/X` trên server → lưu thành `refs/remotes/origin/X` ở local, được phép ghi đè". Dấu `+` cần thiết vì người khác có thể force push lên server, và remote-tracking branch phải phản ánh đúng thực tế.

Refspec cũng dùng cho push:

```bash
git push origin main                 # = main:main
git push origin feat:review/feat     # push nhánh local feat thành nhánh review/feat trên server
git push origin :old                 # nguồn rỗng = XOÁ nhánh old trên server
git push origin --delete old         # cách viết dễ đọc hơn
```

### `git pull` = fetch + tích hợp

```bash
git pull                             # = git fetch + git merge @{u}
git pull --rebase                    # = git fetch + git rebase @{u}
git pull --ff-only                   # = git fetch + git merge --ff-only @{u}
```

Khi nhánh local và server **đều có commit mới** (diverged):

- `merge` tạo một merge commit "Merge branch 'main' of ...". Lịch sử có thêm nhánh rẽ không cần thiết.
- `rebase` đặt commit local lên trên commit mới của server. Lịch sử thẳng (bài 11).
- `--ff-only` từ chối, để bạn tự quyết.

Git ≥ 2.27 sẽ cảnh báo nếu bạn chưa chọn. Nên cấu hình rõ:

```bash
git config --global pull.rebase true     # hoặc
git config --global pull.ff only
```

> 💡 Nhiều người thích tách `pull` thành `fetch` rồi xem trước `git log HEAD..@{u}` trước khi tích hợp. Cách này giúp bạn biết mình sắp nhận về những gì.

### `git push`

1. Hỏi server các ref hiện tại.
2. Gửi các object server còn thiếu.
3. Yêu cầu server cập nhật ref: "đặt `refs/heads/main` thành `cd793ba`".
4. Server **chỉ chấp nhận nếu là fast-forward**: commit cũ của `main` trên server phải reachable từ commit mới.
5. Thành công thì cập nhật `origin/main` ở local.

```
Server:  A ◄── B ◄── X                   (Bình đã push X)
Bạn:     A ◄── B ◄── Y                   (bạn có Y, chưa có X)

git push → ! [rejected] main -> main (fetch first)
```

Push bị từ chối vì nếu chấp nhận, `X` sẽ không còn reachable từ `main` trên server. Commit của Bình sẽ bị "mất". Cách xử lý: `git pull --rebase` (hoặc merge), rồi push lại.

Có hai thông báo từ chối cần phân biệt:

| Thông báo | Nghĩa |
|-----------|-------|
| `(fetch first)` | Server có commit mà local **chưa hề biết** |
| `(non-fast-forward)` | Local đã biết (có trong `origin/main`) nhưng nhánh của bạn không chứa nó, ví dụ sau khi bạn rebase/reset |

### Force push và `--force-with-lease`

`git push --force` bảo server "cứ ghi đè, không cần fast-forward". Cần khi bạn **đã viết lại lịch sử** (rebase, amend) trên nhánh đã push. Nhưng nó có thể xoá mất commit người khác vừa push.

`--force-with-lease` an toàn hơn: "chỉ ghi đè nếu nhánh trên server **vẫn đang ở đúng chỗ mà `origin/main` của tôi ghi nhận**". Nếu ai đó đã push thêm, lệnh bị từ chối.

```bash
git push --force-with-lease
git push --force-with-lease --force-if-includes   # Git ≥ 2.30, an toàn hơn nữa
```

⚠️ Bẫy: nếu IDE hoặc một công cụ nào đó tự động `fetch` ngầm, `origin/main` đã được cập nhật và "lease" luôn khớp. Khi đó `--force-with-lease` cũng ghi đè như `--force`. `--force-if-includes` khắc phục bằng cách kiểm tra thêm: commit trên server phải đã từng nằm trong reflog của nhánh local của bạn.

**Nguyên tắc:** không bao giờ force push lên nhánh dùng chung (`main`, `develop`). Nhánh cá nhân của bạn thì được, kèm `--force-with-lease`.

### Dọn remote-tracking branch

Nhánh bị xoá trên server thì `origin/<nhánh>` ở local **không tự mất**:

```bash
git fetch --prune                    # xoá các origin/* không còn trên server
git config --global fetch.prune true # luôn prune khi fetch
```

## 🧪 Thực hành: mô phỏng server trên máy

Không cần GitHub. Một **bare repo** (repo không có working tree) đóng vai server:

```bash
cd ~/git-lab && rm -rf remote-lab && mkdir remote-lab && cd remote-lab
git init --bare -b main server.git     # "server"

# An tạo commit đầu tiên và push
git clone server.git an && cd an       # cảnh báo "cloned an empty repository" là bình thường
echo 1 > f && git add f && git commit -qm "An 1"
git branch -M main                     # đảm bảo nhánh tên main dù máy bạn mặc định là master
git push -u origin main
cat .git/config                         # xem [remote "origin"] và [branch "main"]
git branch -vv                          # * main [origin/main] An 1
cd ..

# Bình clone
git clone server.git binh && cd binh
git branch -a                           # main, remotes/origin/main, remotes/origin/HEAD
echo 2 > g && git add g && git commit -qm "Binh 2"
git push
cd ..

# An commit tiếp, CHƯA fetch
cd an
echo 3 > h && git add h && git commit -qm "An 3"
git status | head -2                    # "up to date with 'origin/main'", nhưng đã cũ!
git push                                # ! [rejected] (fetch first)

# fetch: chỉ cập nhật origin/main
git fetch
git status | head -3                    # "have diverged, 1 and 1 different commits"
git log --oneline --graph --all
git log --oneline HEAD..origin/main     # những gì sắp nhận: Binh 2
git log --oneline origin/main..HEAD     # những gì sắp gửi: An 3

# tích hợp rồi push
git pull --rebase
git log --oneline --graph --all         # thẳng hàng
git push
git ls-remote origin                    # xem ref trên server

# Nhánh và xoá nhánh trên server
git switch -c feat && git commit --allow-empty -qm "feat" && git push -u origin feat
cd ../binh && git fetch && git branch -r   # Bình thấy origin/feat
cd ../an && git push origin --delete feat  # An xoá feat trên server
cd ../binh && git fetch && git branch -r   # origin/feat VẪN còn ở máy Bình
git fetch --prune && git branch -r         # giờ mới mất
```

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "`origin/main` là nhánh trên server" | Đó là bản sao ref **local**, chỉ đúng tới lần fetch gần nhất. |
| "`git status` báo up to date nghĩa là không có gì mới trên server" | Nó so với `origin/main` local. Phải `fetch` trước mới biết. |
| "`fetch` sẽ sửa code của tôi" | `fetch` không động tới nhánh local, index hay working tree. |
| "`pull` chỉ là tải về" | `pull` = fetch **+ merge/rebase**, có thể tạo merge commit hoặc gây conflict. |
| "`--force-with-lease` luôn an toàn" | Không, nếu có fetch ngầm cập nhật `origin/*`. Dùng kèm `--force-if-includes`. |
| "Xoá nhánh trên server thì `origin/<nhánh>` tự mất" | Cần `fetch --prune`. |

## ✅ Tự kiểm tra

<details>
<summary>1. <code>git fetch</code> thay đổi những gì trong <code>.git/</code>?</summary>

Thêm object mới (thường là packfile) vào `objects/`, cập nhật `refs/remotes/origin/*`, ghi `FETCH_HEAD`, thêm dòng reflog cho các remote-tracking branch. Không động tới `refs/heads/*`, index hay working tree.
</details>

<details>
<summary>2. Vì sao server từ chối push không phải fast-forward?</summary>

Nếu chấp nhận, commit cũ của nhánh trên server sẽ không còn reachable từ nhánh đó, tức là làm "mất" công việc của người khác.
</details>

<details>
<summary>3. Sau khi <code>git commit --amend</code> một commit đã push, vì sao <code>git push</code> bị từ chối?</summary>

Amend tạo commit mới **thay thế** commit cũ (cùng cha). Commit cũ trên server không phải tổ tiên của commit mới, nên đây không phải fast-forward. Cần `--force-with-lease`, và chỉ nên làm trên nhánh cá nhân.
</details>

<details>
<summary>4. Viết refspec để fetch nhánh <code>main</code> của remote <code>upstream</code> vào ref local <code>refs/remotes/upstream/main</code>.</summary>

`+refs/heads/main:refs/remotes/upstream/main`
</details>

<details>
<summary>5. Bạn muốn xem đồng đội đã push gì trước khi tích hợp. Chuỗi lệnh nào?</summary>

`git fetch` → `git log --oneline HEAD..@{u}` (hoặc `git diff HEAD...@{u}`) → rồi mới `git merge`/`git rebase`.
</details>

## 🏋️ Bài tập

1. Thêm remote thứ hai tên `backup` (một bare repo khác), push mọi nhánh và tag lên đó (`git push backup --all` và `--tags`).
2. Mô phỏng quy trình fork: `upstream` (repo gốc), `origin` (fork của bạn). Đồng bộ nhánh `main` của fork với `upstream` mà không tạo merge commit.
3. Mô phỏng bẫy `--force-with-lease`: Bình push commit mới; An chạy `git fetch` (giả làm IDE), rồi `git push --force-with-lease`. Commit của Bình có mất không? Thử lại với `--force-if-includes`.
4. Dùng `git fetch origin main:refs/remotes/origin/main` rồi `git fetch origin main` và so sánh ref nào được cập nhật.

## 📎 Tra nhanh

- [Cheat sheet 04: Remote, Fetch/Pull, Force Push](../cheat-sheet/04-remote.md)
- Pro Git: [3.5 Remote Branches](https://git-scm.com/book/en/v2/Git-Branching-Remote-Branches), [10.5 The Refspec](https://git-scm.com/book/en/v2/Git-Internals-The-Refspec)

---

[← Trước: Hoàn tác](../2-thao-tac-hang-ngay/09-hoan-tac.md) · [Lộ trình](../README.md) · [Tiếp: Viết lại lịch sử →](11-viet-lai-lich-su.md)
