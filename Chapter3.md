# Chương 3: Entity - Object có vòng đời

## Entity là gì ?

Về bản chất nó là **Domain Object** thực thi **Domain Model**.

Ở chương nói về value-object, ta thấy value-object cũng là **Domain Object**, vậy **value-object** và **Entity** khác nhau ở điểm nào, câu trả lời đó là **identity**.

Lấy ví dụ với con người: mỗi người đều có tên, tuổi, chiêu cao, cân nặng, ... Sau một năm tuổi, chiều cao, cân nặng đều có sự thay đổi nhưng người nào vẫn là người đấy -> Đây chính là nhờ **identity**

Trong phần mềm, có những object không thể phân biệt bằng thuộc tính (VD: người dùng hệ thống - khi thay đổi thông tin cá nhân thì người dùng đó vẫn là người dùng đó -> phân biệt user bằng **identity**)

## Tính chất của Entity

Entities là các **Domain Object** được phân biệt bằng **identity** còn **Value-Ọbject** sẽ được phân biệt bằng **các thuộc tính**

VD: Name Value Object sẽ được phân biệt bằng thuộc tính **firstName** và **lastName**

Các tính chất của entity:

- Có thể thay đổi

Khác với value-object, entity có thể thay đổi, ví dụ như "tuổi" và "chiều cao" của một người có thể thay đổi.

Xét ví dụ sau:

```js
class User {
  private string name;

  constructor(string name) {
    this.name = name;
  }
}
```

Hiện tại thì không thể thay đổi "name" của user.

```js
class User {
  private string name;

  constructor(string name) {
    changeName(name);
  }

  changeName(string name) {
    this.name = name;
  }
}
```

Với method `changeName` ta hoàn toàn có thể thay đổi giá trị của name. Khác với "value-object" chỉ có thể "thay đổi" giá trị thông qua việc tạo và gán instance mới thì entity hoàn toàn có thể được thay đổi trực tiếp.

Tuy nhiên ta nên hạn chế việc thay đổi trực tiếp thuộc tính của entity, chỉ thay đổi những thuộc tính cần thay đổi.

> Với các trường hợp dữ liệu bị lỗi thì ngoài cách throw ra Exception trên server thì nên kiểm tra trước ở phía client để tránh trường hợp server gặp lỗi.

- Dù cùng thuộc tính nhưng khác nhau
Với ví dụ về tên người, các value-object có `firstName` và `lastName` trùng nhau thì sẽ được xem như là một object, còn ở entity thì không hề có chuyện đó. Ta có thể thêm vào `class User` ở phía trên **identity** là `UserId`.

```js
class UserId {
  private string value;

  constructor(string value) {
    this.value = value;
  }
}

class User {
  private string name;
  private UserId id;

  constructor(UserId id, string name) {
    this.id = id;
    this.name = name;
  }
}
```

- Được phân biệt bởi identity
Với class User như ở trên, ta có thể viết method `Equal` thực hiện phép so sánh các id để xem 2 object có khác nhau hay không.

Trong khi value-object cần so sánh mọi thuộc tính để phân biệt 2 object thì entity chỉ cần so sánh `identity` là đủ.

## Sự liên quan giữa các tiêu chuẩn của entity với vòng đời

Với một hệ thống có User, thì từ khi User được tạo cho đến khi người dùng không sử dụng hệ thống nữa thì User sẽ bị xoá đi - Đây là vòng đời của User.

## Model có thể trở thành Entity hay Value Object

Việc sử dụng value-object hay entity là hoàn toàn phụ thuộc vào ngữ cảnh hiện có. Lấy ví dụ:

- Bánh xe đối với ô tô là thứ hoàn toàn có thể thay thế, vậy nên việc phân biệt giữa các bánh xe là điều không cần thiết, trong tình huống này sử dụng `value-object` sẽ hợp lí hơn.
- Bánh xe trong xưởng sản xuất cần một định danh để biết được bánh xe được sản xuất khi nào, thế nên sử dụng `entity` sẽ hợp lí hơn.

Có 2 ưu điểm cơ bản như sau:

- Tăng tính document cho code.

Với các lập trình viên bảo trì hệ thống hoặc tham gia giữa chừng vào dự án thì việc có đầy đủ kiến thức về dự án, hệ thống hiện tại là điều không thể. Dù có bản thiết kế nhưng thiết kế chỉ bao quát được những điều kiện to, còn những điều kiện nhỏ, chi tiết thì chỉ có đọc code mới hiểu được. Ngoài ra có những trường hợp hệ thống vận hành không khớp với thiết kế 100% thì việc đọc code lại trở nên quan trọng hơn bao giờ hết.

Giả sử với class UserName như sau:

```js
class UserName {
  public string name { get; set; }
}
```

Thì việc hiểu ý nghĩa của nó khó hơn rất nhiều so với việc có một class UserName như thế này:

```js
class UserName {
  private string name;

  constructor(string name) {
    if (name.length < 3) throw Exception;

    this.name = name;
  }
}
```

Sẽ dễ dàng để người đọc code hiểu được điều kiện cần có của UserName là gì.
Mọi domain rules sẽ được viết vào các `Domain Object`. Nếu không thì để hiểu rõ được các domain rules thì chỉ có một cách duy nhất đó là **ĐỌC LẠI** toàn bộ code hiện có (điều này đối với người có kinh nghiệm cũng không phải một chuyện dễ dàng gì).

- Có thể thay đổi code tuỳ theo domain một cách dễ dàng.
Ví dụ nếu domain rules thay đổi từ việc UserName ít nhất có 3 chữ cái -> 6 chữ cái thì lúc này chỉ cần thay đổi ở class UserName là đủ, ngược lại nếu việc kiểm tra điều kiện trên được phân tán ở nhiều chỗ thì việc sửa code sẽ rất khó khăn.

Trên thực tế, việc domain rules thay đổi là rất hay xảy ra, nên việc hệ thống có thể phản ánh ngay lập tức sự thay đổi đó là một điều rất tuyệt vời.
