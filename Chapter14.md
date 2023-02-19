# Chương 14: Kiến trúc (Architecture)

## Mục đích của kiến trúc (architecture)

### Anti pattern: Smart UI

Smart UI tức là mọi domain rules đều được thể hiện trên UI. Cùng lấy ví dụ với một trang EC cổ điển. Ở các trang EC này người dùng sẽ mua hàng từ website.

![Screen Shot 2022-01-03 at 16 01 46](https://user-images.githubusercontent.com/15076665/147906037-62a86626-3e91-436f-a862-0e13081c95f9.png)

Như các trang EC bình thường, người dùng sẽ mua hàng, xác nhận đơn hàng (có hiển thị giá tiền), nhấn nút xác nhận mua hàng. Người dùng hoàn toàn có thể xem toàn bộ các đơn hàng ở màn hình "Danh sách các đơn hàng", nếu ở màn hình danh sách này có hiển thị giá tiền một đơn hàng thì sẽ rất tiện lợi.

Nhưng ở màn hình danh sách các đơn hàng, người dùng không thể biết được cụ thể đơn hàng đó có những gì. Người dùng sẽ lại phải mở giao diện xác nhận đơn hàng ra để xem (ở giao diện này có hiện thị giá tiền đơn hàng).

![Screen Shot 2022-01-03 at 16 07 37](https://user-images.githubusercontent.com/15076665/147906318-830747e3-83f6-44d5-b93e-df57d5bbe515.png)

Ở hình phía trên chúng ta thấy ở màn hình nào cũng cần đến business logic là "Tính tiền đơn hàng". Sẽ có những vấn đề sau có thể xảy ra:

- Nếu logic nằm rải rác như vậy, trong trường hợp code cũng nằm rải rác tương tự thì khi logic tính tiền thay đổi, việc sửa lại hệ thống sẽ rất tốn thời gian và chi phí
- Khi hệ thống phát triển, mỗi đơn hàng có thể có logic tính tiền khác nhau (sales campaign, coupon, ...) nên code cũng sẽ phải thay đổi tuỳ theo đơn hàng.

UI chỉ đơn thuần là nơi nhập liệu và hiển thị, mọi business logic và những thứ liên quan nên hạn chế có mặt ở UI nhiều nhất có thể. Nên giữ cho UI "ngốc" nhất có thể.　UI càng thông minh thì chi phí sửa và bảo trì sau này sẽ càng tốn kém.

![Screen Shot 2022-01-03 at 17 05 23](https://user-images.githubusercontent.com/15076665/147909766-e4a1adae-ae0d-41af-bac5-5c0a9396e8d4.png)

Hãy quên Smart UI đi, business logic chỉ nên tập trung ở một nơi duy nhất như hình phía trên. Khi đó sẽ không có chỗ cho việc tính toán giá tiền dựa theo điều kiện của từng màn hình nữa. Ngoài ra việc sửa đổi logic tính toán cũng sẽ chỉ diễn ra ở một nơi duy nhất mà thôi.

## DDD cần có kiến trúc

Việc tránh để các business logic xuất hiện ở UI đối với dev là một công việc không hề đơn giản. Có một giải pháp hữu hiệu đó là `Kiến trúc` - `Architecture`.

Kiến trúc là công cụ giúp dev có thể biết được logic nào nên viết ở đâu, qua đó tránh được sự mất trật tự trong việc sắp xếp các logic để từ đó dev có thể tập trung toàn lực cho việc "nắm bắt domain và hiển thị nó một cách tốt nhất".

Điều mà DDD cần nhất ở kiến trúc đó là tránh việc bị cuốn vào domain object, cũng như cách ly những yêu cầu đặc thù của phần mềm đối với domain object.

## Giải thích về kiến trúc

Dưới đây là các kiến trúc thường đi kèm với DDD:

- Clean Architecture
- Layered Architecture
- Hexagonal Architecture

Điều quan trọng nhất ở đây đó là cách ly Architecture type với domain, ta chỉ nên tập trung vào bản chất của domain và tạo ra môi trường phục vụ cho domain đó.

## Layered Architecture

Là kiến trúc kiểu phân tầng (cổ điển và kinh điển nhất)

![Screen Shot 2022-01-03 at 18 10 17](https://user-images.githubusercontent.com/15076665/147914431-bf9f5c50-81fc-4aa8-8ac9-285ebb1f225e.png)

Kiến trúc này gồm 4 tầng:

- Presentation: UI
- Application: tầng chứa các usecase
- Domain: chứa business logic
- Infrastructure
