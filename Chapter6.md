# Chương 6: Thực thi usecase thông qua Application Service

## Application Service là gì ?

Là object thực thi `Usecase`.

Giả sử một hệ thống có chức năng: Đăng kí user, Thay đổi thông tin user. Lúc này 2 chức năng này sẽ được coi là 2 `usecase` và sẽ có 2 method tương ứng được định nghĩa trong `Application Service` để triển khai. Thực tế thì 2 method sẽ sẽ sử dụng `Domain Object` để triển khai.

## Xây dựng Usecase

Xem xét việc xây dựng chức năng User cho hệ thống. Với hệ thống thì chức năng User sẽ gồm những chức năng con sau đây:

- Đăng kí
- Xác nhận thông tin
- Thay đổi thông tin
- Rời khỏi hệ thống

1. **Chuẩn bị Domain Object**

```js
class User {
  private readonly UserName;
  private readonly UserId;
}

class UserName {
  string value;
}

class UserId {
  string id;
}
```

Ở đây `UserId`, `UserName` sẽ là các Value-Object class.

Ngoài ra cần phải có `Domain Service` để check duplicate user

```js
class CheckDuplicateUserService {
  checkDuplicateUser() {}
}
```

Cuối cùng là Repository để phục vụ cho việc lưu trữ, lấy dữ liệu ra từ storage.

```js
interface IUserRepository {
  find();
  save();
  delete();
}
```

Với tính năng đăng kí user, ta sẽ tạo ApplicationService như sau:

```js
class UserApplicationService {
  private readonly userRepository;
  private readonly userService

  register(UserName name) {
    const user = new User(name);

     if (this.userService.checkDuplicateUser(user)) throw new Exception; 

    this.userRepository.save(user);
  }
}
```

Với tính năng lấy thông tin user, ta sẽ có như sau:

```js
class UserApplicationService {
  private readonly userRepository;

  // omit register method

  find(UserId id) {
    const user = this.userRepository.find(id);

    return user;
  }
}
```

Ở đây giá trị trả về sẽ là `Domain Object`. Vấn đề không nằm ở việc domain object được trả về mà là cách domain object được sử dụng phía client.

```js
class Client {
  main() {
    const user = UserApplicationService.find(id);

    user.changeName(new UserName("newName"));
  }
}
```

Về bản chất, việc sử dụng các phương thức của `Domain Object` chỉ được cho phép tại `Application Service`. Nếu vượt qua quy tắc này thì các đoạn code có cùng chức năng sẽ bị rải rác ở nhiều chỗ. Ngoài ra còn một vấn đề khác đó là "việc phụ thuộc vào domain object", khi domain rules thay đổi thì domain object cũng sẽ phải thay đổi theo và khi đó chắc chắn sẽ phải chỉnh lại code ở nhiều chỗ.

Để tránh những rủi ro như trên, có một luồng suy nghĩ cho rằng:

> Không nên công khai Domain Object

Mà thay vào đó, sẽ sử dụng DTO (Data Transfer Object) để trả về phía client (DTO sẽ chỉ chứa data mà không chứa bất kì một phương thức nào).

```js
class UserDto {
  constructor(UserId id, UserName name) {
    this.id = id;
    this.name = name;
  }
}
```

Khi đó `Application Service` sẽ như sau:

```js
class UserApplicationService {
  private readonly userRepository;

  // omit register method

  find(UserId id) {
    const user = this.userRepository.find(id);
    const userDto = new UserDto(user.id, user.name);

    return userDto;
  }
}
```

Ở constructor của UserDto, nếu để các params bị tách lẻ như vậy, khi có sự thay đổi (VD: thêm email property chẳng hạn) thì ngoài việc sửa constructor thì ta cũng cần phải sửa ở tất cả các chỗ có sử dụng UserDto. Vậy nên thay vì để các params bị tách lẻ, ta sẽ túm gọn lại trong một object User (Domain Object).

## Tạo usecase thay đổi thông tin người dùng

Chức năng thay đổi thông tin người dùng có thể thay đổi theo thời gian khi thêm, hoặc bớt các thuộc tính muốn thay đổi, để tránh việc phải thay đổi code ở nhiều chỗ, ta có thể sử dụng `Command Object`

```JS
class UserUpdateCommand {
  constructor(id) {
    this.id = id;
  }

  // idGetter
  // nameGetter, nameSetter
  // emailGetter, emailSetter
}

class UserUpdateCommand {
  constructor(id, name, email) {
    this.id = id;
    this.name = name;
    this.email = email;
  }

  // idGetter
  // nameGetter
  // emailGetter
}
```

Method update của ApplicationService sẽ như sau:

```JS
class UserApplicationService {
  update(UserCommand command) {
    const user = userRepository.find(new UserId(command.id));

    if (command.name) {
      user.changeName(new UserName(command.name);
    }

    if (command.email) {
      user.changeEmail(new UserEmail(command.email));
    }

    userRepository.save(user);
  }
}
```

