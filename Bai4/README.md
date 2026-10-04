# Báo cáo: Quản lý tệp tin bỏ qua (.gitignore) và Sửa lịch sử (Amend)

- **Họ và tên**: Quang Nguyen
- **Mã sinh viên**: [Mã sinh viên]
- **Bài tập**: Bài 4 - Session 04: Quản lý tệp tin bỏ qua (.gitignore) và Sửa lịch sử (Amend)
- **Đường dẫn nộp bài trên GitHub**: `homework/session_04/ex4/`

---

## 1. Mục tiêu thực hành
- Cấu hình tệp tin ẩn `.gitignore` để tự động bỏ qua các file nhạy cảm (`credentials.txt`) và thư mục cấu hình môi trường/IDE (`.idea/`).
- Sử dụng lệnh `git rm --cached` để gỡ bỏ file đã commit ra khỏi cache theo dõi (Staging Area/Index) của Git một cách an toàn mà **không làm mất file vật lý trên đĩa cứng**.
- Chỉnh sửa thông điệp và nội dung của commit gần nhất thông qua tùy chọn `--amend` (`git commit --amend`) để làm sạch lịch sử commit.
- Nắm vững cơ chế hoạt động của 3 vùng nhớ trong Git (Working Directory, Staging Area, Git Repository) và nguyên tắc an toàn khi xử lý dữ liệu nhạy cảm.

---

## 2. Bối cảnh & Vấn đề thực tế
- **Bối cảnh**: Trong quá trình khởi tạo dự án, người lập trình vô tình thêm và commit tệp `credentials.txt` (chứa các thông tin bảo mật nhạy cảm như Database Host, User, Password, API Secret Key) vào Git repository với thông điệp:
  `feat: Khởi tạo dự án và cấu hình credentials`
- **Vấn đề đặt ra**:
  1. Nếu đẩy (push) commit này lên remote repository công khai (như GitHub, GitLab), các thông tin bảo mật sẽ bị lộ lọt, gây nguy cơ an ninh nghiêm trọng.
  2. Cần gỡ bỏ `credentials.txt` khỏi sự theo dõi của Git nhưng **bắt buộc phải giữ lại tệp tin trên thư mục làm việc cục bộ** để phục vụ việc cấu hình và chạy ứng dụng ở máy local (ràng buộc không được xóa vật lý file trên hệ điều hành).
  3. Cần cấu hình để Git tự động bỏ qua tệp này trong tương lai, tránh việc vô tình `git add .` lại thêm vào một lần nữa.
  4. Cần làm sạch lịch sử Git: sửa lại thông điệp commit gần nhất sao cho không còn nhắc đến việc commit credentials, đồng thời loại bỏ hoàn toàn file này khỏi commit đó mà không tạo thêm commit rác.

---

## 3. Cơ sở lý thuyết & Giải thích chi tiết câu lệnh

### 3.1. Phân biệt `git rm` và `git rm --cached`
Git quản lý trạng thái tệp tin qua 3 vùng:
1. **Working Directory**: Thư mục làm việc thực tế chứa các file trên ổ cứng máy tính.
2. **Staging Area (Index / Cache)**: Vùng đệm chuẩn bị các thay đổi trước khi ghi vào lịch sử.
3. **Repository (Commit History)**: Kho lưu trữ chứa các commit chính thức.

So sánh giữa hai lệnh xóa:
- **`git rm <file>`**: Xóa tệp tin ở cả **Staging Area** lẫn **Working Directory**. Lệnh này sẽ xóa hẳn file vật lý trên đĩa cứng (người dùng sẽ mất file cục bộ).
- **`git rm --cached <file>`**: Chỉ gỡ bỏ tệp tin khỏi **Staging Area (Index/Cache)**, đưa tệp tin về trạng thái không được theo dõi (*untracked*), nhưng **giữ nguyên 100% tệp tin vật lý trên Working Directory**. 
  - *Ý nghĩa*: Đây là giải pháp hoàn hảo đáp ứng đúng ràng buộc đề bài, giúp bảo toàn tệp `credentials.txt` ở máy cục bộ mà Git không còn theo dõi nó nữa.

