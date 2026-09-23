# 15. Lưu trữ: packfile, gc, reachability

[← Lộ trình](../README.md) · Giai đoạn 4: Nâng cao · ⏱️ ~45 phút

## 🎯 Mục tiêu

- Hiểu loose object và packfile, và vì sao "lưu snapshot" lại không tốn chỗ.
- Biết `git gc` làm gì, và khi nào dữ liệu **thực sự** bị xoá.
- Biết cách xoá hẳn một file nhạy cảm khỏi lịch sử (và vì sao việc này khó).
- Làm việc với repo lớn: shallow clone, partial clone, sparse-checkout.

> Yêu cầu: [bài 02](../1-nen-tang/02-object-model.md) (object), [bài 05](../1-nen-tang/05-do-thi-commit.md) (reachable), [bài 09](../2-thao-tac-hang-ngay/09-hoan-tac.md) (reflog).

## ❓ Vấn đề

Bài 01 nói Git lưu snapshot, còn bài 02 cho thấy mỗi phiên bản của file là một blob nén **toàn bộ** nội dung. Vậy một file log 100KB sửa 1000 lần sẽ tốn 100MB? Và nếu commit nhầm mật khẩu, `reset` rồi thì nó đã biến mất khỏi ổ đĩa chưa?

## 🧠 Loose object và packfile

### Loose object

Mỗi object mới tạo (`add`, `commit`) được lưu thành **một file riêng**, nén zlib, tại `.git/objects/xx/yyyy...`. Đơn giản, ghi nhanh, nhưng:

- Mỗi phiên bản của file là một bản đầy đủ (đã nén).
- Hàng chục nghìn file nhỏ làm hệ thống file chậm.

### Packfile

Định kỳ (hoặc khi push/fetch), Git gom object vào **packfile**:

```
.git/objects/pack/
├── pack-be6f5d...ff.pack    ← dữ liệu: các object, nhiều cái lưu dạng delta
├── pack-be6f5d...ff.idx     ← chỉ mục: hash → vị trí trong .pack (tra cứu nhanh)
└── pack-be6f5d...ff.rev     ← (Git mới) chỉ mục ngược
```

Trong packfile, Git dùng **nén delta**: một object có thể được lưu dưới dạng "lấy object X, áp các thay đổi sau".

```bash
git verify-pack -v .git/objects/pack/*.idx | grep blob
# hash                                      loại kích thước  kích thước  vị trí  độ sâu  base
# 3d47bf3c7cfdc961537deb6b47c3e95bfa143941 blob   108900   43775       235
# 7599e0c9615053f4425667d889c445b2634f1cf9 blob   11       23          44102   1       3d47bf3c...
```

Ở ví dụ trên, file 108KB có hai phiên bản. Phiên bản **mới hơn** (`3d47`) được lưu đầy đủ, phiên bản **cũ** (`7599`) chỉ là một delta 11 byte dựa trên bản mới.

Điểm quan trọng:

- Delta ở đây là **chi tiết lưu trữ**, hoàn toàn tách biệt với mô hình dữ liệu. Về mặt logic, commit vẫn trỏ tới snapshot đầy đủ.
- Git chọn cặp base/delta bằng **heuristic** (cùng tên file, kích thước gần nhau...), không nhất thiết theo thứ tự thời gian. Git thường giữ **bản mới** đầy đủ vì bản mới được truy cập nhiều hơn.
- Delta có thể dựa trên object **bất kỳ**, kể cả file khác.
- Khi fetch/push, Git gửi dữ liệu dạng packfile, chỉ gồm các object bên kia còn thiếu.

File nhị phân đã nén (ảnh, video, zip) gần như không delta được. Mỗi phiên bản vẫn tốn trọn dung lượng. Đó là lý do có **Git LFS** (bài 16).

```bash
git count-objects -vH        # số loose object, số pack, dung lượng
```

## 🧠 `git gc`: dọn dẹp và đóng gói

`git gc` (thường chạy tự động qua `gc --auto` sau một số lệnh, hoặc qua `git maintenance`) làm các việc:

