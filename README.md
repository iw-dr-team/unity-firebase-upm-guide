# Cài Firebase vào Unity theo hướng thân thiện với Git

Có nhiều cách để đưa Firebase vào dự án Unity. Cách phổ biến nhất là tải file `.unitypackage` rồi import trực tiếp vào project. Cách này vẫn dùng được, nhưng với team làm việc qua GitHub thì khá dễ phát sinh bất tiện: các thư viện native đi kèm Firebase có thể tạo ra những file rất lớn trong `Assets`, thậm chí có file vượt giới hạn 100 MB của GitHub. Khi đó bạn thường phải dùng Git LFS nếu muốn commit chúng vào repo.

Một cách khác là đưa các file lớn đó vào `.gitignore`. Cách này giúp repo nhẹ hơn, nhưng đổi lại người khác clone project về sẽ không có đủ dependency để chạy ngay. Nói ngắn gọn: ai clone repo cũng phải tự cài lại Firebase bằng tay.

Trong thực tế, mình thấy cách phù hợp hơn là cài Firebase qua Unity Package Manager bằng Git URL. Khi dùng cách này, package được khai báo trong `Packages/manifest.json`, Unity sẽ tự tải dependency về thư mục `Packages`/cache thay vì bạn phải commit toàn bộ SDK đã import vào `Assets`. Nhờ đó repo gọn hơn, và khi clone sang máy khác thì Unity cũng có thể tự khôi phục lại package từ `manifest.json`.

## Vấn đề với scoped registry `com.google`

Khi thêm scoped registry cho `com.google`, bạn sẽ thấy khá nhiều package của Google xuất hiện để cài đặt. Tuy nhiên, Firebase Unity SDK thường không có sẵn ở đó theo dạng package chính thức để cài trực tiếp như các package UPM thông thường.

May là có một organization trên GitHub đã đóng gói lại Firebase theo định dạng phù hợp để cài qua Git URL:

- [Organization phân phối Firebase](https://github.com/RageAgainstThePixel)
- [Firebase App](https://github.com/RageAgainstThePixel/com.google.firebase.app)

Lưu ý kỹ thuật: đây là nguồn package do cộng đồng phân phối lại, không phải kênh phát hành chính thức của Google cho Firebase Unity SDK. Vì vậy, bạn nên pin version cụ thể, kiểm tra changelog trước khi nâng version, và chỉ dùng khi team chấp nhận phụ thuộc vào nguồn Git trung gian này.

## Cách mình đang dùng

Ở project này, team đang khai báo trực tiếp các package Firebase trong `manifest.json`. Cách làm này đã được dùng ổn định hơn một năm ở Dancing Road và một số dự án khác của mình.

Điểm lợi chính:

- Không cần commit toàn bộ Firebase SDK đã import vào `Assets`.
- Repo gọn hơn và ít vướng giới hạn file lớn của GitHub hơn.
- Máy mới clone project về có thể để Unity tự restore package từ `manifest.json`.
- Dễ pin đúng version Firebase cho cả team.
- Khi cần nâng hoặc hạ version, thường chỉ cần sửa tag version ở cuối Git URL trong `manifest.json`, ví dụ đổi từ `#12.10.1` sang version khác phù hợp.

Dưới đây là phần `dependencies` mình đang dùng. Nếu cần thêm dịch vụ Firebase khác, bạn có thể vào organization ở trên, tìm đúng repo package tương ứng rồi thêm theo cùng format.

```json
"com.google.external-dependency-manager": "https://github.com/googlesamples/unity-jar-resolver.git?path=upm#v1.2.187",
"com.google.firebase.analytics": "https://github.com/RageAgainstThePixel/com.google.firebase.analytics.git#12.10.1",
"com.google.firebase.app": "https://github.com/RageAgainstThePixel/com.google.firebase.app.git#12.10.1",
"com.google.firebase.auth": "https://github.com/RageAgainstThePixel/com.google.firebase.auth.git#12.10.1",
"com.google.firebase.crashlytics": "https://github.com/RageAgainstThePixel/com.google.firebase.crashlytics.git#12.10.1",
"com.google.firebase.database": "https://github.com/RageAgainstThePixel/com.google.firebase.database.git#12.10.1",
"com.google.firebase.installations": "https://github.com/RageAgainstThePixel/com.google.firebase.installations.git#12.10.1",
"com.google.firebase.messaging": "https://github.com/RageAgainstThePixel/com.google.firebase.messaging.git#12.10.1",
"com.google.firebase.remote-config": "https://github.com/RageAgainstThePixel/com.google.firebase.remote-config.git#12.10.1",
```

## Ghi chú thêm

- `com.google.external-dependency-manager` vẫn cần thiết để resolve các dependency native cho Android/iOS.
- Sau khi sửa `manifest.json`, Unity có thể cần một lúc để tải package và resolve lại dependency.
- Nếu project đã có `packages-lock.json`, version thực tế được restore có thể bị khóa theo file đó cho tới khi bạn cập nhật lock file.