## Rò rỉ Domain Rule

Trong `Application Service` thì không nên có sự xuất hiện của `domain rules`.

Với ví dụ về `Đăng kí User` hoặc `Cập nhật thông tin User`, nếu ta tiến hành check duplicate user ở cả 2 usecases này thì trong trường hợp domain rule thay đổi (không trùng tên -> không trùng email) thì 2 usecases này cũng phải sửa đổi theo.

## Mức độ gắn kết với Application Service

Mức độ gắn kết ở đây được hiểu là phạm vi mà module có thể bao quát, tập trung được.

Để đo mức độ gắn kết ta có "Lack of Cohesion in Methods - (LCOM)" . Được tính dựa theo tỉ lệ giữa: tổng số biến instance và số biến instance được các methods sử dụng.

Cùng lấy ví dụ về "mức độ gắn kết cao" và "mức độ gắn kết thấp".

```TS
class LowCohesion {
  private number v1;
  private number v2;
  private number v3;
  private number v4;

  number methodA() {
    return v1 + v2;
  }

  number methodB() {
    return v3 + v4;
  }
}
```

Ở class `LowCohesion` thì methodA chỉ sử dụng `v1`, `v2` chứ không sử dụng `v3`, `v4`. Điều tương tự cũng xảy ra với `methodB`.

Nếu tách thành 2 class như sau:

```TS
class HighCohesionA {
  private number v1;
  private number v2;

  number methodA() {
    return v1 + v2;
  }
}

class HighCohesionB {
  private number v3;
  private number v4;

  number methodB() {
    return v3 + v4;
  }
}
```

Mỗi method của class đều sử dụng toàn bộ các thuộc tính của class.

**ApplicationService với mức độ gắn kết thấp:**

Với ví du về `UserApplication` ở các phần trước, ta thấy chỉ có `register` method là sử dụng cả `userRepository` và  `userService` còn `delete` method thì chỉ sử dụng `userRepository` vậy nên theo quan điểm về mức độ gắn kết thì cách viết này cho mức độ gắn kết thấp. Ta có thể tách thành 2 class  `UserRegisterService` và `UserDeleteService`

Do tách class nên để tập hợp chúng lại, ta cần sử dụng `package`

VD:

- Application.User.UserRegisterService
- Application.User.UserDeleteService

![Screen Shot 2021-10-04 at 8 28 07](https://user-images.githubusercontent.com/15076665/135775490-6559afdf-5308-4876-b0fb-d0145f6cc51b.png)

Khi mọi xử lí liên quan đến user đều có thể được tìm thấy trong folder `User` thì sẽ rất dễ dàng cho dev sau này.

## Interface của Application Service

Việc sử dụng interface sẽ đảm bảo tính uyển chuyển cho Application Service. Phía client thay vì gọi trực tiếp Application Service thì sẽ gọi thông qua Interface.

Hơn nữa nó còn giúp tiết kiệm thời gian cho dev phía client. Khi Application Service chưa hoàn thiện, thay vì chờ đợi thì phía client có thể tạo ra các `class Mock` để mô phỏng lại phía service do phía client chỉ gọi đến interface.

```TS
class MockUserRegisterService implements IUserRegisterService {
  void handle() {}
}
```

Do chỉ dừng ở mức mô phỏng service nên phía client hoàn toàn có thể test được cả trường hợp phía service phát sinh ra `Exception`

```TS
class MockUserRegisterService implements IUserRegisterService {
  void handle() {
    throw new Exception();
  }
}
```

![Screen Shot 2021-10-10 at 20 45 40](https://user-images.githubusercontent.com/15076665/136694148-8b545f1b-a670-49c3-ae8e-ff5e266bf53f.png)

## Service là gì ?

Hiểu nôm na thì Service là

> Một thứ được sinh ra để phục vụ  người dùng

Domain Service là object thể hiện nghiệp vụ của một lĩnh vực nào đó.
Application Service được tạo ra để giải quyết một vấn đề nào đó của người dùng.

VD:
Các tính năng như UserRegister hay UserWithdraw không hề thuộc về một domain nào cả, nó thuộc về ứng dụng nên đó là những Application Services.

**Service không hề có trạng thái:**

Ví dụ với `UserApplicationService`

```TS
 class UserApplicationService {
  private readonly userRepository: IUserRepository;
}
```

Rõ ràng `UserApplicationService` có "trạng thái" là `userRepository` nhưng `userRepository` không hề thay đổi trực tiếp hành động của service.

Ngược lại ở class dưới đây:

```TS
class UserApplicationService {
  private boolean sendMail;

  register() {
    if (sendMail) {
      sendEmail();
    }
  }
}
```

Biến `sendMail` có ảnh hưởng trực tiếp đến việc gửi email hay không của register service.

Việc có "trạng thái" này sẽ làm cho việc sử dụng service sẽ trở nên phức tạp hơn khi cần phải quan tâm đến "trạng thái" để biết được cách gọi hàm, sử dụng service sao cho phù hợp.