### 3.2. Vai trò và cơ chế hoạt động của `.gitignore`
- Tệp `.gitignore` quy định các quy tắc (patterns) mà Git sẽ tự động bỏ qua, không đưa vào danh sách tệp chưa được theo dõi (*Untracked files*).
- **Lưu ý quan trọng**: `.gitignore` **chỉ có tác dụng với các tệp tin chưa được theo dõi (untracked)**. Nếu một tệp đã từng được commit vào Git trước đó, việc chỉ thêm tên tệp vào `.gitignore` sẽ **không** tự động bỏ theo dõi tệp đó. Do đó, quy trình bắt buộc phải là:
  1. Gỡ file ra khỏi cache của Git bằng lệnh: `git rm --cached <file>`.
  2. Khai báo quy tắc bỏ qua trong tệp `.gitignore`.

### 3.3. Cơ chế hoạt động của `git commit --amend`
- Lệnh `git commit --amend` cho phép sửa đổi (ghi đè) commit ở đỉnh nhánh hiện tại (`HEAD`):
  - Gộp tất cả các thay đổi đang có trong Staging Area vào commit gần nhất.
  - Cho phép cập nhật hoặc thay đổi thông điệp commit (commit message).
  - Git sẽ tạo ra một commit object mới (với mã băm SHA-1 mới) để thay thế hoàn toàn commit cũ, thay vì tạo thêm một commit mới nối tiếp.
- Nhờ `--amend`, commit cũ có chứa thông tin commit nhầm `credentials.txt` sẽ được thay thế bằng commit mới sạch sẽ, không để lại dấu vết file nhạy cảm trong commit đó.

---

## 4. Các bước thực hiện chi tiết

### Bước 1: Trạng thái ban đầu (Commit nhầm file nhạy cảm)
Ban đầu, repository đã thực hiện commit khởi tạo có chứa cả file nhạy cảm:
```bash
git commit -m "feat: Khởi tạo dự án và cấu hình credentials"
```
Thông tin commit ban đầu hiển thị trong lịch sử:
- File được theo dõi: `README.md`, `credentials.txt`.
- Commit message: `feat: Khởi tạo dự án và cấu hình credentials`.

### Bước 2: Gỡ bỏ `credentials.txt` khỏi cache của Git
Thực thi lệnh gỡ bỏ tệp tin ra khỏi vùng theo dõi của Git nhưng vẫn giữ lại trên đĩa cứng:
```bash
git rm --cached credentials.txt
```
**Kết quả**:
```text
rm 'credentials.txt'
```
Lúc này, trên Staging Area ghi nhận hành động xóa bỏ tệp khỏi Git index, nhưng tệp `credentials.txt` vẫn nằm nguyên vẹn tại thư mục dự án trên ổ cứng.

### Bước 3: Cấu hình tệp `.gitignore`
Tạo và chỉnh sửa tệp `.gitignore` tại thư mục gốc của repository với nội dung:
```gitignore
# Tệp tin nhạy cảm chứa thông tin bảo mật
credentials.txt

# Cấu hình IDE JetBrains / IntelliJ
.idea/
```
Đưa tệp `.gitignore` vào Staging Area để chuẩn bị ghi nhận:
```bash
git add .gitignore
```

### Bước 4: Sửa đổi commit gần nhất bằng `git commit --amend`
Gộp thay đổi (xóa file `credentials.txt` khỏi cache và bổ sung `.gitignore`) vào commit gần nhất, đồng thời sửa đổi thông điệp commit thành thông điệp chuẩn xác, sạch sẽ:
```bash
git commit --amend -m "Initial commit: Khởi tạo dự án"
```
Lúc này, commit ban đầu đã được ghi đè hoàn toàn:
- File `credentials.txt` bị loại bỏ khỏi commit.
- File `.gitignore` và `README.md` được ghi nhận vào commit.
- Thông điệp được sửa thành `Initial commit: Khởi tạo dự án`.

---

## 5. Kết quả kiểm tra & Minh chứng

### 5.1. Kiểm tra trạng thái tệp tin (`git status`)
Thực thi lệnh:
```bash
git status
```
**Kết quả hiển thị trên Terminal:**
```text
On branch main
nothing to commit, working tree clean
```

