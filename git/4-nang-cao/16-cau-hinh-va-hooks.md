# 16. Cấu hình, ignore, attributes, hooks

[← Lộ trình](../README.md) · Giai đoạn 4: Nâng cao · ⏱️ ~50 phút

## 🎯 Mục tiêu

- Hiểu ba tầng cấu hình và thứ tự ưu tiên.
- Viết `.gitignore` chính xác, và biết vì sao file đã tracked không bị ignore.
- Dùng `.gitattributes` cho line ending, file nhị phân, diff/merge tuỳ biến, LFS.
- Viết và chia sẻ hook để tự động hoá kiểm tra.

## ❓ Vấn đề

Máy Windows commit CRLF, máy Mac commit LF, và mọi file đều "thay đổi". `node_modules` lọt vào repo. Ai đó commit code không qua lint. Đây đều là những vấn đề mà cấu hình đúng giải quyết một lần là xong.

## 🧠 Cấu hình (`git config`)

### Ba tầng

| Tầng | File | Cờ | Phạm vi |
|------|------|----|---------|
| System | `/etc/gitconfig` | `--system` | Mọi người dùng trên máy |
| Global | `~/.gitconfig` hoặc `~/.config/git/config` | `--global` | Mọi repo của bạn |
| Local | `.git/config` | `--local` (mặc định) | Repo hiện tại |
| Worktree | `.git/config.worktree` | `--worktree` | Một worktree (cần bật `extensions.worktreeConfig`) |

Tầng **gần hơn thắng**: local > global > system. Tham số `-c key=value` trên dòng lệnh thắng tất cả.

```bash
git config --list --show-origin          # xem mọi giá trị và nó đến từ file nào
git config --show-origin user.email      # giá trị hiện hành của một key
git config --global --edit               # mở file global để sửa
```

### Cấu hình theo thư mục: `includeIf`

Tách email công ty và email cá nhân:

```ini
# ~/.gitconfig
[user]
    name = Nguyễn An
    email = an@personal.dev
[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work
```

```ini
# ~/.gitconfig-work
[user]
    email = an@company.com
```

### Cấu hình nên có

```bash
git config --global init.defaultBranch main
git config --global pull.rebase true           # hoặc pull.ff only
git config --global fetch.prune true
git config --global rebase.autoSquash true
git config --global rebase.updateRefs true
git config --global merge.conflictStyle zdiff3
git config --global rerere.enabled true
git config --global diff.algorithm histogram
git config --global push.autoSetupRemote true  # push lần đầu tự đặt upstream
git config --global core.editor "code --wait"
```

### Alias

```bash
git config --global alias.st "status -sb"
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.unstage "restore --staged"
git config --global alias.amend "commit --amend --no-edit"
git config --global alias.wip '!git add -A && git commit -m WIP'   # "!" = chạy lệnh shell
```

## 🧠 `.gitignore`

### Quy tắc

`.gitignore` chỉ quyết định file **untracked** có hiện ra trong `git status` và có bị `git add .` thêm vào hay không. **File đã có trong index (tracked) không bao giờ bị ignore.**

| Mẫu | Khớp |
|-----|------|
| `*.log` | Mọi file `.log` ở mọi cấp |
| `/build` | Chỉ `build` ở **cùng cấp** với file `.gitignore` |
| `build/` | Chỉ **thư mục** tên `build` (ở mọi cấp) |
| `docs/*.md` | `.md` ngay trong `docs/` (không đệ quy) |
| `docs/**/*.md` | `.md` ở mọi cấp trong `docs/` |
| `!keep.log` | Phủ định: **không** ignore `keep.log` |
| `# ...` | Chú thích |

⚠️ Không thể "bỏ ignore" một file nằm trong thư mục đã bị ignore:

```gitignore
logs/            # ignore cả thư mục
!logs/keep.txt   # KHÔNG có tác dụng, Git không duyệt vào thư mục đã bị ignore

# Đúng:
logs/*
!logs/keep.txt
```

### Các nơi đặt quy tắc ignore

| Nơi | Commit vào repo? | Dùng cho |
|-----|:---:|----------|
| `.gitignore` (mọi thư mục) | ✅ | Quy tắc chung của dự án: `node_modules/`, `dist/` |
| `.git/info/exclude` | ❌ | Quy tắc riêng bạn, chỉ repo này |
| `core.excludesFile` (mặc định `~/.config/git/ignore`) | ❌ | Quy tắc riêng bạn, mọi repo: `.DS_Store`, `.idea/` |

### Gỡ lỗi

