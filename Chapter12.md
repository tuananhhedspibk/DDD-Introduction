# Chương 12: Sử dụng Aggregate để bảo vệ domain business rules

## Aggregate bảo vệ domain rule

Trong lập trình hướng đối tượng, chúng ta thường thấy việc một Object là tập hợp của nhiều objects con khác.

Với những object như vậy, luôn có một quy tắc về tính bất biến (Invariant - duy trì tính nhất quán về mặt dữ liệu cũng như các quan hệ phụ thuộc khác khi cập nhật object).

Tính bất biến (Invariant) không cho phép các thao tác với dữ liệu của object được tiến hành một cách không có giới hạn. Các thao tác với object cần có một TRẬT TỰ nhất định.

Aggregate (kết tập) cũng là một cách để duy trì tính bất biến (Invariant) này.

Aggregate có 2 thứ cần phải chú ý:

- Phạm vi: Aggregate sẽ bao hàm những gì ở bên trong.
- Root: Object chứa aggregate.

Mọi thao tác từ bên ngoài tác động vào Aggregate sẽ phải thông qua Object root này. Các object bên trong Aggregate sẽ không được đưa ra ngoài nên sẽ bảo toàn được tính bất biến.

## Cấu trúc cơ bản của Aggregate

![Screen Shot 2021-11-21 at 11 52 30](https://user-images.githubusercontent.com/15076665/142747543-30bfb9d9-a962-47dd-88ea-f8d02d0c47f8.png)

Hình phía trên là một ví dụ tiêu biểu về Aggregate.

Phía bên ngoài sẽ không được phép thao tác trực tiếp với các object bên trong Aggregate, mọi thao tác sẽ thông qua Aggregate root.

Ví dụ ở trên, khi muốn thay đổi `userName`.

```ts
const userName = new UserName("test");

// NG
user.name = userName;

// OK
user.changeName(userName);
```

Bản chất thì 2 cách làm trên đều cho ra cùng một kết quả, nhưng việc thay đổi thông qua method sẽ giúp ta có thể kiểm tra giá trị đầu vào trước khi tiến hành thay đổi (việc thay đổi trực tiếp sẽ dẫn đến việc gán giá trị `null` cho thuộc tính). Đồng nghĩa với việc tránh những dữ liệu không chính xác tồn tại.

![Screen Shot 2021-11-21 at 11 58 39](https://user-images.githubusercontent.com/15076665/142747700-c8536a17-f4b6-43ae-9af7-43524ee50a56.png)

Aggregate User tồn tại như một thành viên bên trong Aggregate Circle. Thế nhưng Aggregate Circle lại không bao hàm Aggregate User nên việc thay đổi thông tin của Aggregate User sẽ không được thông qua Aggregate Circle, ở Aggregate Circle sẽ chỉ thêm Aggregate User như một thành viên trong thuộc tính `member` của mình mà thôi.

```ts
circle.members.add(user);
```

Thế nhưng cách làm này lại vi phạm quy tắc của Aggregate. Vậy nên sau này việc thêm user vào `members` của circle sẽ được tiến hành như sau:

```ts
circle.addMember(user);
```

Nếu ta quy định `members` là private thì ngoài cách thông qua method `addMember` như trên thì không có cách nào khác nữa.
Hơn nữa nếu ta thêm quy định một circle chỉ có tối đa 30 members thì việc thêm user thông qua method sẽ đảm bảo cho quy tắc trên luôn được kiểm tra.

> Trong lập trình OOP, việc uỷ thác việc thay đổi dữ liệu của object cho chính object sẽ có lợi ích về tính trực quan cũng như đảm bảo tính bất biến và toàn vẹn của dữ liệu.

## Quy tắc demeter

Theo như quy tắc thì một class chỉ nên biết và tương tác với các classes khác ít nhất có thể. Vì bản thân việc để nhiều class gắn kết với nhau sẽ khiến cho code khó có thể maintain. Quy tắc demeter cũng quy định rằng các method chỉ nên gọi đến các methods của object thuộc phạm vi local.

> Tránh tình trạng: A.getObjectB().getObjectC().display()

① Method M thuộc Object O có thể gọi các methods bên trong O

```ts
class ObjectO {
  void otherMethod() {}

  void methodM() {
    otherMethod();
  }
}
```

② Method M có thể gọi các method của object P được truyền như tham số vào method M

```ts
class ObjectO1 {
  test() {}
}

class ObjectO2 {
  void methodM(ObjectO1 o1) {
    o1.test();
  }
}
```

③ Method M có thể gọi các objects được tạo bên trong M

```ts
class ObjectO {
  void methodM() {
    const p = new P();

    p.method();
  }
}
```

④ Method M có thể gọi đến các object là component của O.

```ts
class ObjectO1 {
  void test() {}
}

class ObjectO2 {
  ObjectO1 o1;

  void methodM() {
    o1.test();
  }
}
```

## Các quy tắc cơ bản khi thao tác với object

Về cơ bản phần này sẽ nói về việc ứng dụng quy tắc demeter trong thực tế như thế nào.

Ta lấy một ví dụ trong thực tế, khi điều khiển xe oto, ta không trực tiếp điều khiển bánh xe mà sẽ thông qua oto (root)

```ts
circle.members.add(member);
```

Việc gọi thao tác trên members như trên là vi phạm quy tắc demeter. Thay vào đó việc gọi như dưới đây là hoàn toàn phù hợp với quy tắc demeter.

```ts
circle.join(newMember);
```

Lấy một ví dụ khác:

```ts
if (circle.members.count > 30) {
  throw Exception();
}
```

Việc viết code như thế này là vi phạm quy tắc demeter, hơn nữa nếu sau này có nhiều chỗ cũng cần kiểm tra điều kiện này thì chỉ cần một sự thay đổi nhỏ (giới hạn trên từ 30 tăng lên thành 50) thì việc sửa code sẽ rất khó khăn.

```ts
class Circle {
  isFull() {
    return this.members.count > 30;
  }
}
```

Việc thu tất cả lại trong một method sẽ giúp việc sửa code sau này sẽ dễ thở hơn rất nhiều.

## Che dấu đi dữ liệu bên trong

> Dữ liệu nội bộ bên trong một object không nên được công khai ra bên ngoài

Tuy nhiên nếu không công khai toàn bộ thì việc lưu dữ liệu sẽ gặp phải khó khăn

```ts
class UserRepository implements IUserRepository {
  save(User user) {
    const userDataModel = new UserDataModel({ id: user.id, email: user.email });

    context.Users.add(userDataModel);
    context.save();
  }
}
```

Vậy nên chỉ cần công khai dữ liệu cho Repository sử dụng là đủ, việc công khai toàn bộ dữ liệu là hoàn toàn không cần thiết.

Ngoài ra còn một cách tiếp cận khác là thông qua `Notification`

```ts
interface IUserNotification {
  void id (UserId id);
  void name (UserName name);
}

public class UserDataModelBuilder implements IUserNotification {
  private UserId id;
  private UserName name;

  void id (UserId id) {
    this.id = id;
  }

  void name (UserName userName) {
    this.name = userName;
  }

  public UserDataModel build() {
    return new UserDataModel({ id, name });
  }
}

public class User {
  private readonly UserId id;
  private UserName name;

  public void notify (IUserNotification note) {
    note.id(this.id);
    note.name(this.name);
  }
}
```

class User sẽ nhận Notification object và truyền dữ liệu tương ứng vào. Cách làm này giúp ta không nhất thiết phải công khai dữ liệu nội bộ của object nhưng vẫn có thể truyền dữ liệu ra bên ngoài được.

```ts
class UserRepository {
  save (User user) {
    const userDataModelBuilder = new UserDataModelBuilder();
    user.notify(userDataModelBuilder);

    const userDataModel = userDataModelBuilder.build();

    // ...
  }
}
```

## Làm cách nào để phân chia Aggregate

Nên phân chia Aggregate như thế nào, ta hoàn toàn có thể phân chia dựa theo "ĐƠN VỊ".

Xét ví dụ dưới đây:
Ta có 2 aggregates là `User` và `Circle`. Với mối quan hệ như hình vẽ.

![Screen Shot 2022-01-01 at 17 59 52](https://user-images.githubusercontent.com/15076665/147847422-2ccb813a-a02a-4d01-8115-c099c782a9a6.png)

Về mặt nguyên tắc, mọi thay đổi của `Circle` chỉ được phép diễn ra trong nội bộ của `Circle`, ta cũng có điều tương tự với `User`. Thế nhưng giả sử sự thay đổi trong `Circle` kéo theo sự thay đổi của `User` như đoạn code sau:

```ts
class Circle {
  private members: User[];

  changeMemberName(UserId id, UserName name) {
    const target = members.filter(member => member.UserId === id);

    if (target) target.changeName(name);
  }
}
```

Vấn đề ở đây sẽ nằm ở repository, vì hiện tại repository của `Circle` không hề động chạm gì đến `User` nên nếu viết như trên ta sẽ phải sửa lại cả `Circle Repository`.

```ts
class CircleRepository {
  save(CircleEntity entity) {
    if (entity.user) {
      const user = entity.user;
      user.save();  
    }
  }
}
```

Từ đó nảy sinh một vấn đề đó là trùng lặp code với `User Repository`. Đây là một điều mà chúng ta luôn muốn chánh. Vậy nên hãy cố gắng gói gọn sự thay đổi của Aggregate trong phạm vi của chính nó để tránh việc phải thay đổi Repository.

## Kết hợp bằng ID

Có một cách mà `Circle` không nhất thiết phải lưu các instances `User` trong Aggregate của mình đó lả lưu trữ ID của các instances. Vì khi không lưu trực tiếp các instances ở bên trong Aggregate thì việc gọi đến method của `User Entity` là điều không thể.

```ts
class Circle {
  public members: UserId[];
}
```

Nếu làm thế này thì dù có công khai `members` property đi chăng nữa thì việc gọi đến method của `User Entity` là không thể. Khi đó nếu muốn gọi đến method của `User entity` thì chỉ có cách là truyền `UserId` cho `UserRepository` để lấy về `User` instance và từ đó sẽ gọi đến các methods của user.

Không những thế, việc làm này còn giúp tiết kiệm bộ nhớ. Ta thử lấy việc thay đổi tên của Circle làm ví dụ:

```ts
class CircleApplicationService {
  private readonly circleRepository: ICircleRepository;

  async update (CircleUpdateCommand command) {
    const id = new CircleId(command.id);
    const circle = await circleRepository.findById(id);
    if (!circle) throw new Exception;

    if (command.name) {
      const name = new CircleName(command.name);
      circle.changeName(name);

      if (circleService.exists(circle)) throw Exception;

      circleRepository.save(circle);
    }
  }
}
```

Do đoạn code trên chỉ thay đổi tên của Circle chứ không hề động chạm gì đến `User instance` nên nếu lưu các `User instances` vào trong Circle Aggregate thì sẽ dẫn đến lãng phí bộ nhớ.

## Độ lớn của Aggregate

Transaction sẽ lock dữ liệu lại, vậy nên khi quy mô của Aggregate càng lớn thì phạm vi lock sẽ lớn theo.
Nếu để cho Aggregate càng lớn thì khả năng thất bại của xử lí sẽ cao. Vậy nên, hãy giữ cho độ lớn của Aggregate nhỏ nhất có thể.

Ngoài ra việc sử dụng nhiều Aggregate trong cùng một transaction cũng là điều nên tránh. Nhiều Aggregate đồng nghĩa với việc phạm vi lock data sẽ càng rộng.

## Tránh sự không nhất quán trong từ ngữ

Điều kiện về members trong Circle đó là 「Nếu tính cả circle's owner thì số lượng members sẽ phải nhỏ hơn 30」. 30 là một con số cụ thể nhưng trong code thì giá trị này lại là 29.

```ts
class Circle {
  private members: User[];

  boolean isFull() {
    return this.members.length >= 29;
  }
}
```

Về logic thì đoạn code trên không sai, nhưng nó dễ gây hiểu nhầm cho người đọc code, đặc biệt là các dev bảo trì sau này. Vậy nên code của chúng ta nên tránh việc không nhất quán trong cách dùng từ (chỉ nội dung nghiệp vụ, logic).

Vậy nên ta có thể thêm method đếm số lượng members THỰC SỰ vào circle như sau:

```ts
class Circle {
  private members: User[];
  
  isFull() {
    return this.memberCounts() >= 30;
  }

  memberCounts() {
    return this.members.length + 1;
  }
}
```
