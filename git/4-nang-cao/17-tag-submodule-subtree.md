# 17. Tag, submodule, subtree

[← Lộ trình](../README.md) · Giai đoạn 4: Nâng cao · ⏱️ ~50 phút

## 🎯 Mục tiêu

- Phân biệt lightweight tag, annotated tag, signed tag, và biết cách phát hành phiên bản.
- Hiểu submodule thực chất là một **con trỏ tới commit của repo khác** nằm trong tree.
- So sánh submodule và subtree để chọn đúng cách quản lý code dùng chung.

> Yêu cầu: [bài 02](../1-nen-tang/02-object-model.md) (tag object, tree mode), [bài 03](../1-nen-tang/03-refs-branch-head.md) (ref).

## ❓ Vấn đề

Làm sao đánh dấu "đây là bản 2.3.0 đã phát hành" một cách bền vững? Và khi dự án dùng một thư viện nội bộ nằm ở repo khác, làm sao nhúng nó vào mà vẫn cố định được phiên bản?

## 🧠 Tag

### Ba loại tag

| Loại | Lệnh | Bên trong | Dùng khi |
|------|------|-----------|----------|
| **Lightweight** | `git tag v1.0` | Chỉ là ref `refs/tags/v1.0` trỏ thẳng tới commit | Đánh dấu tạm, cá nhân |
| **Annotated** | `git tag -a v1.0 -m "..."` | Ref trỏ tới **tag object** (người tạo, ngày, message), tag object trỏ tới commit | **Phát hành** |
| **Signed** | `git tag -s v1.0 -m "..."` | Annotated + chữ ký GPG/SSH | Phát hành cần xác thực |

```bash
git cat-file -t v1.0        # commit  → lightweight
git cat-file -t v1.1        # tag     → annotated
git show v1.1               # hiện thông tin tag + commit
git tag -v v1.2             # xác minh chữ ký
```

Nhiều công cụ (`git describe`, trang Release của GitHub) mặc định chỉ xét **annotated tag**. Hãy dùng annotated cho mọi bản phát hành.

### Tag không tự được push

```bash
git push origin v1.1              # push một tag
git push origin --tags            # push mọi tag (kể cả tag rác local)
git push --follow-tags            # push commit + các annotated tag trỏ tới commit đó
git config --global push.followTags true
```

### Không di chuyển tag đã công bố

Tag được thiết kế để **cố định**. Nếu đã push `v1.0` rồi xoá và tạo lại trỏ chỗ khác, những ai đã fetch sẽ **không** tự cập nhật (Git không ghi đè tag đã có khi fetch). Hệ quả: hai người có hai `v1.0` khác nhau. Nếu phát hành sai, hãy tạo `v1.0.1`.

```bash
git tag -d v1.0                       # xoá local
git push origin --delete v1.0         # xoá trên remote
```

### `git describe`

```bash
git describe                  # v1.1-1-gf160608
#                               │    │  └ "g" + hash rút gọn của HEAD
#                               │    └ số commit kể từ tag
#                               └ annotated tag gần nhất
git describe --tags           # xét cả lightweight tag
```

Rất tiện để nhúng phiên bản vào bản build.

### Semantic Versioning

`MAJOR.MINOR.PATCH` (ví dụ `2.3.1`): tăng MAJOR khi thay đổi phá tương thích, MINOR khi thêm tính năng tương thích, PATCH khi sửa lỗi. Kết hợp với Conventional Commits (bài 12), công cụ có thể tự tính phiên bản kế tiếp.

## 🧠 Submodule

### 🔬 Bên trong: gitlink

Submodule **không** chép code của repo con vào repo cha. Tree của repo cha chỉ lưu một mục đặc biệt, mode `160000` (gọi là **gitlink**), trỏ tới **hash commit** của repo con:

```bash
git ls-tree HEAD
# 100644 blob 9b6e268...   .gitmodules
# 100644 blob b80f0bd...   a
# 040000 tree 6d57e92...   vendor
git ls-files -s vendor/lib
# 160000 211dd7a4eaab... 0	vendor/lib      ← hash là commit trong repo lib, không phải blob
```

Kèm theo là file `.gitmodules` (commit vào repo) cho biết lấy repo con ở đâu:

