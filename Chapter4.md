# Chương 4: Sử dụng domain-service để tăng tính mềm dẻo cho hệ thống

## Domain service là gì ?

Trong hệ thống sẽ có các phương thức không phù hợp để đưa vào `value-object` hoặc `entity`. Những phương thức như vậy sẽ được đưa vào `Domain Service`.

① **Những phương thức không phù hợp**
Ta lấy ví dụ với hệ thống có User, Domain rule quy định các User không được phép trùng tên nhau. Nếu vậy thì nơi ta nên định nghĩa phương thức kiểm tra đó là entity.

```js
class User {
  private readonly UserName userName;

  constructor(UserName name) {
    this.userName = name;
  }

  exists(User user) {
    // check user exists or not
  }
}
```

Khi đó nếu kiểm tra user có tồn tại hay không ta sẽ gọi như sau:

```js
user.exists(otherUser)
```

Biến `user` tự gọi đến hàm `exists` thì trông không hợp lí một chút nào cả. Vậy nên ta sẽ đưa nó vào trong `Domain Service`.

② **Giải quyết sự không phù hợp**

Về bản chất thì `Domain Service` cũng là một object. `Domain Service` cho User sẽ viết như sau:

```js
class UserService {
  public boolean Exists(User user) {}
}
```

## Sử dụng Domain Service

Trong thực tế, từ `service` được dùng với nhiều ý nghĩa khác nhau. Nhưng trong phát triển phần mềm, từ `service` có thể hiểu theo ý nghĩa là "thứ được tạo ra để phục vụ cho người dùng".

Trong DDD thì `service` được chia thành 2 nhóm

- Nhóm 1: service phục vụ cho domain.
- Nhóm 2: service phục vụ cho ứng dụng.

## Điều gì sẽ xảy ra khi lạm dụng Domain Service

Nếu ta viết toàn bộ các phương thức vào trong `Domain Service` thì class `UserService` sẽ trông như sau:

```js
class UserService {
  changeName(User user, UserName name) {
    user.name = name;
  }
}
```

Nếu viết toàn bộ các phương thức vào trong `Domain Service` thì `Entity` sẽ chỉ còn `setter` và `getter`. Khi đó nếu đọc code của Entity sẽ khá khó nắm bắt các phương thức của entity cũng như khó nắm bắt được các `Domain Rules`. Lúc này entity chỉ có một nhiệm vụ duy nhất đó là `lưu trữ dữ liệu`.

Khi đó chiến lược đóng gói `Dữ liệu` và `Phương thức` vào trong Object sẽ hoàn toàn bị phá sản.

Lúc này phương thức `changeName` nên được viết vào `UserEntity` class.

**Cố gắng tránh lạm dụng Domain Service:**

Nếu ban đầu vẫn còn phân vân về việc nên viết vào `Entity` hoặc `Value-Object` hay là viết vào `Domain Service` thì hãy viết vào `Entity` hoặc `Value-Object`.

> Hãy cố gắng tránh việc viết vào Domain Service nhiều nhất có thể

Việc lạm dụng `Domain Service` sẽ làm cho logic sẽ bị phân tán đi nhiều chỗ.

## Một ví dụ về Domain Service ứng dụng cho hệ thống logistic

Trong thực tế, hàng hoá từ kho được xuất ra không bao giờ đến trực tiếp tay người nhận ngay lập tức mà thường qua các kho trung gian

![Screen Shot 2021-09-25 at 21 14 54](https://user-images.githubusercontent.com/15076665/134771165-8f08259c-871d-4f8f-b43e-97b5ee2d1852.png)

Thử mô hình hoá quá trình vận chuyển này bằng code.

① **Định nghĩa các hành động của kho hàng**

**Kho hàng** là một thuật ngữ của domain, nên ta sẽ định nghĩa nó thành một `Entity` class

```js
class PhysicalDistributionBase {
  public Baggage ship (Baggage baggage) {}
  public void receive (Baggage baggage) {}
  public void transport(PhysicalDistributionBase to, Baggage baggage) {
    const shippedBaggage = ship(baggage);
    to.receive(shippedBaggage);
  }
}
```

Việc viết phương thức `transport` vào `entity` class trông có vẻ không ổn cho lắm.

② **Định nghĩa Domain Service**

Nếu định nghĩa `transport` method vào `domain service`, ta sẽ có service class như sau:

```js
class TransportService {
  public void transport (PhysicalDistributionBase from, PhysicalDistributionBase to, Baggage baggage) {
    const shippedBaggage = from.ship(baggage);
    to.receive(shippedBaggage);
  }
}
```

Kinh nghiệm ở đây đó là các method cảm giác "không vừa" với `entity` nên được cho sang `domain service`.

Một vài cách đặt tên cho Domain service class:

1. DomainName: VD: User
2. DomainName + Service: VD: UserService. Đây là cách làm phổ biến nhất, trong class này sẽ là tập hợp các hàm service, tuy nhiên nếu tách riêng từng hàm ra thành các class riêng rẽ thì có thể đặt tên class gần với chức năng của hàm (VD: `CheckDuplicateUserService`)
3. DomainName + DomainService: VD: UserDomainService.
