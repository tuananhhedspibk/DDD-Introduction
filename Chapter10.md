# Chương 10: Đảm bảo tính toàn vẹn của dữ liệu

## Thế nào là tính toàn vẹn của dữ liệu

Là việc dữ liệu

- Không mâu thuẫn, đồng nhất, nhất quán
- Không có sự sai lệch

## Xác nhận những lỗi nghiêm trọng

Cùng xét đến ví dụ về `UserApplicationService`.

Khi có 2 người dùng cùng đăng kí với cùng một cái tên "naruse", theo domain rule thì hệ thống sẽ trả ra lỗi do domain rule không cho phép hai user có cùng một tên. Nhưng do việc lưu user vào DB sẽ được chuyển tiếp cho `UserRepository` nên nếu trường hợp 2 người dùng cùng đăng kí một cái tên cùng một lúc thì khi dữ liệu của một trong hai người dùng chưa được lưu vào DB, thì việc kiểm tra tên có trùng nhau hay không giữa các users sẽ bị bỏ qua.

Kết quả là có hai users mang cùng một tên "naruse".

![Screen Shot 2021-10-30 at 18 15 06](https://user-images.githubusercontent.com/79828986/139527294-3a54b6d0-15b8-4b51-ae83-9a16590dc6c7.png)

## Sử dụng unique key để phòng ngừa

Khi đã thiết lập unique key, thì chương trình (phía code) không cần phải quan tâm đến việc kiểm tra xem dữ liệu có bị trùng lặp hay không.

Tuy nhiên lúc này có một vấn đề đó là, phía use-case lại phụ thuộc vào công nghê (unique key) được sử dụng ở tầng infrastructure. Giả sử, nếu domain rule thay đổi từ việc không cho phép `username` trùng nhau thành không cho phép `email` trùng nhau, thì lúc này ta cần phải thiết lập lại unique-key ở phía relational database.

> Không nên sử dụng unique-key như một công cụ để đảm bảo domain rule, hãy sử dụng nó như một công cụ nhằm đảm bảo tính toàn vẹn cho dữ liệu

Hãy kết hợp việc sử dụng unique-key với code của chúng ta.

## Phòng ngừa bằng transaction

Cùng lấy một ví dụ về trang EC. Khi khách hàng sử dụng point đã tích luỹ được để mua hàng, hệ thống cần làm những việc sau:

1. Trừ point của khách hàng
2. Check hàng trong kho, nếu còn sẽ giảm số lượng hàng

Nhưng nếu trong kho không còn hàng mà point của khách hàng vẫn bị trừ và hàng không được gửi tới cho khách hàng thì khả năng cao khách hàng sẽ phàn nàn với trung tâm chăm sóc khách hàng của bạn.

Transaction sẽ giúp bạn giải quyết vấn đề này, point của khách hàng sẽ thực sự bị trừ khi tiến hành commit mọi thay đổi lên DB, nếu chương trình bị kết thúc giữa chừng, mọi sự thay đổi nằm trong transaction đều không có tác dụng (do không được commit).

![Screen Shot 2021-10-31 at 17 56 14](https://user-images.githubusercontent.com/15076665/139575278-8ad11ec4-a103-4950-a369-f72fe035dd1e.png)

## Các Patterns sử dụng transaction

Song song với việc sử dụng transaction, ta cũng cần phải đảm bảo việc không phụ thuộc vào công nghệ.

**1. Transaction Scope:**

Có một phương pháp để giải quyết vấn đề tránh phụ thuộc vào công nghệ khi sử dụng transaction đó là `Transaction Scope`. Transaction scope định nghĩa phạm vi thực thi transaction, xong trên thực tế nó không thực thi việc bắt đầu một transaction. Thế nhưng, trong transaction khi connection đến DB được thực hiện xong thì transaction cũng sẽ được bắt đầu thực hiện. Kết quả là trong scope này ta có thể nhận được thành quả sau khi thực hiện transaction.

Với ngôn ngữ OOP như Java, ta có thể sử dụng AOP pattern (Aspect Oriented Programming).

```Java
@Transactional
public class UserApplicationService {}
```

`@Transactional` annotation sẽ có chức năng tương tự như transaction scope. Khi method kết thúc, sẽ commit đến DB, nếu phát sinh ngoại lệ, sẽ tự động rollback.

Một ưu điểm của việc sử dụng annotation ở đây đó là không cần phải xem cụ thể nội dung method để có thể biết nó có đảm bảo tính toàn vẹn dữ liệu hay không, chỉ cần nhìn thấy `@Transactional` annotation là đủ,

**2. Unit of work:**

Là Object ghi lại sự thay đổi của các objects khác. Những việc thay đổi, xoá Object nếu không thông báo cho UnitOfWork thì sẽ không thể lưu vào DB.
Ngoài ra cũng cần gọi commit để lưu thay đổi vào DB.

Khi sử dụng pattern này, mọi sự thay đổi (Create, Update, Delete) dữ liệu vào DB đều phải thông qua UnitOfWork Object.

```TS
class UnitOfWork {
  public void registerNew(Object value);
  public void registerDirty(Object value);
  public void registerClean(Object value);
  public void registerDeleted(Object value);
  public void commit();
}
```

Các method bắt đầu với `register` đều là các method tiến hành ghi lại trạng thái của instance. `commit` method là để áp dụng sự thay đổi đó vào DB.

Ta có thể sử dụng UnitOfWork với base class Entity.

```TS
abstract class Entity {
  markNew() { UnitOfWork.registerNew() }
  markDirty() { UnitOfWork.registerDirty() }
  markClean() { UnitOfWork.registerClean() }
  markDeleted() { UnitOfWork.registerDeleted() }
}

class UserEntity extends Entity {
  constructor() { markNew() }
  changeName() { markDirty() }
}
```

Ngoài ra còn một pattern khác đó là sử dụng Repository trong UnitOfWork

```TS
class UnitOfWork {
  public UserRepository userRepository;
}
```

client sẽ sử dụng `userRepository` như một thuộc tính public để lưu trữ, thay đổi dữ liệu. Ngoài ra với cách làm này ta cũng không cần phải sử dụng base class Entity nữa.

## Transaction gây ra lock

Transaction sẽ lock DB lại để đảm bảo tính toàn vẹn của dữ liệu.

Tuy nhiên khi sử dụng transaction, nên đảm bảo rằng phạm vi lock là hẹp nhất có thể. Vì phạm vi lock càng rộng thì khả năng xử lí gặp lỗi sẽ càng cao.
