# CloudFormation:
khai báo, cấu hình các tài nguyên hạ tầng AWS bằng code

# Lợi ích của CFN
- Infrastructure as code
	Không tài nguyên nào cần tạo thủ công, điều này giúp quản lý dễ hơn
	Code có thể dễ dàng quản lý version, ví dụ dùng Git
	Thay đổi hạ tầng có thể review thông qua code
- Cost
	Mỗi tài nguyên trong Stack đều được đánh dấu, vì thế có thể dễ dàng theo dõi chi phí của stack
	Ước tính chi phí mà tài nguyên sử dụng, bằng CFN template
	Saving strategy: trong giai đoạn dev, có thể tự động xóa template trong thời gian không sử dụng, và tạo lại khi cần thiết
- Productivity
	Có khả năng xóa bỏ và tái tạo hạ tầng trên Cloud
	Tự động tạo Diagram cho template
	Khai báo hạ tầng hoàn toàn bằng code, không cần quan tâm thứ tự khai báo
- Phân tách các công việc, tạo nhiều stack cho nhiều ứng dụng. Ví dụ
	VPC stacks
	Network stacks
	App stacks
- Tận dụng các template có sẵn

Separate stack

Template: là file JSON hoặc YAML chứa code khai báo
Stack: tập hợp các tài nguyên được cung cấp theo như file template mô tả

Flow Works
Template -> S3 bucket <- AWS CloudFormation -> Stack -> AWS Resources
Các template sẽ được upload lên S3 và AWS CFN sẽ lấy file từ S3 để đọc. Sau khi có thông tin, CFN sẽ tạo stack. Trong stack sẽ diễn ra các công việc tạo tài nguyên

Để update template, không thể chỉnh sửa file đã upload trên S3, mà phải thực hiện flow works từ đầu (upload lại từ đầu)

Các stacks được xác định bởi tên. Xóa bỏ stack cũng sẽ xóa bỏ các tài nguyên được tạo ra trong stack

## Parameters:
- cho phép nhập vào giá trị tùy chọn mỗi lần tạo stack
- validate các giá trị đầu vào (Type, Min/MaxLength, Min/MaxValue, AllowedValues, AllowedPattern)

Khi nào sử dụng Parameters
- giá trị cấu hình chưa thể xác định ngay từ đầu, có thể thay đổi sau này
- tạo sự linh hoạt trong việc cấu hình => đem đi sử dụng nhiều nơi, nhiều trường hợp
- không muốn upload lại nhiều lần file template mỗi lần chỉnh sửa

Để sử dụng Parameters, sử dụng Fn:Ref để tham chiếu tới. Viết tắt là !Ref

Rseudo Parameters: là các tham số không cần khai báo và có thể sử dụng ở bất kỳ template nào
Ví dụ: AWS::AccountId, AWS::Region, AWS::StackId,.,.

## Resources:
- là thành phần chính khai báo các tài nguyên
- thuộc tính DependsOn dùng để xác định tài nguyên chỉ được tạo sau khi 1 tài nguyên khác đã tạo xong.
	Function !Ref và !GetAtt tự động áp dụng thuộc tính DependsOn
- thuộc tính DeletionPolicy để xác định hành động khi tài nguyên bị xóa. Có 3 hành động là Retain, Snapshot, Delete
- thuộc tính UpdateReplacePolicy để xác định hành động khi update replatement xảy ra với tài nguyên

## Mapping:
- là các giá trị cố định được sử dụng cho template
- dùng để phân biệt giá trị được sử dụng cho từng môi trường, Region, AMI type,.,

## So sánh Mapping với Parameters
Sử dụng Mapping trong trường hợp ta biết tất cả giá trị có thể sử dụng liên quan đến Region, AZ, account, environment (dev vs prod).
Sử dụng Parameter trong trường hợp giá trị dành riêng cho người dùng
Sử dụng Mapping an toàn hơn Parameters vì các giá trị trong Mapping đều được xác định trước

## Outputs:
- khai báo các giá trị mà có thể dùng làm đầu vào cho các stack khác. Ví dụ: outputs của 1 stacks có thể là VPC ID hoặc Subnet ID để các stack khác có thể tham chiếu tới
- Có thể xem giá trị các outputs trong AWS Console hoặc AWS CLI
- Thường dùng cho mô hình Cross stack - các stack tạo sau sử dụng giá trị export của stack tạo trước

## Conditions:
- xác định xem tài nguyên có được tạo hay không
- dựa vào các giá trị tham số đầu vào

Các tài nguyên sau khi tạo xong thường có giá trị trả về. Lấy Id thì sử dụng Fn::Ref. Để lấy giá trị khác thì sử dụng Fn::GetAtt

Ruels
- validate Parameters dựa vào các Parameters khác
- bao gồm 1 Rule Condition (tùy chọn) và một hoặc nhiều Assertions

## Metadata:
- cung cấp thêm thông tin cho template
- AWS::CloudFormation::Designer thông tin kích thước, vị trí resource hiển thị trên design graph
- AWS::CloudFormation::Interface nhóm và sắp xếp các Parameters hiển thị trên AWS Console
- AWS::CloudFormation::Init dùng thay thế cho User data

## Nested Stacks: mô hình lồng ghép các Stack.
- Tạo ra các template định nghĩa nested stack, rồi upload lên S3
- Root stack thông qua nested stack template để gọi đến chúng
- 2 Root sack khác nhau cùng gọi đến 1 nested stack thì sẽ có 2 nested riêng biệt.

## Exported Stack Output Values (Cross stack) vs Nested Stacks
- Có 1 tài nguyên trung tâm chia sẻ cho nhiều stack khác => dùngg Cross stack
- Nếu các stack có nhu cầu cập nhật riêng lẻ => dùng Cross stack
- Nếu tài nguyên chỉ được sử dụng bởi 1 stack và muốn tái sử dụng 1 phần code => dùng Nested stack
- Trong trường hợp Nested stack update, cần phải update thủ công lại Root stack


dùng Change set giúp xem trước các thay đổi có thể ảnh hưởng đến tài nguyên đang hoạt động

tạo mới mà lỗi thì sẽ xóa hết các tài nguyên đã tạo
update mà lỗi thì sẽ rollback các tài nguyên về trạng thái trước
