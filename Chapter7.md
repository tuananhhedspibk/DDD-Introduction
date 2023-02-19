# Chương 7: Điều khiển các mối quan hệ phụ thuộc

## Sự phụ thuộc trong kĩ thuật

Việc sử dụng object sẽ phát sinh một vấn đề đó là **quan hệ phụ thuộc**. Khi các objects phụ thuộc vào nhau quá nhiều, chỉ cần một thay đổi nhỏ ở một object cũng đủ để gây ảnh hưởng lên một phạm vi rộng.

Trên thực tế, rất khó để tránh được **quan hệ phụ thuộc** này, thế nên thay vì "tránh" nó hãy tìm cách "kiểm soát" nó.

## Thế nào là phụ thuộc

Như ví dụ dưới đây, là sự phụ thuộc của ObjectA vào ObjectB

```ts
class ObjectA {
  private ObjectB objectB;
}
```

Ngoài ra còn có thể thấy quan hệ phụ thuộc còn xuất hiện giữa class và interface

```ts
interface IUserRepository {
  find();
}

class UserRepository implements IUserRepository {
  find() {}
}
```

Khi interface không tồn tại thì sẽ gây ra lỗi khi compile code do `UserRepository` class triển khai interface `IUserRepository`.

Lấy một ví dụ khác như sau:

```ts
class UserApplicationService {
  private readonly UserRepository userRepository;

  register() {
    //....
    this.userRepository.find();
   // ....
  }
}
```

Quan hệ giữa `UserApplicationService` và `UserRepository` đúng là quan hệ phụ thuộc nhưng nó cũng phụ thuộc luôn về mặt kĩ thuật khi `UserApplicationService` sẽ cần phải quan tâm đến loại DB mà `UserRepository` sử dụng (RDB hay NoSQL). Nếu sử dụng interface `IUserRepository` thì vấn đề phụ thuộc đến hạ tầng kĩ thuật sẽ được giải quyết (có thể thay đổi từ RDB sang NoSQL hoặc ngược lại mà không ảnh hưởng gì đến `UserApplicationService`). Từ đó sẽ tách riêng được business logic với phần implementation.

## Nguyên tắc của quan hệ phụ thuộc ngược

**Dependency Inversion Principle:**

- Các modules cấp cao không nên phụ thuộc vào module cấp thấp. Cả hai loại modules đều nên phụ thuộc vào các yếu tố trừu tượng
- Các yếu tố trừu tượng không nên phụ thuộc vào việc triển khai, việc triển khai nên phụ thuộc vào các yếu tố trừu tượng.

Nguyên tắc của quan hệ phụ thuộc ngược:

- Tăng tình uyển chuyển, mềm dẻo cho phần mềm
- Đảm bảo cho business logic không phụ thuộc vào các yếu tố công nghệ

## Hãy phụ thuộc vào các yếu tố trừu tượng

Trong chương trình cũng có sự phân cấp level

- Level cấp thấp: gần với các yếu tố công nghệ, xử lí mang tính chất cụ thể
- Level cấp cao: gần với người dùng, con người, mang tính trừu tượng cao

Các khái niệm module cấp cao, cấp thấp trong các nguyên tắc của "quan hệ phụ thuộc ngược" cũng được định nghĩa tương tự.

Như ví dụ về `UserApplicationService` sử dụng `UserRepository` thì `UserRepository` là cấp thấp, `UserApplicationService` là cấp cao.

Nếu `UserApplicationService` sử dụng `UserRepository` thay vì interface của nó thì `UserApplicationService` sẽ phụ thuộc vào công nghệ sử dụng phía `UserRepository` tức là module cấp cao sẽ phụ thuộc vào module cấp thấp. Điều này vi phạm quy tắc của "quan hệ phụ thuộc ngược"

## Kiểm soát yếu tố trừu tượng

Phát triển phần mềm theo cách truyền thống sẽ làm cho các module cấp cao phụ thuộc vào module cấp thấp (yếu tố trừu tượng sẽ phụ thuộc vào yếu tố cụ thể).

Điều này vô tình dẫn đến hệ quả, nếu module cấp thấp có sự thay đổi, nó sẽ tạo ra ảnh hưởng lớn đến các modules cấp cao.

Các domain rules thương nằm ở level cấp cao, nếu level cấp cao phụ thuộc vào level cấp thấp thì khi level cấp thấp thay đổi (thay đổi DB) sẽ gây ảnh hưởng đến domain rules. Đây là một điều rất kì cục và nên tránh.

> Module cấp cao nên là TRUNG TÂM của phần mềm thay vì các module cấp thấp

Các modules cấp cao chỉ nên sử dụng các modules cấp thấp với tư cách của một client. Tức là thuần tuý đưa ra các lời gọi/ yêu cầu tới cho modules cấp thấp.

## Điều khiển quan hệ phụ thuộc