```ini
[submodule "vendor/lib"]
    path = vendor/lib
    url = https://github.com/team/lib.git
```

Repo con là một **repo Git đầy đủ** nằm trong thư mục con (dữ liệu `.git` của nó được đặt trong `.git/modules/vendor/lib` của repo cha).

Hệ quả của thiết kế này:

- Repo cha cố định **chính xác một commit** của repo con. Đây là ưu điểm lớn nhất: build tái lập được.
- Cập nhật repo con = commit mới **trong repo con** + commit trong repo cha để **đổi gitlink** sang hash mới.
- Sau `git submodule update`, repo con ở trạng thái **detached HEAD** (vì repo cha trỏ tới một commit, không trỏ tới nhánh).

### Quy trình

```bash
# Thêm
git submodule add https://github.com/team/lib.git vendor/lib
git commit -m "Thêm submodule lib"

# Clone repo có submodule
git clone --recurse-submodules <url>
# hoặc nếu đã clone:
git submodule update --init --recursive

# Kéo code mới của repo cha (gitlink có thể đã đổi)
git pull
git submodule update --recursive           # checkout đúng commit mà repo cha trỏ tới
git config --global submodule.recurse true # để pull/switch tự làm việc này

# Nâng cấp repo con lên bản mới
cd vendor/lib && git fetch && git switch --detach origin/main && cd -
# hoặc: git submodule update --remote vendor/lib
git add vendor/lib && git commit -m "Nâng lib lên bản mới"

# Xem trạng thái
git submodule status                        # "+" ở đầu = repo con đang ở commit khác gitlink
git diff --submodule
```

### Bẫy thường gặp

| Bẫy | Hậu quả | Cách tránh |
|-----|---------|-----------|
| Quên `submodule update` sau `pull` | Repo con vẫn ở commit cũ, `git status` báo `modified: vendor/lib (new commits)` | `submodule.recurse true` |
| Commit trong repo con lúc detached HEAD rồi update | Commit bị "bỏ rơi" (cứu được bằng reflog của repo con) | `git switch -c` trong repo con trước khi sửa |
| Push repo cha trước khi push repo con | Người khác không fetch được commit mà gitlink trỏ tới | `git push --recurse-submodules=check` (hoặc `on-demand`) |
| Vô tình `git add .` khi repo con đang lệch | Commit gitlink sai phiên bản | Đọc kỹ `git diff --submodule` trước khi commit |

## 🧠 Subtree

Subtree là cách khác: **chép thật** nội dung (và tuỳ chọn cả lịch sử) của repo khác vào một thư mục con, thông qua merge.

```bash
git subtree add  --prefix=vendor/lib https://github.com/team/lib.git main --squash
git subtree pull --prefix=vendor/lib https://github.com/team/lib.git main --squash
git subtree push --prefix=vendor/lib https://github.com/team/lib.git feature-x
```

🔬 Bên trong: `subtree add` tạo một merge commit mà cha thứ hai là lịch sử của repo con (hoặc một commit squash), với toàn bộ file được dời vào `vendor/lib/`. Với người dùng repo cha, đó chỉ là **file bình thường**: clone về là có ngay, không cần lệnh đặc biệt.

`git subtree` là script contrib, có thể chưa được cài sẵn trong mọi bản phân phối Git.

## ⚖️ Submodule hay subtree?

| Tiêu chí | Submodule | Subtree |
|----------|-----------|---------|
| Cách lưu | Con trỏ tới commit (gitlink) | Chép nội dung vào repo cha |
| Clone | Cần `--recurse-submodules` | Bình thường |
| Người dùng cần biết? | Có, phải học lệnh submodule | Không |
| Cố định phiên bản | Chính xác, rõ ràng | Theo lần `pull` gần nhất |
| Sửa code dùng chung rồi đẩy ngược lại | Tự nhiên (repo con là repo đầy đủ) | Được (`subtree push`), nhưng rườm rà hơn |
| Kích thước repo cha | Nhỏ | Lớn hơn (chứa code con) |
| Phù hợp | Thư viện lớn, phát triển độc lập, cần cố định phiên bản | Code nhỏ, ít thay đổi, muốn người dùng không phải bận tâm |

