# Chương 13: Đặc tả hệ thống (Specification)

## Specification là gì ?

Là một object dùng để đánh giá các object khác có đạt một tiêu chuẩn được định nghĩa từ trước hay không. Việc đánh giá này là một rule quan trọng trong domain.

## Xác nhận các xử lí đánh giá phức tạp

Phần đánh giá dựa theo các điều kiện được chỉ ra sẽ được triển khai thông qua method của một object.

Ví dụ như class `Circle` ta có method đánh giá là `isFull`

```ts
class Circle {
  // Abbreviation
  
  public isFull() {
    return this.countMembers() >= 30;
  }
}
```

Điều kiện hiện tại là khá đơn giản, nhưng với các điều kiện phức tạp hơn thì sao. Ta thử xét điều kiện dưới đây:

- Ta đưa thêm một khái niệm mới đó là `premium user`
- Số lượng users tối đa (bao gồm cả owner) trong một circle là 30
- Nếu có trên 10 premium user thì số lượng users tối đa sẽ tăng lên thành 50.

Trong Circle có lưu các user entity là member nhưng làm cách nào để biết được user có phải là premium hay không, ta cần phải gọi tới `UserRepository`. Nhưng trong Circle không hề lưu UserRepository, vậy nên việc kiểm tra điều kiện sẽ phải tiến hành tại `ApplicationService`

```ts
class CircleApplicationService {
  private readonly circleRepository: ICircleRepository;
  private readonly userRepository: IUserRepository;  

  async join(command: CircleJoinCommand) {
    const circle = await circleRepository.findById(new CircleId(command.circleId));

    const users = await userRepository.find(circle.members);
    const premiumMembers = users.filter(user => user.isPremium);
    const circleUpperLimit = premiumMembers.length < 10 ? 30 : 50;

    if (circle.countMembers() >= circleUpperLimit) throw new Exception();
  }
}
```

Về bản chất thì đây là domain rule, việc viết logic dựa theo domain rule trong ApplicationService là điều cần tránh. Vì khi đó Domain Entity sẽ không còn ý nghĩa gì nữa và ngoài ra các domain rules quan trọng đều nằm rải rác ở các ApplicationService.

Vậy nên ta sẽ viết `isFull() method` vào trong Circle Entity class, tuy nhiên nếu chỉ lưu id của các members thì việc lấy thông tin của user mà không có `UserRepository` là điều không thể, do đó `isFull()` method cần có tham số đầu vào là `UserRepository`.

```ts
class CircleEntity {
  private readonly members: UserId[];  

  async isFull(userRepository: UserRepository) {
    const users = userRepository.findByIdList(this.members);

    const premiumUsers = users.filter(user => user.isPremium);
    const upperLimit = premiumUsers.length < 10 ? 30 : 50;

    return this.members.length >= upperLimit;
  }
}
```

Tuy nhiên đây là một cách giải quyết khá tồi vì Repository sẽ không được phép truyền vào domain entity mà chỉ góp phần hoàn thiện cho `Domain Design` mà thôi.

Vì thế để `entity` và `value-object` chuyên tâm cho việc biểu thị `domain model` ta cần tránh việc thao tác với repository trong chúng.

## Giải quyết nhờ 「Specification」

Vì không được phép sử dụng repository trong `entity` hoặc `value-object` nên ta sẽ sử dụng `Specification` object.

```ts
class CircleFullSpecification {
  private readonly userRepository: IUserRepository;

  constructor(userRepository: IUserRepository) {
    this.userRepository = userRepository;
  }

  isSatisfiedBy(circle: Circle) {
    const users = this.userRepository.find(circle.members);
    const premiumUsers = users.filter(user => user.isPremium);
    const upperLimit = premiumUsers.length < 10 ? 30 : 50;

    return circle.countMembers() >= upperLimit;
  }
}
```

Specification object chỉ làm nhiệm vụ đánh giá các objects nên code trở nên rõ ràng và gọn hơn rất nhiều.

### Các object mà ta khó thấy được nhiệm vụ của chúng

Khi mọi xử lí nhằm đánh giá một điều kiện nào đó (`isFull`, `isEmpty`, ...) được đặt trong object thì nhiệm vụ chính của object sẽ bị lu mờ đi, người đọc code hoàn toàn không hiểu được tại sao object này lại tồn tại.

```ts
class Circle {
  public boolean isFull();
  public boolean isEmpty();
  public boolean isLocked();
  public boolean isPrivate();
  public void join(user: User);
}
```

Lúc này, sự phụ thuộc vào object sẽ vượt ngoài tầm kiểm soát. Bất cứ một sự thay đổi nhỏ nào cũng sẽ gây ra những ảnh hưởng đáng kể cho hệ thống.

Việc đánh giá một object không hoàn toàn giới hạn trong việc sử dụng các methods, ta hoàn toàn có thể chuyển quá trình đánh giá đó ra một object bên ngoài như `Specification Object` ở ví dụ trên.

## Tránh việc sử dụng Repository

Specification về bản chất là một dạng domain object mang tính chất liệt kê. Bên trong specification ta nên tránh việc sử dụng Repository, mà thay vào đó sẽ là `First Class Collection`.

`First Class Collection` là một pattern sử dụng một tập hợp các objects nhất định chứ không chỉ đơn thuần là sử dụng List các objects chung chung.