Ta xét ví dụ về `UserApplicationService`.

```ts
class UserApplicationService {
  private readonly IUserRepository userRepository;

  constructor() {
    userRepository = new InMemoryUserRepository();
  }
}
```

`UserRepository` sẽ được dùng cho production còn `InMemoryUserRepository` sẽ được dùng cho develop. Nhưng nếu ứng dụng có vấn đề, ngoài việc phải tái hiện lại được bug, ta cần chuẩn bị dữ liệu. Hơn thế nữa phải quay về sử dụng `InMemoryUserRepository` - đây là một việc khá phiền phức.

Để giải quyết vấn đề trên, ta sử dụng hai pattern:

- Service Locator
- IoC Container

## Service Locator

Tương tự như ví dụ `UserApplicationService` ở trên, ta sử dụng một object có tên là `ServiceLocator` để lấy về instance phù hợp.

```ts
class UserApplicationService {
  private readonly IUserRepository userRepository;

  constructor() {
    this.userRepository  = ServiceLocator.Resolve<IUserRepository>();
  }
}
```

Trước đấy ở startup script, ta cần đăng kí sẽ trả về instance loại nào

```ts
ServiceLocator.Register<IUserRepository, InMemoryRepository>();
```

Nếu cho môi trường production, ta sẽ đăng kí như sau:

```ts
ServiceLocator.Register<IUserRepository, UserRepository>();
```

Khi đó nếu switch giữa môi trường `production` vs `development` ta chỉ cần thay đổi ở `startup script` là đủ.

![Screen Shot 2021-10-23 at 17 27 33](https://user-images.githubusercontent.com/15076665/138549115-a87cafc8-b89f-4835-b40e-92bb4a6f7007.png)

Với việc sử dụng `Service Locator` như trên ta có thể thiết lập cho việc switch giữa các môi trường với nhau.

![Screen Shot 2021-10-23 at 17 29 17](https://user-images.githubusercontent.com/15076665/138549159-593581fc-c72b-43db-bae2-a1111fd8634a.png)

## Service Locator cũng là anti-pattern

Ngoài ưu điểm như đã nói, Service Locator cũng có những nhược điểm như sau:

- Khá khó thấy được "quan hệ phụ thuộc" khi nhìn từ bên ngoài.

```ts
class UserApplicationService {
  constructor();
  register();
}
```

Đây chính là `UserApplicationService` khi nhìn từ bên ngoài do ta đã sử dụng `Service Locator` để giải quyết vấn đề phụ thuộc giữa instance với môi trường. Nếu như vậy, khi đọc code ta chỉ có thể hiểu rằng instance được tạo từ class này sẽ chỉ có method register, thế nhưng có thể việc thực thi method register cũng có thể gặp lỗi vì ta cần đăng kí với ServiceLocator về instance sẽ trả về (UserRepository hay InMemoryUserRepository) -> Nếu không đọc kĩ code thì sẽ không thể hiểu nguyên nhân gây ra lỗi.

- Khó để duy trì test
Khi sử dụng `ServiceLocator` khi test ta cần đăng kí như sau:

```ts
ServiceLocator.Register<IUserRepository, InMemoryUserRepository>();
const userApplicationService = UserApplicationService();
```

Nếu ta thêm vào `IFooRepository`

```ts
class UserApplicationService {
  private readonly IUserRepository userRepository;
  private readonly IFooRepository fooRepository;

  constructor() {
    this.userRepository = ServiceLocator.resolve<IUserRepository>();
    this.fooRepository = ServiceLocator.resolve<IFooRepository>();
  }
}
```

Khi đó nếu chạy test mà không đăng kí thêm `IFooRepository` thì sẽ có lỗi xảy ra (nhưng nguyên nhân lại không nằm ở test).

## IoC Container Pattern

Chúng ta cần biết về `Dependency Injection`. Lấy ví dụ như sau

```ts
const userRepository = new InMemoryUserRepository();
const userApplicationService = new UserApplicationService(userRepository);
```

Đây là ví dụ điển hình về Dependency Injection. Truyền object phụ thuộc vào constructor (cũng có thể gọi là Constructor Injection)

Tuy nhiên nếu thêm một "quan hệ phụ thuộc" nữa thì lúc chạy test sẽ dẫn đến compile error.

Ta có thể sử dụng `IoC Container` pattern để tạo ra `UserApplicationService` instance.

```ts
// IoC Container
const serviceCollection = new ServiceCollection();
serviceCollection.addTransient<IUserRepository, InMemoryUserRepository>();
serviceCollection.addTransient<UserApplicationService>();

const provider = serviceCollection.buildServiceProvider();
const userApplicationService = provider.getService<UserApplicationService>
```

Quá trình setup `IoC Container` cũng được tiến hành trong startup script - tương tự như `Service Locator`.
