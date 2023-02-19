# Chương 2: Value-Object

## Value-Object là gì ?

Value-Object là object mang giá trị đặc trưng riêng của hệ thống (khác với các giá trị nguyên thuỷ). Khi value-object đứng riêng thì nó không có ý nghĩa gì cả, nó chỉ có ý nghĩa khi đính kèm với một entity.

VD: Với hệ thống quản lí nhân viên thì ID là một Value-Object. Khi ID này đứng riêng, nó hoàn toàn không có ý nghĩa gì, nó chỉ có ý nghĩa khi gắn liền với entity là nhân viên.

![Screen Shot 2021-07-04 at 12 23 06](https://user-images.githubusercontent.com/15076665/124372039-9e262c00-dcc2-11eb-9e04-dabc005413fd.png)

Value có 3 tính chất sau:

- Bất biến
- Có thể thay đổi (exchange)
- Tính bình đẳng (Equality)

### Tính bất biến của value

```typescript
let a = 1;
a.changeTo(2) // a: 1-> 2

"Hello".changeTo("HaHa") // NG
```

Ta có thể thay đổi giá trị của biến nhưng không thể thay đổi giá trị của "MỘT GIÁ TRỊ".

## Ưu và nhược điểm của việc không thay đổi trạng thái của object

**Ưu điểm:**

1. Tránh được những lỗi tiềm tàng vì tránh được việc trạng thái của object bị thay đổi một cách không mong muốn.
2. Thuận lợi hơn khi tiến hành xử lí song song, khi đó ta sẽ không phải ghi nhớ quá nhiều về việc thay đổi giá trị của object.
3. Tiết kiệm được tài nguyên bộ nhớ, vì ta có thể cache object do object không bị thay đổi trạng thái.

**Nhược điểm:**

1. Khi muốn thay đổi trạng thái hoặc giá trị của object ta phải tạo ra một instance mới của object -> khá tồi về mặt performance.

Lời khuyên ở đây là:

> Nếu vẫn còn sự phân vân thì hãy lựa chọn "Object không thay đổi trạng thái".

## Có thể trao đổi giá trị

Giá trị là không thể thay đổi nhưng nếu không thay đổi giá trị thì không thể phát triển được một phần mềm như mong muốn.

Ví dụ:

```js
var c = 1;
c = 2;
```

Thay vì hiểu theo hướng thay đổi giá trị, hãy nghĩ rằng ta đang thay thế giá trị của biến `c`. Tương tự vậy với value-object, ta không thể thay đổi giá trị của nó nhưng có thể thay thế giá trị cho nó.

```js
var fullName = new FullName("ABC", "XYZ")
fullName = new FullName("ABC", "DEF")
```

## So sánh dựa theo tính bình đẳng

Xét ví dụ

```JS
console.log(0 === 0);
console.log(1 === 0);
```

Đây là 2 ví dụ so sánh giá trị, việc so sánh ở đây sẽ được hiểu theo nghĩa `so sánh từng thuộc tính cấu thánh nên giá trị` chứ không phải là `so sánh trực tiếp giá trị với giá trị`, việc so sánh các value-object của hệ thống cũng tương tự như vậy.

```js
var nameA = new FullName("ABC", "XYZ");
var nameB = new FullName("ABC", "XYZ");

console.log(nameA.isEqual(nameB));
```

Chúng ta cũng có thể so sánh bằng cách "tự tay" so sánh từng thuộc tính một

```js
var nameA = new FullName("ABC", "XYZ");
var nameB = new FullName("ABC", "XYZ");

console.log(nameA.firstName === nameB.firstName);
console.log(nameA.lastName === nameB.lastName);
```

Cách này không sai nhưng nhìn khá "mất tự nhiên", nếu coi value-object của hệ thống giống như một loại "value" thì khi so sánh các giá trị nguyên thuỷ, code sẽ trông như sau:

```js
console.log(1.value === 0.value)
```

Khá hài hước khi lấy ra "value" của "value".

## So sánh bằng equal method vs So sánh thông qua thuộc tính

Việc so sánh thông qua equal method sẽ có ưu điểm là khi thêm các thuộc tính mới ta không cần thiết phải sửa lại code.

Ví dụ với Object FullName như sau:

```js
// Ban đầu chỉ có firstName và lastName

var result = nameA.firstName === nameB.firstName && nameA.lastName === nameB.lastName;

// Nếu có thêm middleName

var result = nameA.firstName === nameB.firstName
  && nameA.lastName === nameB.lastName
  && nameA.middleName === nameB.middleName;
```

Việc thay đổi này tuy dễ dàng nhưng nếu nó được lặp đi lặp lại nhiều lần cũng sẽ khiến cho dev cảm thấy mệt mỏi và có thể phát sinh nhầm lẫn.

Nếu sử dụng `equal method` ta chỉ cần thay đổi `equal method` là đủ.

```js
boolean equal(other) {
  return string.Equals(firstName, other.firstName)
    && string.Equals(lastName, other.lastName)
    && string.Equals(middleName, other.middleName);
}
```

Không chỉ `equal method` với các trường hợp khác ta cũng chỉ cần sửa ở một chỗ là đủ.

## Các tiêu chuẩn đối với Value-Object

Trong thực tế việc lựa chọn value-object phải dựa theo domain model.

## Value-object có hành động

Với value-object thì yếu tố quan trọng nhất đó là định nghĩa được một "hành động". Lấy ví dụ về "Money Value-Object".

Với tiền thì sẽ có thuộc tính quan trọng đó là "số lượng" và "đơn vị" ($, ...).

```ts
class Money {
  private readonly number amount;
  private readonly string currency;

  constructor(number amount, string currency) {
    this.amount = amount;
    this.currency = currency;
  }
}
```

Value-Object không chỉ đơn thuần chứa dữ liệu mà nó còn có thể có thêm các methods khác. Ví dụ "tiền" ở trên, ta có thể thêm method `Add`

```ts
class Money {
  private readonly number amount;
  private readonly string currency;

  constructor(number amount, string currency) {
    this.amount = amount;
    this.currency = currency;
  }

  public add(Money args) {
     return new Money(this.amount + args.amount, currency);
  }
}
```

Vì Value-Object là không thể thay đổi nên ở method `Add` chúng ta trả về một instance mới. Kết quả trả về sẽ được đưa vào một biến mới.

```ts
const money = new Money(1000, "$")
const allowance = new Money(2000, "$")

const result = money.add(allowance);
```

Xử lí ở trên hoàn toàn tương tự với xử lí tính toán cho các kiểu dữ liệu nguyên thuỷ.

```ts
const str1 = "str1";
const str2 = "str2";

const res = str1 + str2;
```

**Những phương thức không được định nghĩa**: ở trên ta có phương thức **thêm tiền** được định nghĩa nhưng phương thức **nhân tiền** lại không được định nghĩa. Khi đó trong phần định nghĩa của value-object ta chỉ đưa ra `function signature` mà thôi.

```ts
class Money {
  public Money multiply(Money args);
}
```

## Động cơ sử dụng value-object

Việc viết các class để từ đó tạo ra các object hiển thị các giá trị đặc trưng của hệ thống là một việc rất quen thuộc.

Việc phân chia, phân tán code dường như ít được để ý đến ở thời điểm hiện tại. Việc định nghĩa nhiều class value-object như vậy sẽ làm tăng số lượng các files định nghĩa class. Ban đầu, rất ít người có thể vượt qua được trở ngại này.

Vậy nên để vượt qua điều đó, chúng ta phải nắm được mục đích chính của bản thân khi sử dụng `value-object` là gì.

Có thể liệt kê 4 mục đích cơ bản như sau:

① Tăng khả năng hiển thị các giá trị đặc trưng của hệ thống.
Ta lấy ví dụ ở các công ty kinh doanh sản phẩm, họ sẽ có các mã hàng như:

- ItemNumber
- SerialNumber
- LotNumber

Các mã này được tạo nên từ "chữ" và"số". Nếu chỉ đơn thuần sử dụng kiểu dữ liệu `string` cho chúng

```ts
const serialNumber = "abc-123-xyz";
```

Khi đó nếu đọc code, chúng ta sẽ không thể rõ được đây là loại mã gì. Đồng thời việc tìm ra cấu trúc của nó cũng như trả lời cho câu hỏi nó được tạo ra như thế nào là vô cùng khó khăn, nếu ta định nghĩa value-object, mọi chuyện sẽ dễ dàng hơn nhiều.

```ts
class SerialCode {
  private string productCode;
  private string lotNumber;
  private string branch;
}
```

So với việc một "string" duy nhất thì việc sử dụng value-object có ý nghĩa hơn rất nhiều. Ngoài ra đây cũng là một cách "viết doc" cho hệ thống.

② Không để cho các giá trị "lệch chuẩn" tồn tại.
Ta lấy ví dụ với `userName`, ở phía người dùng nó chỉ đơn thuần là một `string`. Nhưng ở phía hệ thống sẽ có những quy tắc như:

- Tên phải có độ dàng trong khoảng [n, m]
- Tên chỉ chứa các kí tự alphabet và kí tự số
- ...

Lấy ví dụ:

```ts
const userName = "me";
```

Nếu thiết kế hệ thống yêu cầu độ dài tối thiểu của `userName` là 3, đoạn code trên vẫn được compile và chạy bình thường nhưng về mặt logic thì hoàn toàn sai. Ta có thể thêm code kiểm tra điều kiện

```ts
if (userName.length < 3) {

} else {
  // throw exception
}
```

 Việc này không hề khó nhưng càng thêm nhiều điều kiện thì code sẽ cồng kềnh và nếu điều kiện mới thêm không chính xác sẽ gây hại cho hệ thống.

Nếu sử dụng value-object, ta sẽ có class như sau:

```ts
class UserName {
  private readonly value;

  constructor (value) {
    if (!value || value.length < 3) {
       throw Exception
    }

    this.value = value;
  }
}
```

③ Tránh việc gán nhầm giá trị.
Việc gán giá trị là việc làm thường thấy đối với các lập trình viên, nhưng việc gán nhầm giá trị vẫn thường xuyên xảy ra.

```ts
class User {
  private readonly id;

  constructor(name: string) {
    this.name = id;
  }
}
```

Đoạn code này không sai, có những hệ thống sử dụng `name`, `email` làm id. Đối với người viết code thì không vấn đề gì, nhưng với người đọc code sau này có thể gây ra sự hiểu nhầm rằng "đoạn code này liệu có chính xác ?". Đọc code là không đủ mà cần xem xét đến thiết kế của hệ thống để biết được liệu nó có chính xác hay không.

Giải pháp ở đây vẫn là `value-object`

```ts
class UserName {
  private readonly value;
}

class UserId {
  private readonly value;
}

class User {
  private readonly id;
  private readonly name;

  constructor(UserName name) {
    this.id = name; // compiler error
  }
}
```

Dù `UserName` và `UserId` chỉ là lớp bao của string nhưng với `User` class nếu gán nhầm thì sẽ phát sinh lỗi ngay từ lúc compile.

Bản thân chương trình luôn có các lỗi tiềm tàng mà chỉ đọc code thôi ta không thể biết được, chỉ khi nào thực thi code mọi thứ mới trở nên rõ ràng. Việc sử dụng `value-object` này giúp giảm đi các lỗi tiềm tàng, qua đó người bảo trì hệ thống sau này sẽ không quá "vất vả" trong việc vận hành và bảo trì hệ thống.

④ Tránh việc logic nằm rải rác, không tập trung.

Với các logic kiểm tra điều kiện, nó có thể bị lặp lại ở nhiều nơi trong project, nếu điều kiện kiểm tra thay đổi ta sẽ phải thay đổi ở nhiều chỗ, việc làm này khá vất vả đặc biệt là với các hệ thống lớn.

Các logic thế này chỉ nên tập trung ở một chỗ

```ts
class UserName {
  private readonly value;

  constructor(name: string) {
    if (!name || name.length < 3) throw Exception;

    this.value = name;
  }
}

class User {
  private readonly UserName name;

  constructor(name: string) {
    const userName = new UserName(name);

    this.name = userName;
  }

  updateName(newName: string) {
    const newUserName = new UserName(newName);

    this.name = newUserName;
  }
}
```

Nếu điều kiện của `userName` có thay đổi thì ta chỉ cần thay đổi ở một chỗ duy nhất là được.
