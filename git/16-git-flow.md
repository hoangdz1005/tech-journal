# 16. Git Flow

[← Mục lục](README.md)

Git Flow là mô hình phân nhánh cho Git, cung cấp khung làm việc chặt chẽ cho các dự án lớn. Mô hình này định nghĩa chiến lược phân nhánh xoay quanh chu kỳ phát hành, gồm hai nhánh chính (`main` và `develop`) và các nhánh phụ cho tính năng (feature), phát hành (release) và sửa lỗi khẩn (hotfix). Nhờ đó công việc được tổ chức tốt, lịch sử sạch sẽ, và vai trò của từng loại công việc được phân định rõ ràng.

| Nhánh | Tạo từ | Merge vào | Mục đích |
|-------|--------|-----------|----------|
| `main` | — | — | Mã đang chạy production |
| `develop` | `main` | — | Tích hợp các tính năng cho bản phát hành tới |
| `feature/*` | `develop` | `develop` | Phát triển tính năng mới |
| `release/*` | `develop` | `main` và `develop` | Chuẩn bị phát hành |
| `hotfix/*` | `main` | `main` và `develop` | Sửa lỗi khẩn cấp trên production |

> ⚠️ **Lưu ý:** `git flow` **không** có sẵn trong Git, cần cài extension riêng (ví dụ `brew install git-flow-avh` trên macOS, hoặc bản `git-flow-next`).

### `git flow init`

Khởi tạo repository theo mô hình phân nhánh git-flow (hỏi tên các nhánh và tiền tố).

### `git flow feature start <feature>`

Tạo nhánh tính năng mới theo git-flow (tạo `feature/<feature>` từ `develop`).

### `git flow feature finish <feature>`

Hoàn tất nhánh tính năng theo git-flow (merge vào `develop` và xoá nhánh tính năng).

### Các lệnh tương tự cho release và hotfix

```bash
git flow release start 1.2.0
git flow release finish 1.2.0

git flow hotfix start 1.2.1
git flow hotfix finish 1.2.1
```

---

[← Trước: Tìm kiếm & Bisect](15-tim-kiem-va-bisect.md) · [Mục lục](README.md) · [Tiếp: Worktree →](17-worktree.md)