Kiểm tra chi tiết trạng thái bỏ qua của file nhạy cảm:
```bash
git status --ignored
```
**Kết quả hiển thị:**
```text
On branch main
Ignored files:
  (use "git add -f <file>..." to include in what will be committed)
	.idea/
	credentials.txt

nothing to commit, working tree clean
```
**Đánh giá:**
- Tệp `credentials.txt` không còn xuất hiện trong danh sách theo dõi của Git (không ở dạng `staged`, `modified`, hay `untracked`).
- Tệp đã được chuyển vào danh sách `Ignored files` do `.gitignore` quản lý.

### 5.2. Kiểm tra lịch sử commit gần nhất (`git log -n 1`)
Thực thi lệnh:
```bash
git log -n 1
```
**Kết quả hiển thị trên Terminal:**
```text
commit cae46f9d1338f25b395ae612600caa0a7645d4b1
Author: Quang Nguyen <nguyendangquang0510@example.com>
Date:   Sat Oct 3 20:46:26 2026 +0700

    Initial commit: Khởi tạo dự án
```

Kiểm tra danh sách các file nằm trong commit gần nhất:
```bash
git show --stat
```
**Kết quả hiển thị:**
```text
commit cae46f9d1338f25b395ae612600caa0a7645d4b1
Author: Quang Nguyen <nguyendangquang0510@example.com>
Date:   Sat Oct 3 20:46:26 2026 +0700

    Initial commit: Khởi tạo dự án

 .gitignore | 5 +++++
 README.md  | ...
```
**Đánh giá:**
- Commit gần nhất đã hiển thị thông điệp đã sửa đổi thành công: `Initial commit: Khởi tạo dự án`.
- Trong cây tệp tin của commit chỉ có `.gitignore` và `README.md`, hoàn toàn không có `credentials.txt`.

### 5.3. Xác minh tệp tin vật lý trên ổ đĩa cục bộ
Kiểm tra sự tồn tại của tệp `credentials.txt` trong thư mục làm việc:
```powershell
Get-ChildItem -Path credentials.txt
```
**Kết quả hiển thị:**
```text
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         10/3/2026   8:46 PM            159 credentials.txt
```
Nội dung file vật lý vẫn được bảo toàn nguyên vẹn:
```text
# THÔNG TIN BẢO MẬT HỆ THỐNG
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=admin
DB_PASSWORD=SuperSecretPassword123!
API_SECRET_KEY=sec_live_987654321abcdef
```
**Đánh giá:**
- Đáp ứng tuyệt đối ràng buộc của đề bài: Không sử dụng lệnh xóa vật lý, giữ lại tệp tin ở thư mục làm việc cục bộ.

---

## 6. Lưu ý an toàn & Best Practices khi làm việc với Git

1. **Nguyên tắc khi sử dụng `git commit --amend`**:
   - Chỉ nên dùng `--amend` cho các commit **ở môi trường cục bộ (local)** chưa được `push` lên remote repository công khai.
   - Nếu commit đã được `push` lên remote, việc `--amend` sẽ làm thay đổi mã SHA của commit, buộc phải sử dụng `git push --force`, điều này có thể gây xung đột lịch sử nghiêm trọng cho các thành viên khác trong nhóm làm việc.

2. **Xử lý sự cố rò rỉ thông tin mật (Secrets Leakage)**:
   - Nếu commit chứa mật khẩu/API key đã bị `push` lên remote repository công khai (như GitHub):
     - Việc sửa commit bằng `git commit --amend` hoặc `git filter-repo` / BFG Repo-Cleaner chỉ làm sạch mã nguồn hiện tại nhưng các web cache, GitHub events, hoặc bản clone của người khác vẫn có thể đã lưu lại thông tin.
     - **Hành động quan trọng và cấp thiết nhất**: Ngay lập tức vô hiệu hóa (revoke), thay đổi mật khẩu (rotate credentials) và tái cấp phát API Key mới cho hệ thống.

3. **Cấu hình `.gitignore` chuẩn từ đầu**:
   - Luôn luôn tạo tệp `.gitignore` trước khi thực hiện commit đầu tiên của dự án.
   - Cần cấu hình bỏ qua các tệp môi trường (`.env`, `credentials.txt`, `*.pem`, `*.key`), các thư mục biên dịch / phụ thuộc (`node_modules/`, `target/`, `dist/`), và tệp cấu hình của IDE cá nhân (`.idea/`, `.vscode/`, `.DS_Store`).
