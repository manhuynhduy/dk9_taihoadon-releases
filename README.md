# dk9_taihoadon-releases

Nơi phát hành bản cài **dk9hoadon** — ứng dụng tải hóa đơn điện tử từ Cổng
hóa đơn điện tử Tổng cục Thuế (hoadondientu.gdt.gov.vn).

| File | Dùng cho |
|------|----------|
| [`dk9hoadon_v4.6.18.exe.zip`](dk9hoadon_v4.6.18.exe.zip) | Windows 64-bit |
| [`dk9hoadon_v4.6.18_macos_arm.zip`](dk9hoadon_v4.6.18_macos_arm.zip) | macOS Apple Silicon (M1/M2/M3…) |
| [`MyRootCA.crt`](MyRootCA.crt) | Chứng chỉ gốc — cài 1 lần trên máy Windows (xem bên dưới) |
| `latest.json` | Thông tin bản mới nhất, app tự đọc để kiểm tra cập nhật |

> macOS Intel: hiện **không có** bản đóng gói.

---

## Cài chứng chỉ để Windows tin bản .exe

File `.exe` được ký số bằng chứng chỉ riêng của DK9 (không mua từ nhà cung cấp
công cộng như DigiCert/Sectigo). Chuỗi ký **không nhúng chứng chỉ gốc** bên
trong file, nên nếu chưa cài `MyRootCA.crt` thì Windows không dựng được chuỗi
tin cậy và sẽ hiện **"Unknown publisher" / "Nhà phát hành không rõ"**.

Cài `MyRootCA.crt` **một lần duy nhất trên mỗi máy** là xong — mọi bản cập nhật
sau không phải cài lại.

### Cách 1 — Bấm đúp (dễ nhất)

