# 12. Quy trình làm việc nhóm

[← Lộ trình](../README.md) · Giai đoạn 3: Cộng tác phân tán · ⏱️ ~45 phút

## 🎯 Mục tiêu

- So sánh ba cách tích hợp nhánh: merge commit, rebase + fast-forward, squash merge.
- Hiểu ưu nhược điểm của trunk-based, GitHub Flow và Git Flow.
- Viết commit và message có giá trị lâu dài.
- Có một quy trình cá nhân rõ ràng, từ lúc tạo nhánh đến khi merge PR.

> Yêu cầu: [bài 08](../2-thao-tac-hang-ngay/08-merge-va-conflict.md), [bài 10](10-remote-fetch-push.md), [bài 11](11-viet-lai-lich-su.md).

## ❓ Vấn đề

Git không áp đặt quy trình nào. Cùng một công cụ, có nhóm có lịch sử thẳng tắp dễ đọc, có nhóm lại có một "mạng nhện" merge commit mà không ai dám đào vào. Khác biệt nằm ở **quy ước**, và bài này giúp bạn chọn quy ước có chủ đích.

## 🧠 Ba cách đưa một nhánh vào `main`

Giả sử có:

```
        A ◄── B ◄── F              ◄── main
              ▲
              └── D ◄── E          ◄── feature
```

### 1. Merge commit (`--no-ff`)

```
        A ◄── B ◄── F ◄────── M    ◄── main
              ▲               │
              └── D ◄── E ◄───┘
```

- ✅ Giữ nguyên lịch sử thật: ai làm gì, lúc nào, trên nhánh nào.
- ✅ Revert cả tính năng bằng một lệnh: `git revert -m 1 M`.
- ✅ `git log --first-parent main` cho thấy "mỗi PR một dòng".
- ❌ Đồ thị rối nếu nhiều nhánh song song, và các commit "wip", "fix" của nhánh cũng vào lịch sử chính.

### 2. Rebase rồi fast-forward

```
        A ◄── B ◄── F ◄── D' ◄── E'   ◄── main
```

- ✅ Lịch sử thẳng, `git log` và `git bisect` dễ dùng.
- ✅ Mỗi commit được giữ riêng biệt, nên cần **mỗi commit đều sạch** (build được, có ý nghĩa).
- ❌ Mất ranh giới "các commit này thuộc cùng một tính năng".
- ❌ Commit được tạo lại. Hash trên `main` khác hash trên nhánh của bạn.

### 3. Squash merge

```
        A ◄── B ◄── F ◄── S          ◄── main       (S = D + E gộp lại)
```

- ✅ Mỗi PR = một commit, lịch sử chính rất gọn.
- ✅ Không cần dọn dẹp commit trên nhánh.
- ❌ Mất chi tiết từng bước. PR lớn thành một commit khổng lồ, khó bisect.
- ❌ Git **không biết** nhánh đã được merge (`S` không có quan hệ cha con với `E`). `git branch -d` sẽ báo *not fully merged*. Tiếp tục làm trên nhánh cũ rồi squash lần nữa dễ gây conflict.

### Chọn cách nào?

| Nếu nhóm bạn... | Nên chọn |
|-----------------|----------|
| PR nhỏ, không muốn bắt mọi người dọn commit | **Squash merge** |
| Coi trọng từng commit, mọi người quen `rebase -i` | **Rebase + fast-forward** |
| Cần dấu vết tính năng rõ ràng, hay phải revert cả tính năng | **Merge commit** |

Không có lựa chọn đúng tuyệt đối. Điều quan trọng là **cả nhóm dùng cùng một cách**, và cấu hình nó trên GitHub/GitLab (Settings → Merge button).

## 🧠 Mô hình phân nhánh

### Trunk-based development

```
main ──●──●──●──●──●──●──●──●──
        \─●─/  \─●●─/  \─●─/        nhánh sống vài giờ đến 1–2 ngày
```

- Mọi người tích hợp vào `main` **thường xuyên** (ít nhất mỗi ngày).
- Nhánh rất ngắn, PR nhỏ. Tính năng chưa xong được ẩn bằng **feature flag**.
- `main` luôn ở trạng thái deploy được, nhờ CI mạnh.
- Phù hợp: sản phẩm web/SaaS, deploy liên tục. Đây là xu hướng chủ đạo hiện nay.

### GitHub Flow

Một biến thể đơn giản của trunk-based:

1. `main` luôn deploy được.
2. Tạo nhánh từ `main` cho mỗi việc.
3. Mở PR, review, CI xanh.
4. Merge vào `main` và deploy.

### Git Flow

```
main     ──●─────────────●───────────●──       (chỉ chứa bản phát hành, có tag)
            \           / \         /
release      \     ●──●    \   ●──●
              \   /    \    \ /    \
develop  ──●───●──●──●──●────●──●───●──        (tích hợp)
            \    /    \      /
feature      ●──●      ●──●─
hotfix                  (từ main, merge vào cả main và develop)
```

- Hai nhánh sống lâu: `main` (production) và `develop` (tích hợp).
- Nhánh phụ: `feature/*`, `release/*`, `hotfix/*`.
- Phù hợp: phần mềm **phát hành theo phiên bản**, cần duy trì nhiều phiên bản song song (app mobile, thư viện, phần mềm cài đặt).
- Nhược: nhiều nghi thức, nhánh sống lâu dễ conflict lớn. Chính tác giả Git Flow cũng khuyên dùng mô hình đơn giản hơn cho ứng dụng web deploy liên tục.

## ✍️ Viết commit tốt

### Commit nguyên tử (atomic)

Một commit = **một thay đổi logic**, và bản thân nó phải build/test được.

