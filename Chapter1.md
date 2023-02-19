# Chương 1: DDD là gì ?

## Modeling

Là quá trình trừu tượng hoá các đối tượng trong thực tế. Tuỳ vào domain (lĩnh vực) mà việc mô hình hoá sẽ có sự khác biệt.

VD: Cùng là một cây BÚT nhưng với:

- Writer: sẽ quan tâm đến việc có viết được hay không.
- Present: sẽ quan tâm đến hình thức bên ngoài nhiều hơn.

## Domain Model

Trong DDD, sau khi tiến hành `modeling` ta sẽ thu được các `models` các `models` này được gọi là `domain models`.

## Domain Object

Là thể hiện của Domain Model.

Khi domain thay đổi thì domain model cũng sẽ phải thay đổi theo. Cần phải hiểu rõ nghiệp vụ (Domain), nếu không thì hệ thống sẽ vận hành không như ý.

## Mối quan hệ giữa Domain, Domain Model, Domain Object

![Screen Shot 2021-06-15 at 16 12 16](https://user-images.githubusercontent.com/15076665/122008924-8579d300-cdf4-11eb-9d2b-25f961317c19.png)

## Những pattern cơ bản

- value-object
- entity
- domain-service

Những patterns cần biết khi triển khai ứng dụng:

- Repository
- Application service
- Factory

Các patterns khác cần biết:

- Aggregate
- Specification

## Phân loại pattern

Pattern sẽ được phân thành 2 nhóm:

- Biểu thị nghiệp vụ (Domain knowledge)
- Triển khai ứng dụng

Dưới đây là sơ đồ mối liên hệ giữa các pattern:

![Screen Shot 2021-07-03 at 21 37 25](https://user-images.githubusercontent.com/15076665/124354424-e99af480-dc46-11eb-8444-c32d2514458f.png)