1. Tải [`MyRootCA.crt`](MyRootCA.crt) về máy.
2. **Kiểm tra dấu vân tay trước khi cài** (xem mục [Xác minh](#xác-minh-chứng-chỉ-trước-khi-cài) — đừng bỏ qua bước này).
3. Bấm đúp file → **Install Certificate…**
4. Chọn **Local Machine** → **Next** (Windows sẽ hỏi quyền Administrator).
   *Chọn Current User thì chỉ tài khoản đang đăng nhập tin, các tài khoản khác trên máy vẫn báo không tin cậy.*
5. Chọn **Place all certificates in the following store** → **Browse…** →
   **Trusted Root Certification Authorities** → **OK**.
6. **Next** → **Finish**. Windows hiện cảnh báo bảo mật → **Yes**.
7. Xong. Mở lại file `.exe`, phần Publisher giờ hiện **DK9 Software Publisher**.

### Cách 2 — PowerShell (chạy bằng quyền Administrator)

```powershell
Import-Certificate -FilePath "$HOME\Downloads\MyRootCA.crt" -CertStoreLocation Cert:\LocalMachine\Root
```

Triển khai nhiều máy cùng lúc thì dùng `certutil`:

```powershell
certutil -addstore -f Root "\\may-chu\chia-se\MyRootCA.crt"
```

---

## Xác minh chứng chỉ trước khi cài

Cài một chứng chỉ gốc là việc nên làm cẩn thận. **Hãy đối chiếu dấu vân tay**
dưới đây với file bạn vừa tải; nếu lệch thì **không cài** và báo lại người quản
trị.

```
Tên (CN)    : DK9CustomDevRootCA
Có hiệu lực : 18/08/2026 → 15/08/2036
SHA-256     : BF:49:AC:28:D3:DF:8A:66:1A:16:97:95:B3:9C:C9:06:
              F4:0C:F1:44:5E:2F:56:27:2E:62:2C:C4:84:5B:FE:AD
```

Xem dấu vân tay của file đã tải — PowerShell:

```powershell
Get-FileHash -Algorithm SHA256 "$HOME\Downloads\MyRootCA.crt"   # hash của FILE
(Get-PfxCertificate "$HOME\Downloads\MyRootCA.crt").Thumbprint  # vân tay CHỨNG CHỈ (SHA-1)
```

Chính xác nhất là xem bằng `certutil` (hiện cả SHA-256 của chứng chỉ):

```powershell
certutil -dump "$HOME\Downloads\MyRootCA.crt"
```

---

## Kiểm tra bản .exe đã ký đúng chưa

Sau khi cài chứng chỉ gốc, chạy trong PowerShell:

```powershell
Get-AuthenticodeSignature "C:\duong\dan\dk9hoadon_v4.6.18.exe" | Format-List Status, StatusMessage, SignerCertificate
```

| Kết quả | Nghĩa là |
|---------|----------|
| `Valid` | Đã ký và Windows tin chuỗi ký — đúng như mong đợi |
| `UnknownError` | Đã ký nhưng **chưa cài** `MyRootCA.crt`, hoặc cài sai store |
| `NotSigned` | File chưa được ký — không nên chạy, hãy tải lại từ repo này |
| `HashMismatch` | File bị sửa đổi sau khi ký — **không chạy**, tải lại |

Chuỗi ký của bản phát hành:

```
Chứng chỉ ký : CN=DK9 Software Publisher   (dùng để ký, hết hạn 17/08/2028)
Cấp bởi      : CN=DK9CustomDevRootCA       (chứng chỉ gốc, hết hạn 15/08/2036)
Mục đích     : Code Signing
```

---

## Điều cần biết trước khi cài

**Cài chứng chỉ gốc nghĩa là gì.** Máy của bạn sẽ tin **mọi** chứng chỉ do
`DK9CustomDevRootCA` cấp, không chỉ riêng app này. Chứng chỉ gốc này không giới
hạn mục đích sử dụng, nên về nguyên tắc ai giữ được khóa riêng của nó cũng có
thể ký phần mềm khác hoặc cấp chứng chỉ HTTPS mà máy bạn tin. Vì vậy:

- Chỉ cài trên máy thuộc quyền quản lý của DK9, dùng cho công việc.
- Chỉ cài khi dấu vân tay khớp đúng chuỗi SHA-256 ở trên.
- Không cài trên máy cá nhân dùng chung với người khác nếu không cần thiết.
- Gỡ bỏ khi không dùng app nữa (xem mục dưới).

**Cài chứng chỉ không làm mất hẳn cảnh báo SmartScreen.** Cài rồi thì Windows
hiện đúng tên nhà phát hành thay vì "Unknown publisher", nhưng SmartScreen còn
dựa vào *uy tín* của file và chứng chỉ. Chứng chỉ tự ký không có uy tín tích lũy
như chứng chỉ mua từ nhà cung cấp công cộng, nên lần đầu chạy vẫn có thể gặp
màn hình xanh **"Windows protected your PC"** → bấm **More info** → **Run
anyway**. Cảnh báo này thường hết sau khi file được chạy nhiều lần trên nhiều
máy.

---

## Gỡ chứng chỉ

PowerShell với quyền Administrator:

```powershell
Get-ChildItem Cert:\LocalMachine\Root |
  Where-Object { $_.Subject -eq 'CN=DK9CustomDevRootCA' } |
  Remove-Item
```

Hoặc giao diện: `certlm.msc` → **Trusted Root Certification Authorities** →
**Certificates** → tìm `DK9CustomDevRootCA` → xóa.

---

## Câu hỏi thường gặp

**Cập nhật app lên bản mới có phải cài lại chứng chỉ không?**
Không. Windows tin theo **chuỗi chứng chỉ gốc**, không tin từng file `.exe`.
Mọi bản ký bởi chuỗi này đều được chấp nhận sau một lần cài.

**Khi nào mới phải cài lại?**
Chỉ khi đổi hẳn chứng chỉ gốc — ví dụ gốc hết hạn (sau 15/08/2036) hoặc khóa
riêng bị lộ nên phải thay. Chứng chỉ ký con hết hạn (17/08/2028) mà vẫn do cùng
gốc cấp thì **không** cần cài lại.

**Máy không cài chứng chỉ thì app có chạy không?**
Vẫn chạy. Chỉ là Windows báo nhà phát hành không rõ và SmartScreen cản nhiều
hơn.

**File `MyRootCA.crt` công khai như vậy có an toàn không?**
Có. Đây là phần **công khai** của chứng chỉ, giống như khóa công khai — nó chỉ
dùng để *xác minh* chữ ký, không dùng để *tạo* chữ ký. Khóa riêng (`.key`,
`.pfx`) không nằm trong repo này và không được công khai.

**macOS có cần cài gì không?**
Không. Bản macOS dùng cơ chế ký/notarize riêng của Apple, không liên quan
chứng chỉ này.
