# 15. Tìm kiếm & Bisect

[← Mục lục](README.md)

## Tìm kiếm

`git grep` tìm chuỗi hoặc mẫu (pattern) trong các file của repository. Đây là cách nhanh để tìm mã, chú thích hay văn bản trong nhiều file, giúp làm quen và điều hướng codebase lớn. Có nhiều tuỳ chọn để tìm kiếm có mục tiêu, rất tiện cho việc phân tích và bảo trì mã.

### `git grep <pattern>`

Tìm chuỗi trong working directory và index.

> ⚠️ **Lưu ý:** Mặc định `git grep` tìm trong các **file đang được theo dõi** ở working directory. Dùng `--cached` để tìm trong index, `--untracked` để tìm cả file untracked, hoặc truyền commit để tìm trong một phiên bản cũ.

```bash
git grep "TODO"
git grep -n "useState" -- "*.tsx"   # hiện số dòng, lọc theo đuôi file
git grep "API_KEY" v1.0.0           # tìm trong tag v1.0.0
```

### `git grep -e <pattern>`

Tìm theo một mẫu cụ thể. `-e` hữu ích khi mẫu bắt đầu bằng `-` hoặc khi kết hợp nhiều mẫu.

```bash
git grep -e "--force"
git grep -e "foo" --and -e "bar"
```

---

## Bisect

`git bisect` là công cụ gỡ lỗi giúp tìm chính xác commit đã gây ra lỗi. Git dùng **tìm kiếm nhị phân** trên lịch sử commit để thu hẹp phạm vi nhanh chóng. Bạn đánh dấu một commit tốt (chưa có lỗi) và một commit xấu (đã có lỗi), sau đó lần lượt kiểm tra các commit ở giữa và đánh dấu tốt/xấu. Sau vài bước, Git chỉ ra commit gây lỗi.

### `git bisect start`

Bắt đầu phiên bisect.

### `git bisect bad`

Đánh dấu phiên bản hiện tại là xấu (có lỗi).

### `git bisect good <commit>`

Đánh dấu commit chỉ định là tốt (không có lỗi).

### `git bisect reset`

Kết thúc phiên bisect và quay về nhánh ban đầu.

### `git bisect visualize`

Mở công cụ trực quan (ví dụ `gitk`, hoặc `git log` nếu không có giao diện) để xem các commit còn lại cần kiểm tra.

### Ví dụ quy trình đầy đủ

```bash
git bisect start
git bisect bad                 # commit hiện tại có lỗi
git bisect good v1.2.0         # v1.2.0 chạy tốt

# Git checkout một commit ở giữa → bạn kiểm tra rồi đánh dấu:
git bisect good                # hoặc: git bisect bad
# ... lặp lại cho tới khi Git báo "<hash> is the first bad commit"

git bisect reset
```

> 💡 Tự động hoá bằng script test (exit code 0 = good, khác 0 = bad):
>
> ```bash
> git bisect run npm test
> ```

---

[← Trước: Bảo trì & dọn dẹp](14-bao-tri-va-don-dep.md) · [Mục lục](README.md) · [Tiếp: Git Flow →](16-git-flow.md)
