### Về vấn đề sử dụng variable, class, function

**Tuân theo chuẩn PSR-1**

1. Về việc viết class, viết hàm, biến: 

Phải đặt theo kiểu camelCase (Cú pháp lạc đà): 

Tất cả mọi tên biến, function đều phải viết hoa trữ cái đầu nhưng trừ chữ đầu tiên.

VD: searchUser, _isCheckUser,...

(Kể cả class, id, name, hay bất kỳ thuộc tính nào khác được viết trong HTML cũng sử dụng quy tắc này).

3. Tên biến phải có ý nghĩa dựa theo nội dung, tính chất của biến đó. Về việc đặt function cũng tương tự.

4. Tên class phải viết hoa chữ cái đầu tiên của mỗi từ
VD: User, UserGroup, ...

5. Tên hằng số phải viết hoa toàn bộ, các từ cách nhau bởi dấu gạch dưới
VD: const MAX_USER = 100;

6. Về việc đặt tên file: Tuân thủ theo tiêu chuẩn PSR-4

7. Các file controller bên trong folder app/Http/Controllers/Admin phải đặt tên theo định dạng: AdminTênController.php
(Tránh tình trạng tên file trùng nhau khi sử dụng chung với các controller khác thuộc Service)

---

### Về việc tạo file:

1. Cố gắng khai thác việc tạo file dựa theo Resource của Laravel.
Không tạo file thủ công có cùng tính năng với resource.

2. Nếu tên file blade có chứa nhiều chữ thì cách nhau bởi dấu gạch dưới (cú pháp con rắn [snake_case])
: 

VD: user_group.blade.php

3. Khi tạo file blade, nếu file đó có chứa nhiều phần thì cần phân chia các phần bằng cách sử dụng comment để phân biệt các phần.

Hoặc tốt hơn là tạo ra các file blade khác để chứa các phần đó.

Ngoài ra, nếu có thể thì cố gắng tạo ra các file blade có thể tái sử dụng được. (Partials)

VD: Tạo ra file blade chứa các phần chung của các file blade khác như header, footer, sidebar, ...
Sau đó sử dụng @include để gọi các file blade này vào các file blade khác.

---

### Về viết code: 

**Tuân thủ theo chuẩn PSR-2**

>Việc quan trọng nhất là cố gắng nhìn bao quát tính năng mình đang làm. Nếu có thể, hãy tách nhỏ nó ra thành các function nhỏ, mỗi function chỉ làm một việc duy nhất. Điều này sẽ giúp cho việc debug, test, tái sử dụng và maintain code dễ dàng hơn.

Không viết code dài dòng, không viết code lặp lại. Nếu có thể, hãy viết code ngắn gọn, dễ hiểu, dễ đọc, dễ debug.

#### Format code
Luôn format code trước khi commit.

Format kể cả code thuộc Backend lẫn Frontend.

Về việc format code, có thể sử dụng một số  tool trong editor để format code tự động hoặc tự research để tìm ra cách format code theo chuẩn PSR-2.

#### Response 
$data trả về nên là một mảng chứa nhiều phần tử, không phải là một object.

Ví dụ về lỗi như sau:
![image-3.png](./resources/images/markdown/image-3.png)

Nếu như trường hợp cần trả về cùng lúc 2 object là **$user** và $item thì với cách viết như trên chắc chắn không thể  làm được.

Cách fix, phải chỉnh lại như sau:
```php
['data' => $data]
```
![image4.png](./resources/images/markdown/image-4.png)
