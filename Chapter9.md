# Chương 9: Factory - nơi tiến hành các xử lí sinh phức tạp

## Mục tiêu chính của factory

Là một object để xử lí việc tạo ra các object có trong tầng domain.

Factory cũng là nơi bắt đầu vòng đời của một object.

## Sinh ID tại factory

Sinh ID có thể thực hiện theo nhiều cách, nhưng tiêu biểu vẫn là hai cách sau:

- Sử dụng thư viện có sẵn
- Thực thi increment dựa theo id trong DB

Nếu tiến hành increment dựa theo id trong DB thì trong constructor của entity class ta cần truy xuất ID lớn nhất ra từ DB rồi tăng 1 lên. Điều này vi phạm quy tắc "quan hệ phụ thuộc ngược" (module cấp cao không phụ thuộc vào module cấp thấp).

```ts
class UserEntity {
  constructor(UserId userId, UserName userName) {
    if (userId) {
      updateUser(userId, userName);
    } else {
      createUser(userName);
    }
  }
}
```

Ngoài việc vi phạm quy tắc của "quan hệ phụ thuộc ngược", trong class entity ta còn cần phải tách thành 2 constructor khác nhau (một hàm dùng cho tạo Entity mới, một hàm dùng cho update Entity hiện có).

Vậy nên ta có thể tiến hành sinh ID tại Factory

```ts
interface IUserFactory {
  create(userName: UserName);
}
```

Ngoài ra thì class `UserApplicationService` cũng có thể sử dụng Factory

```ts
class UserApplicationService {
  private readonly IUserFactory userFactory;
  private readonly IUserRepository userRepository;
  private readonly UserService userService;

  register(userName: string) {
    const userName = new UserName(userName);

    const userEntity = userFactory.create(userName);
  }
}
```

## Sử dụng tính năng tự sinh ID

Nếu sử dụng tính năng tự sinh ID có sẵn của DB thì entity sau khi tạo ra cho đến lúc được lưu vào DB sẽ không có ID (đây là một điều khá kì cục vì ID chính là yếu tố cần thiết để phân biệt giữa các entities với nhau).

## Sử dụng method sinh ID tại repository

```ts
interface IUserRepository {
  find(UserId id): UserEntity;
  save(UserEntity userEntity): void;
  nextID(): UserId;
}
```

Khi đó việc sinh ra ID mới sẽ như sau:

```ts
const userEntity = new UserEntity(userRepository.nextID());
```

Khi đó sẽ không có tình trạng entity không có ID nữa.

Tuy nhiên trong repository ngoài việc lưu, chỉnh sửa dữ liệu trong DB, ta còn gọi thêm một API ngoài để sinh ID mới thì lúc này repository sẽ phụ thuộc vào quá nhiều công nghệ. Vậy nên Repository chỉ nên thực hiện hai nhiệm vụ:

- Lấy dữ liệu
- Lưu, sửa dữ liệu

Việc thêm cả phần sinh ID vào sẽ khiến cho Repository phải "kham" nhiều nhiệm vụ hơn mức cần thiết.

## Đóng gói các xử lí tạo object phức tạp

Quá trình init sẽ được giao cho constructor, với những constructor không quá phức tạp ta có thể giữ nguyên nó. Với trường hợp constructor phức tạp thì nên tách ra sử dụng Factory.