| ❌ Không nên | ✅ Nên |
|-------------|--------|
| "Thêm login + sửa CSS header + đổi tên biến" | Ba commit riêng |
| Commit làm hỏng build, commit sau mới sửa | Gộp lại (fixup) trước khi merge |

Lợi ích: review dễ, `revert` chính xác, `bisect` hiệu quả (bài 14), `cherry-pick` sạch.

### Message

```
feat(auth): khoá tài khoản sau 5 lần đăng nhập sai          ← tiêu đề ≤ ~50–72 ký tự, mệnh lệnh

Trước đây kẻ tấn công có thể thử mật khẩu không giới hạn.   ← thân: VÌ SAO, không phải "làm gì"
Giờ tài khoản bị khoá 15 phút sau 5 lần sai liên tiếp.       ← (code đã nói "làm gì" rồi)
Bộ đếm reset khi đăng nhập thành công.

Refs: #1234                                                  ← trailer: liên kết issue, co-author...
```

Quy tắc:

- Tiêu đề ngắn, viết ở thể **mệnh lệnh** ("Thêm", "Sửa", "Add", "Fix"), không có dấu chấm cuối.
- Dòng trống giữa tiêu đề và thân.
- Thân giải thích **vì sao** và **bối cảnh**. Hai năm sau, người đọc `git blame` cần điều này nhất.

[Conventional Commits](https://www.conventionalcommits.org/) là quy ước phổ biến: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`... Nó giúp sinh changelog và đánh số phiên bản tự động.

## 🧭 Quy trình cá nhân mẫu (GitHub Flow + rebase)

```bash
# 1. Bắt đầu từ main mới nhất
git switch main && git pull --ff-only
git switch -c feat/khoa-tai-khoan

# 2. Làm việc, commit nhỏ và thường xuyên (kể cả commit tạm)
git add -p && git commit -m "feat(auth): đếm số lần đăng nhập sai"
git commit --fixup=<hash>                     # sửa commit trước đó

# 3. Cập nhật với main (thường xuyên, để conflict nhỏ)
git fetch && git rebase origin/main

# 4. Dọn dẹp trước khi mở PR
git rebase -i --autosquash origin/main
git range-diff @{u}...HEAD                    # nếu đã từng push: xem lại đã đổi gì

# 5. Push và mở PR
git push -u origin HEAD                       # lần đầu
git push --force-with-lease --force-if-includes   # sau khi rebase

# 6. Sau khi PR được merge
git switch main && git pull --ff-only
git branch -d feat/khoa-tai-khoan             # dùng -D nếu nhóm dùng squash merge
git fetch --prune
```

### Bảo vệ nhánh chính

Trên GitHub/GitLab, nên bật cho `main`:

- Cấm push trực tiếp và force push.
- Bắt buộc PR, tối thiểu 1 review.
- Bắt buộc CI pass và nhánh phải cập nhật với `main` trước khi merge.

## ⚠️ Hiểu lầm thường gặp

| Hiểu lầm | Thực tế |
|----------|---------|
| "Git Flow là cách chuẩn dùng Git" | Đó là một mô hình trong số nhiều mô hình. Với deploy liên tục, trunk-based/GitHub Flow thường phù hợp hơn. |
| "Lịch sử thẳng luôn tốt hơn" | Thẳng thì dễ đọc, nhưng merge commit giữ thông tin tính năng. Đó là đánh đổi. |
| "Message commit không quan trọng vì đã có PR" | PR nằm trên nền tảng (có thể đổi). Message nằm vĩnh viễn trong repo, và `blame`/`log` hiện nó. |
| "Nhánh sống lâu an toàn hơn" | Nhánh càng sống lâu, conflict khi tích hợp càng lớn. |

## ✅ Tự kiểm tra

<details>
<summary>1. Nhóm dùng squash merge. Vì sao <code>git branch -d feature</code> báo not fully merged sau khi PR đã merge?</summary>

Commit squash trên `main` là commit mới, không có `E` (đầu nhánh feature) làm tổ tiên. Git chỉ dựa vào reachability nên không biết nội dung đã được đưa vào.
</details>

<details>
<summary>2. Vì sao trunk-based cần feature flag?</summary>

Vì tích hợp vào `main` liên tục, kể cả tính năng chưa xong. Feature flag ẩn tính năng dở dang trên production cho tới khi hoàn thiện.
</details>

<details>
<summary>3. Thân commit message nên tập trung vào điều gì?</summary>

Vì sao thay đổi và bối cảnh (vấn đề, lựa chọn đã cân nhắc, hạn chế). Diff đã cho thấy "làm gì".
</details>

## 🏋️ Bài tập

1. Trong một repo nháp, mô phỏng cùng một nhánh feature được tích hợp theo 3 cách (merge `--no-ff`, rebase + ff, `merge --squash` + commit) trên 3 bản sao của `main`. So sánh `git log --graph` và `git log --first-parent`.
2. Viết lại message cho 5 commit gần nhất của một dự án bạn đang làm theo chuẩn trên (chỉ viết ra giấy, đừng rewrite nhánh chung!).
3. Viết tài liệu `CONTRIBUTING.md` ngắn mô tả quy trình Git cho nhóm của bạn: mô hình nhánh, cách merge, quy ước message.

## 📎 Tra nhanh

- [Cheat sheet 16: Git Flow](../cheat-sheet/16-git-flow.md)
- Pro Git: [5.1 Distributed Workflows](https://git-scm.com/book/en/v2/Distributed-Git-Distributed-Workflows), [5.2 Contributing to a Project](https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project)
- [trunkbaseddevelopment.com](https://trunkbaseddevelopment.com/)

---

[← Trước: Viết lại lịch sử](11-viet-lai-lich-su.md) · [Lộ trình](../README.md) · [Tiếp: Stash và worktree →](../4-nang-cao/13-stash-worktree.md)
