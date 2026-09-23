# Git Cheat Sheet (Tiếng Việt)

[← Về lộ trình học Git](../README.md)

Tra cứu nhanh các lệnh Git thường dùng, chia theo chủ đề. Mỗi lệnh có mô tả ngắn và ví dụ khi cần.

> 📚 Đây là tài liệu **tra cứu**. Muốn hiểu từng lệnh làm gì bên trong, hãy học theo [lộ trình](../README.md).

> Bộ tài liệu dựa trên cấu trúc và danh sách lệnh của **Git Cheat Sheet** của Flavio Copes ([flaviocopes.com](https://flaviocopes.com)).
> Phần diễn giải được viết lại bằng tiếng Việt. Chỗ nào bản gốc chưa chính xác thì có thêm ghi chú **⚠️ Lưu ý**.

## Yêu cầu trước khi đọc

- Biết dùng terminal / command line cơ bản.
- Hiểu sơ lược về lập trình và khái niệm quản lý phiên bản (version control).

## Mục lục

| # | Tài liệu | Các mục trong bản gốc |
|---|----------|------------------------|
| 01 | [Lệnh cơ bản, Working Directory & Staging Area](01-co-ban.md) | Basic Git Commands · The Working Directory and the Staging Area |
| 02 | [Branch & Checkout](02-branch.md) | Working with Branches · Checkout |
| 03 | [Merge & xử lý xung đột](03-merge-va-xung-dot.md) | Merging · Handling Merge Conflicts |
| 04 | [Remote, Fetch/Pull, Force Push](04-remote.md) | Remotes · Fetching and Pulling · Force Pushing |
| 05 | [Sửa commit & Squash](05-amend-va-squash.md) | Amending Commits · Squashing |
| 06 | [Stash](06-stash.md) | Stashing |
| 07 | [Tag](07-tag.md) | Tagging |
| 08 | [Hoàn tác thay đổi & Reflog](08-hoan-tac-va-reflog.md) | Reverting Changes · Reflog |
| 09 | [Lịch sử: Log, ngày tương đối, Blame](09-lich-su.md) | Viewing History Logs · Relative dates · Blaming |
| 10 | [Diff](10-diff.md) | Diffs |
| 11 | [Rebase, Cherry-pick, Patch](11-rebase-cherry-pick-patch.md) | Rebasing · Cherry-Picking · Patching |
| 12 | [Cấu hình, bảo mật, alias, attributes](12-cau-hinh.md) | Configuration · Security · Setting Aliases · Attributes |
| 13 | [Tham chiếu, tracking & index](13-tham-chieu-va-index.md) | Exploring Git References · Tracking · Index Manipulation |
| 14 | [Toàn vẹn dữ liệu, dọn dẹp & đóng gói](14-bao-tri-va-don-dep.md) | Data Integrity · Cleanup · Handling Untracked Files · Archiving |
| 15 | [Tìm kiếm & Bisect](15-tim-kiem-va-bisect.md) | Searching · Bisecting |
| 16 | [Git Flow](16-git-flow.md) | Git Flow |
| 17 | [Worktree](17-worktree.md) | Working trees |
| 18 | [Subtree & Submodule](18-subtree-va-submodule.md) | Subtree · Submodules |

## Quy ước trong tài liệu

- `<...>` là giá trị bạn cần thay vào, ví dụ `<branch>` → `feature/login`.
- `<commit>` có thể là hash (`a1b2c3d`), tên nhánh, tag hoặc biểu thức như `HEAD~2`.
- Các thuật ngữ quen thuộc (commit, branch, merge, rebase, stash...) được giữ nguyên tiếng Anh.

## Bảng thuật ngữ nhanh

| Thuật ngữ | Ý nghĩa |
|-----------|---------|
| Repository (repo) | Kho chứa mã nguồn cùng toàn bộ lịch sử thay đổi |
| Working directory | Thư mục làm việc, nơi bạn trực tiếp sửa file |
| Staging area / index | Vùng chuẩn bị, chứa những thay đổi sẽ vào commit tiếp theo |
| Commit | Một "ảnh chụp" trạng thái của dự án kèm thông điệp mô tả |
| Branch | Nhánh phát triển độc lập |
| HEAD | Con trỏ tới commit/nhánh hiện tại |
| Remote | Repository từ xa (GitHub, GitLab...) |
| Upstream | Nhánh/remote mà nhánh local đang theo dõi |