```bash
git check-ignore -v path/to/file         # file bị ignore bởi dòng nào, ở file nào
git status --ignored                     # liệt kê file đang bị ignore
git rm --cached -r node_modules          # ngừng track thư mục đã lỡ commit
```

## 🧠 `.gitattributes`

`.gitattributes` gán **thuộc tính** cho đường dẫn, điều khiển cách Git chuyển đổi, so sánh và merge file.

### Line ending

```gitattributes
* text=auto              # Git tự nhận biết file văn bản, lưu LF trong repo
*.sh text eol=lf         # luôn LF khi checkout (script shell)
*.bat text eol=crlf      # luôn CRLF khi checkout
*.png binary             # nhị phân: không chuyển đổi, không diff dạng văn bản
```

🔬 Bên trong: đây là cơ chế **filter** giữa working tree và object database. Khi `add`, Git chuyển CRLF → LF trước khi hash (nội dung lưu trong blob là LF). Khi checkout, Git chuyển LF → CRLF nếu cần. Đặt quy tắc trong `.gitattributes` (commit vào repo) tốt hơn cấu hình `core.autocrlf` trên từng máy.

Sau khi thêm quy tắc cho repo đã có sẵn file:

```bash
git add --renormalize .
git commit -m "Chuẩn hoá line ending"
```

### Diff và merge tuỳ biến

```gitattributes
*.lock      -diff                 # không hiện diff (file sinh tự động)
package-lock.json merge=ours      # khi conflict, giữ bản của mình (cần cấu hình driver "ours")
*.md        diff=markdown         # nhận diện tiêu đề làm ngữ cảnh hunk
*.docx      diff=word             # dùng textconv để diff (cần cấu hình driver)
```

```bash
git config diff.word.textconv docx2txt
git config merge.ours.driver true
```

### Export và thống kê

```gitattributes
tests/      export-ignore          # không đưa vào git archive
docs/**     linguist-documentation # GitHub không tính vào thống kê ngôn ngữ
```

### Git LFS

Với file nhị phân lớn (ảnh gốc, video, model), **Git LFS** thay nội dung file trong repo bằng một **file con trỏ** nhỏ và lưu nội dung thật ở server riêng. Nó hoạt động qua cơ chế filter của `.gitattributes`:

```bash
git lfs install
git lfs track "*.psd"            # thêm dòng: *.psd filter=lfs diff=lfs merge=lfs -text
git add .gitattributes
```

## 🧠 Hooks

Hook là script chạy tự động tại các thời điểm trong vòng đời Git. Chúng nằm ở `.git/hooks/` (xem file `*.sample` để có ví dụ), phải có quyền thực thi, và mã thoát khác 0 sẽ **chặn** thao tác (với các hook "pre-").

### Hook phía client hay dùng

| Hook | Chạy khi | Dùng để |
|------|----------|---------|
| `pre-commit` | Trước khi tạo commit | Lint, format, chặn commit secret |
| `prepare-commit-msg` | Trước khi mở editor | Chèn mã ticket từ tên nhánh |
| `commit-msg` | Sau khi viết message | Kiểm tra định dạng message (Conventional Commits) |
| `post-commit` | Sau commit | Thông báo |
| `pre-push` | Trước khi push | Chạy test |
| `post-checkout`, `post-merge` | Sau switch/merge | Cài lại dependency nếu lockfile đổi |

Hook phía server (`pre-receive`, `update`, `post-receive`) chạy trên server và dùng để áp chính sách. Trên GitHub/GitLab, vai trò này do branch protection và CI đảm nhận.

### Ví dụ: `commit-msg`

```bash
#!/bin/sh
# .githooks/commit-msg
if ! head -1 "$1" | grep -qE '^(feat|fix|docs|refactor|test|chore)(\(.+\))?: .+'; then
  echo "❌ Message phải theo Conventional Commits, ví dụ: feat(auth): thêm đăng nhập" >&2
  exit 1
fi
```

### Chia sẻ hook với nhóm

`.git/hooks/` **không** được commit hay clone. Cách chia sẻ:

```bash
mkdir .githooks && mv commit-msg .githooks/ && chmod +x .githooks/commit-msg
git config core.hooksPath .githooks     # mỗi người chạy một lần (hoặc qua script setup)
```

Hoặc dùng công cụ như `pre-commit` (Python), `husky`/`lefthook` (Node). Chúng tự cấu hình `core.hooksPath` khi cài dependency.