```ts
class CircleMembers {
  public id: CircleId;
  private readonly owner: User;
  private readonly members: User[];

  constructor(id: CircleId, owner: User, members: User[]) {
    this.id = id;
    this.owner = owner;
    this.members = members;
  }

  countMembers() {}
  countPremiumMembers() {}
}
```

Ta thấy `CircleMembers` khác với List thông thường ở chỗ nó lưu:

- CircleId
- Members

Khi đó Specification sẽ như sau:

```ts
class CircleMembersFullSpecification {
  isSatisfiedBy(CircleMembers members) {
    const premiumMembersCount = members.countPremiumMembers();
    const upperLimit = premiumMembersCount < 10 ? 30 : 50;

    return members.countMembers() > upperLimit;
  }
}
```

Khi đó bên trong ApplicationService (usecase) - ta sẽ phải nạp đầy dữ liệu cho `First class collection`

```ts
const circleMembers = new CircleMembers(circle.id, circle.owner, circle.members);
```

## Kết hợp giữa Specification và Repository

Có thể kết hợp Repository và Specification bằng cách truyền Specification vào Repository để từ đó tìm ra object match với yêu cầu của Specification.

Ta thử xét ví dụ khi User muốn Join vào Circle phù hợp với mình, khi đó ta cần có chức năng gợi ý Circle cho User. Lúc này ta cần định nghĩa thế nào là một Recommendation Circle (Circle mới tạo? Circle với tần suất hoạt động cao? ...)

- Circle mới tạo là Circle được tạo trong vòng 1 tháng (điều kiện giả định)
- Circle với tần suất hoạt động cao là Circle có số lượng members > 10 (điều kiện giả định)

Như từ trước tới nay, việc tìm kiếm Recommendation Circle sẽ được thực hiện trong Repository.

```ts
interface ICircleRepository {
  findRecommended(Date now): Circle[];
}
```

Nhưng có một vấn đề ở đây là điều kiện đưa ra Recommendation Circle sẽ phụ thuộc vào class thực thi Repository. Trong tương tai thì tính năng Recommendation này sẽ trở nên quan trọng, do đó việc để nó phụ thuộc vào class implement Repository nằm ở tầng infrastructure là điều không nên.

## Giải quyết bằng Specification

Chúng ta hoàn toàn có thể định nghĩa việc kiểm tra xem Circle có là Recommendation hay không ngay tại Specification Object.

```ts
class CircleRecommendSpecification {
  private readonly executeDate: Date;

  constructor(executeDate: Date) {
    this.executeDate = executeDate;
  }

  isSatisfiedBy(Circle circle) {
    if (circle.countMembers() < 10) return false;

    return circle.createdAt > executeDate.addMonths(-1);
  }
}
```

Lúc này ta không cần phải đưa điều kiện của Recommendation Circle vào trong Repository nữa. Tuy nhiên nếu vẫn muốn truyền Specification vào Repository ta vẫn có thể sử dụng interface như sau.

```ts
interface ISpecification<T> {
  isSatisfiedBy(T input);
}

class CircleRecommendationSpecification implements ISpecification<Circle> {}

interface ICircleRepository {
  find(specification: ISpecification<Circle>): Circle[];
}
```

Điều này tránh việc phải thêm method tuỳ theo loại của Specification.  Ví dụ nếu ta có thêm một class nữa implements `ISpecification<Circle>` thì việc truyền vào ICircleRepository là việc hoàn toàn có thể. Khi đó việc sử dụng Repository và Specification sẽ như sau:

```ts
class CircleRecommendationApplicationService {
  find() {
    const circleRecommendSpecification = new CircleRecommendationSpecification(new Date());

    const recommendCircles = circleRepository.find(circleRecommendSpecification);
  }
}
```

Nhờ việc sử dụng Specification, mọi điều kiện để trở thành Recommendation Circle sẽ nằm trong domain object chứ không được ghi lại trên usecase (ApplicationService).

## Vấn đề về hiệu năng khi kết hợp Specification và Repository

Khi truyền Specification vào Repository ta sẽ phải truy vấn toàn bộ dữ liệu rồi sau đó tìm ra những bản ghi phù hợp với điều kiện của Specification để rồi tạo ra các Entity Object tương ứng.

Điều này hoàn toàn ổn khi lượng dữ liệu nhỏ nhưng khi dữ liệu nhiều lên thì hiệu năng sẽ tệ đi trông thấy. Đây là một vấn đề mà ta cần cân nhắc khi kết hợp Repository và Specification.

## Sử dụng Read Model cho các Query phức tạp

Việc tránh để các domain rules bị rò rỉ ra ngoài là cần thiết song nếu điều đó ảnh hưởng đến hiệu năng của hệ thống, từ đó gây ảnh hưởng đến người dùng là việc cần phải xem xét lại.

Có những câu query dùng để lấy ra các dữ liệu phức tạp, nhưng thực sự thì chúng không phải là logic của domain. Ngược lại thì command (ghi dữ liệu vào DB) thì lại liên quan đến domain khá nhiều.

Vậy nên để tránh ảnh hưởng của các câu query lên domain logic, ta nên tách biệt rõ ràng giữa query và command để vừa đảm bảo yêu cầu về hiệu năng của hệ thống cũng như kiểm soát được logic của hệ thống.

Đây là những điều liên quan đến hai tư tưởng:

- CQS (Command-Query Separation)
- CQRS (Command Query Responsibility Segregation)
