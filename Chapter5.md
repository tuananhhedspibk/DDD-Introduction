# Chương 5: Sử dụng repository cho các xử lí liên quan đến dữ liệu

## Repository là gì ?

Là object thực hiện việc trừu tượng hoá các xử lí như "lưu dữ liệu" hoặc "chỉnh sửa dữ liệu" trong database.

Trong thực tế, chúng ta sẽ không trực tiếp lưu dữ liệu vào trong DB mà sẽ thông qua `Instance của Repository`.

![Screen Shot 2021-09-26 at 15 16 23](https://user-images.githubusercontent.com/15076665/134795964-cde1b35a-3ba0-4c99-9785-6333d7bd6852.png)

Repository khác so với Domain Object mà chúng ta đã từng biết. Repository có chức năng làm nổi bật vai trò của `Domain Object` đối với `Domain Model`.

## Nhiệm vụ của Repository

Repository có chức năng là `lưu trữ` và `tái cấu trúc` `domain object`. Việc lưu trữ này không giới hạn ở `Relational Database` mà cũng có thể là:

- Lưu object vào file
- Lưu object vào NoSQL

Ngoài 2 chức năng trên Repository còn đảm bảo là có thể lấy và đưa ra dữ liệu từ nơi lưu trữ.

Với ví dụ về User như ở các chương trước, ta có thể vận dụng repository như sau:

```JS
class Program {
  private IUserRepository userRepository;
  
  constructor(IUserRepository userRepository) {
    this.userRepository = userRepository;
  }

  void createUser(UserName name) {
    const newUser = new User(name);

    const userService = new UserService(userRepository);

    if (userService.exist(newUser)) throw Exception;

    userRepository.save(newUser);
  }
}
```

Việc lưu User object sẽ được uỷ nhiệm lại cho `UserRepository` nên việc lưu vào RDB hay NoSQL hay file là không quan trọng đối với domain.

Lúc này Domain Service `UserService` sẽ như sau:

```JS
class UserService {
  private IUserRepository userRepository;

  constructor(IUserRepository userRepository) {
    this.userRepository = userRepository;
  }

  exists(User user) {
    const found = userRepository.find(user.name);

    return found !== null;
  }
}
```

Ta có thể thấy thông qua object trừu tượng là `userRepository` mà các thao tác như lưu trữ, sửa đổi, tìm kiếm dữ liệu có thể thực hiện một cách dễ dàng, tách biệt hoàn toàn so với `business logic`.

## Repository Interface

Repository được định nghĩa dưới dạng interface (trừu tượng hoá)

```JS
interface IUserRepository {
  User find(UserName name);
  void save(User user);
}
```

Ta cũng có thể xem xét đến việc đưa method `exist` vào Repository nhưng việc check duplicate này lại gần với domain mà chức năng của Repository chỉ là lưu trữ dữ liệu vào DB nên việc đưa `exist` method vào Repository là không hợp lí.

Không nên đưa các xử lí liên quan đến tầng infrastructure vào trong domain service.

## Tạo Repository sử dụng SQL

Với việc sử dụng repository thì business logic hoàn toàn không phụ thuộc vào kĩ thuật sử dụng để lấy, lưu dữ liệu ở tầng `infrastructure`.

Có thể trong usecase hoặc domain-service sử dụng `IRepository` nhưng thực chất các lời gọi hàm này đều được chuyển tiếp đến `class Repository`.

## Xây dựng Repository sử dụng ORM

Không sử dụng trực tiếp câu lệnh SQL.

ORM cần một model class để sử dụng như là Entity (model này thường sẽ giống với cấu trúc các bảng của DB).
Model class này sẽ được định nghĩa ở tầng `infrastructure`.

## Định nghĩa các phương thức cho Repository

Tránh việc viết quá nhiều các phương thức `update` ví dụ như sau:

```JS
class UserRepository implements IUserRepository {
  void updateName (UserName name) {}
  void updateGender (UserGender gender) {}
  void updateEmail (UserEmail email) {}
}
```

Thay vào đó chỉ sử dụng một phương thức `update` duy nhất. Ngoài ra ta không nên định nghĩa phương thức `create object` trong `Repository`, công việc này sẽ do `Factory` đảm nhận.

Theo như vòng đời của object, ta cần phương thức để huỷ object khi không cần thiết nữa. Việc này cũng nên được thực hiện trong `Repository`.

Phương thức liên quan đến truy vấn dữ liệu hay sử dụng nhất là `tìm kiếm dữ liệu bằng ID`

```JS
 class UserRepository implements IUserRepository {
  User find (UserID id) {}
}
```

Tuy nhiên cũng có những trường hợp cần đến phương thức `findAll` để lấy ra toàn bộ dữ liệu có trong DB. Nhưng nếu xét đến hiệu năng thì phương thức này có hiệu năng khá tồi nên hãy chú trọng việc định nghĩa các phương thức tìm kiếm theo điều kiện.

```JS
 class UserRepository implements IUserRepository {
  User findById (UserID id) {}
  User findByName (UserName name) {}
}
```