Trước khi dùng cả hai, hãy cân nhắc: nếu thư viện có thể phát hành qua **package manager** (npm, Maven, pip, Go modules...), đó thường là lựa chọn đơn giản nhất. Nếu các dự án liên quan chặt chẽ, **monorepo** cũng là một phương án.

## 🧪 Thực hành

```bash
cd ~/git-lab && rm -rf sm && mkdir sm && cd sm
git init -q lib && (cd lib && echo v1 > lib.txt && git add . && git commit -qm "lib v1")
git init -q app && cd app && echo app > a && git add . && git commit -qm app

# 1. Tag
git tag v1.0                           # lightweight
git tag -a v1.1 -m "Phát hành 1.1"     # annotated
git cat-file -t v1.0; git cat-file -t v1.1
git cat-file -p v1.1
git commit -q --allow-empty -m next
git describe                            # v1.1-1-g<hash>

# 2. Submodule (repo local cần cho phép giao thức file)
git -c protocol.file.allow=always submodule add ../lib vendor/lib
git commit -qm "thêm submodule"
cat .gitmodules
git ls-files -s vendor/lib              # mode 160000, hash là commit của lib

# 3. Cập nhật repo con
(cd ../lib && echo v2 > lib.txt && git commit -qam "lib v2")
(cd vendor/lib && git pull -q)
git status --short                      #  M vendor/lib
git diff --submodule                    # Submodule vendor/lib xxx..yyy: > lib v2
git add vendor/lib && git commit -qm "nâng lib lên v2"

# 4. Clone repo có submodule
cd .. && git -c protocol.file.allow=always clone -q --recurse-submodules app app2
(cd app2/vendor/lib && git status | head -1)   # HEAD detached at ...
```

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "Push code là push luôn tag" | Tag phải push riêng hoặc dùng `--follow-tags`. |
| "Sửa tag trên remote thì mọi người tự cập nhật" | Fetch không ghi đè tag đã có. Đừng di chuyển tag đã công bố. |
| "Submodule chứa code của repo con" | Repo cha chỉ chứa **hash commit** (gitlink) và `.gitmodules`. |
| "Submodule theo dõi một nhánh" | Nó cố định một commit. `--remote` chỉ là tiện ích để nhảy tới đầu nhánh. |
| "Detached HEAD trong submodule là lỗi" | Đó là trạng thái bình thường sau `submodule update`. |

## ✅ Tự kiểm tra

<details>
<summary>1. Vì sao nên dùng annotated tag cho bản phát hành?</summary>

Nó là object riêng chứa người tạo, ngày, message (và có thể ký). `git describe` và nhiều công cụ phát hành mặc định chỉ xét annotated tag.
</details>

<details>
<summary>2. Trong tree của repo cha, submodule được lưu dưới dạng gì?</summary>

Một mục mode `160000` (gitlink) chứa hash **commit** của repo con.
</details>

<details>
<summary>3. Đồng đội báo "fatal: remote error: upload-pack: not our ref" khi update submodule. Nguyên nhân khả dĩ?</summary>

Gitlink trong repo cha trỏ tới một commit của repo con **chưa được push** lên server của repo con.
</details>

## 🏋️ Bài tập

1. Tạo tag ký bằng SSH (`git config gpg.format ssh`, `user.signingkey`) và xác minh bằng `git tag -v`.
2. Mô phỏng bẫy "push repo cha trước repo con" và thử `git push --recurse-submodules=check`.
3. Nhúng cùng một thư viện bằng subtree (có `--squash`), cập nhật thư viện, `subtree pull`, rồi so sánh `git log --graph` với cách dùng submodule.

## 📎 Tra nhanh

- [Cheat sheet 07: Tag](../cheat-sheet/07-tag.md)
- [Cheat sheet 18: Subtree & Submodule](../cheat-sheet/18-subtree-va-submodule.md)
- Pro Git: [2.6 Tagging](https://git-scm.com/book/en/v2/Git-Basics-Tagging), [7.11 Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

---

[← Trước: Cấu hình và hooks](16-cau-hinh-va-hooks.md) · [Lộ trình](../README.md) · [Tiếp: Bài tập tổng hợp →](../bai-tap-tong-hop.md)