⚠️ Hook phía client có thể bị bỏ qua (`git commit --no-verify`). Kiểm tra bắt buộc phải nằm ở **CI**. Hook chỉ giúp phát hiện sớm.

## 🧪 Thực hành

```bash
cd ~/git-lab && rm -rf cfg && git init cfg && cd cfg

# 1. Tầng cấu hình
git config user.name "Tên riêng repo"
git config --show-origin user.name
git -c user.name="Dòng lệnh" config user.name

# 2. .gitignore
mkdir -p logs && touch logs/a.log logs/keep.txt app.log
printf 'logs/\n!logs/keep.txt\n*.log\n' > .gitignore
git status --short --untracked-files=all     # keep.txt KHÔNG hiện
git check-ignore -v logs/keep.txt
printf 'logs/*\n!logs/keep.txt\n*.log\n' > .gitignore
git status --short --untracked-files=all     # giờ thì keep.txt hiện

# 3. File tracked không bị ignore
git add -A && git commit -qm init
echo "debug" > debug.log && git add -f debug.log && git commit -qm "lỡ commit log"
echo more >> debug.log && git status --short     #  M debug.log dù *.log bị ignore
git rm --cached -q debug.log && git commit -qm "ngừng track debug.log"

# 4. Hook commit-msg
mkdir .githooks
cat > .githooks/commit-msg <<'EOF'
#!/bin/sh
head -1 "$1" | grep -qE '^(feat|fix|docs|chore)(\(.+\))?: .+' || { echo "❌ Sai định dạng message" >&2; exit 1; }
EOF
chmod +x .githooks/commit-msg
git config core.hooksPath .githooks
git commit --allow-empty -m "sửa linh tinh"              # bị chặn
git commit --allow-empty -m "chore: thử hook"            # thành công
git commit --allow-empty -m "bỏ qua" --no-verify         # hook bị bỏ qua
```

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "Thêm vào `.gitignore` là Git ngừng theo dõi file" | Chỉ tác dụng với file untracked. File đã tracked cần `git rm --cached`. |
| "`!pattern` luôn bỏ ignore được" | Không được nếu thư mục cha đã bị ignore. |
| "Hook được clone cùng repo" | `.git/hooks` không được chia sẻ. Dùng `core.hooksPath` hoặc công cụ quản lý hook. |
| "Hook đảm bảo chất lượng code" | Hook client bị bỏ qua được bằng `--no-verify`. CI mới là nơi bắt buộc. |
| "`core.autocrlf` là cách tốt nhất xử lý line ending" | `.gitattributes` commit vào repo sẽ nhất quán cho mọi người. |

## ✅ Tự kiểm tra

<details>
<summary>1. <code>user.email</code> được đặt ở cả global và local. Commit dùng giá trị nào?</summary>

Local. Tầng gần repo hơn được ưu tiên.
</details>

<details>
<summary>2. Viết <code>.gitignore</code> để ignore mọi thứ trong <code>dist/</code> trừ <code>dist/.gitkeep</code>.</summary>

```
dist/*
!dist/.gitkeep
```
</details>

<details>
<summary>3. Vì sao với <code>* text=auto</code>, blob trong repo luôn là LF dù bạn dùng Windows?</summary>

Filter chuyển CRLF → LF khi `add` (trước khi hash thành blob), và chuyển ngược khi checkout nếu cần.
</details>

## 🏋️ Bài tập

1. Thiết lập `includeIf` để các repo trong `~/work/` dùng email công ty. Kiểm tra bằng `git config --show-origin user.email` trong một repo ở đó.
2. Viết hook `pre-commit` chặn commit nếu có file chứa chuỗi `PRIVATE KEY`.
3. Viết hook `prepare-commit-msg` tự chèn mã ticket (ví dụ `ABC-123`) từ tên nhánh `feat/ABC-123-mo-ta` vào đầu message.
4. Tạo repo có file CRLF, thêm `.gitattributes` với `* text=auto`, chạy `git add --renormalize .` và quan sát diff.

## 📎 Tra nhanh

- [Cheat sheet 12: Cấu hình, bảo mật, alias, attributes](../cheat-sheet/12-cau-hinh.md)
- [Cheat sheet 13: Thao tác với index](../cheat-sheet/13-tham-chieu-va-index.md)
- Pro Git: [8.1 Git Configuration](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration), [8.2 Git Attributes](https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes), [8.3 Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)

---

[← Trước: Lưu trữ và gc](15-luu-tru-va-gc.md) · [Lộ trình](../README.md) · [Tiếp: Tag, submodule, subtree →](17-tag-submodule-subtree.md)
