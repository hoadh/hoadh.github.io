---
title: "Áp dụng Luật Demeter"
author: "hoad"
date: 2020-09-30T03:19:07+07:00
---

>    "Không nói chuyện với người lạ"

## Giới thiệu

*Luật Demeter* có thể giúp bạn áp dụng các quy tắc trừu tượng hơn của Lập trình Hướng đối tượng (OOP) để đảm bảo ứng dụng có thể mở rộng và có thể bảo trì.

Luật Demeter có các nguyên tắc sau:

*    Mỗi đơn vị nên biết ít về đơn vị khác
*    Mỗi đơn vị chỉ nên nói chuyện với bạn của nó (nói các khác, không được nói chuyện với *người lạ*).
*    Chỉ nói chuyện với *bạn thân*

Trong việc thiết kế phần mềm, *loosely coupling* (tạm hiểu là "liên kết lỏng kéo") là một thuật ngữ được nhắc đến nhiều. Đây một trong những yếu tố giúp phần mềm dễ dàng mở rộng và bảo trì.

Luật Demeter sẽ giúp giảm thiểu *tight coupling* (phụ thuộc chặt chẽ với nhau) giữa các thành phần trong ứng dụng. Nhờ đó, chúng ta có thể dễ dàng thay thế hoặc mở rộng khi có nhu cầu.

## Luật Demeter

### Lý thuyết

Luật Demeter cho hàm yêu cầu có phương thức M trong object O có thể chỉ liên quan tới các phương thức thuộc các loại object sau:

1.   O - chính nó
2.   Object là tham số của hàm M
3.   Bất kì object nào được khởi tạo bên trong M
4.   Object là một thuộc tính của O
5.   Biến global, có thể truy cập được qua O (trong phạm vi M)

### Ví dụ tốt

Để hiểu rõ về nội dung có phần nặng lý thuyết ở trên, chúng ta có thể xem qua ví dụ sau để có được hình dung đầu tiên. Ví dụ này được viết với ngôn ngữ lập trình Java:

```java
public class LawOfDemeter
{
  private Topping cheeseTopping;
  
  public void goodExamples(Pizza pizza)
  {
    Foo foo = new Foo();
    
    // (1) Trường hợp gọi phương thức của chính nó
    doSomething();
    
    // (2) Trường hợp gọi các phương thức của đối tượng là tham số đầu vào
    int price = pizza.getPrice();
    
    // (3) Trường hợp gọi phương thức của bất kì đối tượng nào được khởi tạo bên trong class này
    cheeseTopping = new CheeseTopping();
    float weight = cheeseTopping.getWeightUsed();
    
    // (4) Tương tự trường hợp (3)
    foo.doBar();
  }
  
  private void doSomething()
  {
    // ...
  }
}

// Mã nguồn tham khảo tại https://alvinalexander.com/java/java-law-of-demeter-java-examples/
```

### Ví dụ chưa tốt

Dưới đây là một ví dụ vi phạm các nguyên tắc của Luật Demeter:

```java
class BasicExample {
 public String getStreetName(Employee employee) {
  Address address = employee.getAddress();
  return address.getStreetName();
 }
}
```

Để có được `address`, chúng ta gọi phương thức `getAddress()` của `employee` tại dòng thứ ba. Đối tượng `address` không phải là đối tượng được liệt kê trong danh sách các tình huống mà chúng ta được phép tương tác trực tiếp. Thế nhưng tại dòng thứ tư, chúng ta thấy rằng kết quả trả về được lấy từ phương thức `getStreetName()` thuộc đối tượng `address`. 

### Giải pháp khắc phục

Thay vì gọi trực tiếp `getStreetName()` từ `address` để lấy được thông tin tên đường phố, chúng ta tách đoạn mã trên và khai báo một phương thức trong khai báo class `Employee` để trả về giá trị từ `getStreetName()`. Nhờ thế, chúng ta có thể gọi trực tiếp phương thức `getStreetName()` này với đối tượng `employee` như dưới đây:

```java
class Employee {
 private Address address;
 public String getStreetName() {
  return address.getStreetName();
 }
}

class BasicExample {
 public String getStreetName(Employee emplyee) {
  employee.getStreetName();
 }
}
```

## Lợi ích

Việc tuân thủ các nguyên tắc trong Luật Demeter sẽ giúp chúng ta đạt được những lợi ích dưới đây khi lập trình:

*    Các đối tượng sẽ tránh được việc phụ thuộc chặt chẽ với nhau
*    Tái sử dụng class dễ dàng hơn
*    Dễ dàng thay thế class khi yêu cầu thiết kế thay đổi
*    Mã nguồn dễ kiểm thử hơn
*    Các class được thiết kế theo hướng này sẽ ít lỗi hơn

## Kết luận

Hiểu về *Luật Demeter* và áp dụng được vào lập trình sẽ giúp bạn có sản phẩm được thiết kế tốt hơn. Bên cạnh đó, khi kết hợp với những kỹ năng về Refactoring (Tái cấu trúc mã nguồn - Chủ đề của Tạp Chí Lập Trình vol.8) sẽ giúp bạn cải thiện được chất lượng mã nguồn. Hy vọng bài viết này giúp bạn có thêm chút kiến thức để dễ dàng ra quyết định khi thực hiện các nhiệm vụ liên quan đến *refactoring*.