1. **Pack refs**: gom ref vào `packed-refs`.
2. **Hết hạn reflog**: xoá dòng reflog cũ hơn `gc.reflogExpire` (mặc định **90 ngày**) và dòng trỏ tới commit unreachable cũ hơn `gc.reflogExpireUnreachable` (mặc định **30 ngày**).
3. **Repack**: gom loose object vào packfile, tối ưu delta.
4. **Prune**: xoá loose object **unreachable** và cũ hơn `gc.pruneExpire` (mặc định **2 tuần**).

### 🔬 Khi nào một commit thực sự bị xoá?

Một object chỉ bị xoá khi **không reachable** từ bất kỳ đâu:

- Mọi ref: `refs/heads/*`, `refs/tags/*`, `refs/remotes/*`, `refs/stash`...
- **Reflog** (cho tới khi dòng reflog hết hạn).
- Index (blob đang được stage).
- HEAD của các worktree.

```
commit bị "reset" mất
   │
   ├─ reflog còn giữ ──── 30 ngày (unreachable) ────┐
   │                                                ▼
   │                                     dòng reflog hết hạn
   │                                                │
   └─ loose object ───────── 2 tuần ────────────────┤
                                                    ▼
                                           git gc prune → mất hẳn
```

Vì vậy, trong thực tế, commit "lỡ tay xoá" gần như luôn cứu được trong vài tuần.

### Xoá hẳn một file nhạy cảm khỏi lịch sử

Commit nhầm `.env` chứa mật khẩu và **đã push**? Thứ tự đúng:

1. **Đổi (rotate) mật khẩu/khoá ngay lập tức.** Coi như đã lộ. Đây là bước quan trọng nhất, vì dù xoá khỏi lịch sử thì bản clone của người khác, fork, cache của CI, cache của nền tảng vẫn có thể còn.
2. Viết lại lịch sử để loại file (dùng [`git filter-repo`](https://github.com/newren/git-filter-repo), công cụ được khuyến nghị thay cho `filter-branch`):
   ```bash
   git filter-repo --invert-paths --path .env
   ```
3. Force push mọi nhánh và tag, yêu cầu mọi người clone lại.
4. Liên hệ nền tảng (GitHub/GitLab) nếu cần xoá cache hoặc tham chiếu PR cũ.

Nếu chỉ mới commit **local** (chưa push): sửa commit (amend/rebase), rồi dọn local:

```bash
git reflog expire --expire=now --all
git gc --prune=now
```

## 🧠 Repo lớn

| Kỹ thuật | Lệnh | Tác dụng |
|----------|------|----------|
| **Shallow clone** | `git clone --depth 1 <url>` | Chỉ lấy N commit gần nhất. Nhanh cho CI. Một số thao tác lịch sử sẽ hạn chế |
| **Partial clone** | `git clone --filter=blob:none <url>` | Lấy đủ commit và tree, blob chỉ tải **khi cần** (lúc checkout). Lịch sử đầy đủ, dung lượng nhỏ |
| **Sparse checkout** | `git sparse-checkout set apps/web libs/ui` | Chỉ đưa một số thư mục ra working tree. Hợp với monorepo |
| **Bảo trì nền** | `git maintenance start` | Lên lịch prefetch, commit-graph, repack tăng dần |
| **commit-graph** | `git commit-graph write` | File chỉ mục giúp `log --graph`, `merge-base` nhanh hơn nhiều |

## 🧠 Kiểm tra toàn vẹn

```bash
git fsck                         # kiểm tra mọi object: hash khớp nội dung, liên kết đầy đủ
git fsck --unreachable           # liệt kê object unreachable
git fsck --lost-found            # đưa object treo vào .git/lost-found/
```

Nhờ content-addressing (bài 02), bất kỳ thay đổi nào trên đĩa đều bị phát hiện, vì hash tính lại sẽ không khớp tên.

## 🧪 Thực hành

```bash
cd ~/git-lab && rm -rf pk && git init pk && cd pk
seq 1 20000 > big.txt && git add . && git commit -qm v1
echo extra >> big.txt && git commit -qam v2

# 1. Loose object: hai bản đầy đủ
git count-objects -vH           # count: 6, size ~96 KiB

# 2. Pack lại
git gc
git count-objects -vH           # in-pack: 6, size-pack ~44 KiB
ls .git/objects/pack/
git verify-pack -v .git/objects/pack/*.idx | grep blob
# bản mới lưu đầy đủ, bản cũ là delta vài byte

# 3. Commit unreachable sống sót qua gc nhờ reflog
git commit -q --allow-empty -m "sẽ bị bỏ"
git reset -q --hard HEAD~1
git fsck --unreachable --no-reflogs     # thấy commit "sẽ bị bỏ" (nếu bỏ qua reflog)
git gc --prune=now
git log -1 --oneline HEAD@{1}           # vẫn còn! vì reflog giữ nó

# 4. Xoá hẳn
git reflog expire --expire=now --all
git gc --prune=now
git fsck --unreachable --no-reflogs     # không còn gì
```

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "Git lưu mỗi phiên bản đầy đủ nên repo phình rất nhanh" | Packfile dùng nén delta. Với file văn bản, lịch sử thường nhỏ hơn nhiều so với tưởng tượng. |
| "Git lưu delta giữa các commit" | Delta chỉ là chi tiết của packfile, chọn theo heuristic. Mô hình logic vẫn là snapshot. |
| "`reset`/`rebase` xoá commit khỏi ổ đĩa" | Commit còn đó cho tới khi reflog hết hạn **và** gc prune. |
| "Xoá khỏi lịch sử là hết lộ mật khẩu" | Bản clone, fork, cache vẫn có thể còn. Luôn rotate secret trước. |
| "Repo lớn thì phải clone đầy đủ" | Partial clone và sparse-checkout giúp làm việc với repo rất lớn. |

## ✅ Tự kiểm tra

<details>
<summary>1. Bạn <code>reset --hard</code> bỏ 3 commit hôm qua. Chạy <code>git gc</code> ngay. Commit đó còn cứu được không?</summary>

Còn. Reflog vẫn tham chiếu chúng (30 ngày với commit unreachable), nên gc không xoá.
</details>

<details>
<summary>2. Vì sao commit một file ZIP 50MB và sửa nó 10 lần làm repo nặng gần 500MB?</summary>

Dữ liệu đã nén gần như không delta được. Mỗi phiên bản là một blob lớn. Nên dùng Git LFS hoặc không commit file build.
</details>

<details>
<summary>3. <code>--depth 1</code> và <code>--filter=blob:none</code> khác nhau thế nào?</summary>

Shallow (`--depth`) cắt **lịch sử commit**. Partial clone (`--filter=blob:none`) giữ đủ lịch sử commit/tree nhưng tải **nội dung file** khi cần.
</details>

## 🏋️ Bài tập

1. Clone một repo lớn (ví dụ `git/git`) theo ba cách: đầy đủ, `--depth 1`, `--filter=blob:none`. So sánh thời gian và dung lượng `.git`. Thử `git log -p` và `git blame` ở mỗi bản.
2. Commit một "secret" giả, đẩy lên một bare repo local, rồi dùng `git filter-repo` xoá nó. Kiểm tra bằng `git log --all -S` ở cả local và bare repo.
3. Tìm 10 blob lớn nhất trong lịch sử một repo:
   ```bash
   git rev-list --objects --all | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | awk '$1=="blob"' | sort -k3 -n -r | head
   ```

## 📎 Tra nhanh

- [Cheat sheet 14: Toàn vẹn dữ liệu, dọn dẹp](../cheat-sheet/14-bao-tri-va-don-dep.md)
- Pro Git: [10.4 Packfiles](https://git-scm.com/book/en/v2/Git-Internals-Packfiles), [10.7 Maintenance and Data Recovery](https://git-scm.com/book/en/v2/Git-Internals-Maintenance-and-Data-Recovery)

---

[← Trước: Điều tra lịch sử](14-dieu-tra-lich-su.md) · [Lộ trình](../README.md) · [Tiếp: Cấu hình và hooks →](16-cau-hinh-va-hooks.md)
