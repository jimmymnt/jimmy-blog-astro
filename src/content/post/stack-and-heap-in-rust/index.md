---
title: "Stack and Heap in Rust"
publishDate: "3 December 2023"
description: "The Stack and the Heap in Rust program language"
coverImage:
  src: "./rust.jpg"
  alt: "The Stack and the Heap in Rust program language"
tags: ["rust", "stack", "heap"]
---
## Stack và Heap

Hello mọi người, lâu lắm rồi mình mới ngồi lại viết blog với một chủ đề về Rust.

Đầu tiên để nói về Rust - thì nó là một ngôn ngữ lập trình bậc thấp (low level language), nếu bạn đến từ một ngôn ngữ lập trình bậc cao hơn (ví dụ: .NET/PHP/NodeJS/Java) thì có thể có một số khía cạnh mà bạn có thể chưa quen thuộc.

Và cái mình muốn nhắc đến ngày hôm nay là "Cách quản lý bộ nhớ hoạt động" - với một Stack (Ngăn xếp) và Heap (dạng đống 🤣 - data structure). Nếu như bạn đã từng làm với C/C++ thì bạn sẽ quen thuộc với việc sử dụng Ngăn Xếp để phân bổ vùng/bộ nhớ. thì bài viết này sẽ là một thứ gì đó khá mới lạ. Nếu không phải, thì bạn sẽ biết thêm một số thứ từ bài viết này, nhưng với cách tiếp cận ở trong ngôn ngữ lập trình `Rust`.

Như tất các mọi thứ khác, khi chúng ta học, chúng ta sẽ sử dụng một mô hình đơn giản để bắt đầu, điều nay cho phép bạn làm quen với những thứ cơ bản mà không bị chìm đắm vào chi tiết không quan trọng. Các ví dụ sẽ không sử dụng 100% chính xác, nhưng nó sẽ đại diện cho mức độ của mình chúng ta đang cố gắng học ở thời điểm hiện tại. Một khi bạn đã nắm vững cơ bản, việc tìm hiểu thêm về triển khai cấp phát bộ nhớ, bộ nhớ ảo (virtual memory) và các chủ đề nâng cao khác sẽ dễ dàng hơn một phàn nào đó 😄 .

## Quản lý bộ nhớ (Memory management)

Có 2 thuật ngữ liên quan đến việc quản lý bộ nhớ, đó chính là Stack (ngăn xếp) và Heap (đống), 2 thuật ngữ trừu tượng này sẽ giúp bạn xác dịnh được khi nào cần cấp phát và khi nào cần giải phóng.

Bây giờ chúng ta sẽ so sánh xem thử Stack và Heap có những đặc điểm nổi bật gì nhé :letsgo:

Stack thì rất nhanh, và là nơi mà trong Rust các biến sẽ được cấp phát mặc định. Tuy nhiên, việc cấp phát này chỉ tồn tại trong phạm vi của một hàm (function) được gọi và bị giới hạn rất nhiều về kích thước. Ngược lại, Heap (đống) hoạt động chậm hơn và được cấp phát một cách rõ ràng hơn bởi chương trình của chúng ta, nhưng nó lại có một đặc điểm đi ngược lại với Stack, đó là nó có tốc độ chậm hơn, nhưng hiệu quả, không giới hạn về kích thước, và có thể truy cập một cách toàn cục (global access).

## Ngăn Xếp (The Stack)
Okay 😸  bây giờ chúng ta sẽ đi vào một ví dụ cụ thể và đơn giản như sau

filename: `src/main.rs`
```rs
fn main() {
    let age = 31;
}
```

Đoạn code trên của chúng ta sẽ có một giá trị được khởi tạo là `age`, và tất nhiên `age` sẽ phải được khởi tạo ở một nơi nào đó đúng kg nào? Quay lại khái niệm chúng ta nói ở trên, trong Rust, "Stack Allocates" sẽ là mặc định, có nghĩa là giá trị "cơ bản" được cấp phát ở stack (ngăn xếp).

Điều này có ý nghĩa gì?

Well, khi 1 hàm được gọi (called), một số bộ nhớ sẽ được cấp phát cho tất cả các biến cục bộ (local) của nó và một số các thông tin khác, điều này được gọi là "STACK FRAME" (trong phạm vi bài viết này, mình xin phép không đề cấp đến vì nó khá phức tạp và mất nhiều thơi gian).

Quay lại ví dụ phía trên, khi `main()` được gọi, bộ nhớ sẽ cấp phát một số nguyên kiểu 32-bit DUY NHẤT cho stack frame đối với chương trình của chúng ta, điều này được xử lý tự động, và các bạn sẽ không cần phải chỉ định cho Rust biết về điều đó.

Và khi mà chương trình chạy xong, trong ví dụ trên nếu `main()` đã được thực thi và chạy xong, stack frame của chương trình trên sẽ được giải phóng hoàn toàn, hay trong Rust và một số ngôn ngữ lập trình khác, điều này được gọi là `deallocated`.

Điều quan trọng để hiểu ở đây là phương pháp cấp phát trên ngăn xếp (stack allocation) rất, rất nhanh.
Nhưng nhược điểm ở đây là chúng ta không thể giữ các giá trị lại nếu chúng ta cần chúng trong thời gian dài hơn một hàm duy nhất. Chúng ta cũng chưa nói về ý nghĩa của từ 'stack'. Để làm điều đó, chúng ta cần một ví dụ phức tạp hơn một chút:

filename: `src/main.rs`
```rs
fn more_info() {
    let month = 8;
    let day = 11;
}

fn main() {
    let year = 1990;

    more_info();
}
```

Chương trình này có tổng cộng 3 biến: 1 biến trong hàm `main()` và 2 biến ở trong hàm `more_info()`.
Như các bạn đã biết, khi `main()` được gọi, một số nguyên 32-bit sẽ được cấp phát cho stack frame. Tuy nhiên, trước khi chúng ta có thể mô tả điều gì xảy ra khi `more_info()` được gọi, chúng ta cần hiểu được điều gì đang xảy ra với bộ nhớ.

Phewwwww, máy tính của bạn sẽ đa phần xử dụng RAM để cấp phát bộ nhớ cho chương trình, và nó rất nhiều, tưởng tượng đơn giản là như một mảng (array) khổng lồ :giant:, và hãy thử hiện thực hóa nó lên nào.

Yoyo! Khi `main()` được gọi, ta sẽ gặp ngay 1 biến có tên là `year` và giá trị của nó là `1990`, nó sẽ được lưu trữ ở RAM như sau:
| Địa chỉ | Tên | Giá trị |
|-------|-------|-------|
| 0x1000 | year | 1990 |

Đây là 1 ví dụ về Stack Frame.
Wooho! Chúng ta đã có `age` đã được yên vị tại địa chỉ `0x1000`, với giá trị là `1990`.
Khi `more_info()` được gọi, stack frame của chúng ta sẽ là:

| Địa chỉ | Tên | Giá trị |
|-------|-------|-------|
| 0x1002 | day | 11 |
| 0x1001 | month | 8 |
| 0x1000 | year | 1990 |

Sau khi `more_info()` kết thúc, stack frame của chúng ta sẽ như sau:
| Địa chỉ | Tên | Giá trị |
|-------|-------|-------|
| 0x1000 | year | 1990 |

Và cuối cùng, sau khi `main()` kết thúc, tất cả các giá trị sẽ được giải phóng. EASYYYYY GAME! 😄

Nó được gọi là STACK vì như các bạn thấy, nó hoạt động như một "Băng đạn" hay "Hộp tiếp đạn", viên đạn đầu tiên sẽ là viên cuối cùng bay ra khỏi nòng súng (FILO).

Nào, đừng bỏ cuộc, hãy đến với ví dụ 3 và hơi khó hơn 1 chút nào 🤟

filename: `src/main.rs`
```rs
fn new_i() {
    let i = 6;
}

fn more_numbers() {
    let a = 11;
    let b = 1001;
    let c = 10011;

    new_i();
}

fn main() {
    let yoyo = 31;

    more_numbers();
}
```

Phewww, hãy biểu diễn nó trên Stack Frame cho ví dụ trên của chúng ta nào :letsgo:
Khi `main()` được gọi, stack frame đầu tiên sẽ là:
| Địa chỉ | Tên | Giá trị |
|-------|-------|-------|
| 0x1aa0 | yoyo | 31 |

Tiếp theo, ở trong `main()` chúng ta sẽ gọi hàm `more_numbers()`, sau khi `more_numbers()` được gọi, lần lượt stack frame của chúng ta sẽ thay đổi như sau:
| Địa chỉ | Tên | Giá trị |
|-------|-------|-------|
| **0x1aa3** | **c** | **10011** |
| **0x1aa2** | **b** | **1001** |
| **0x1aa1** | **a** | **11** |
| 0x1aa0 | yoyo | 31 |

và sau đó `more_numbers()` sẽ gọi cho `new_i()`, stack frame của chúng ta sẽ là:
| Địa chỉ | Tên | Giá trị |
|-------|-------|-------|
| 0x1aa4 | i | 6 |
| **0x1aa3** | **c** | **10011** |
| **0x1aa2** | **b** | **1001** |
| **0x1aa1** | **a** | **11** |
| 0x1aa0 | yoyo | 31 |

Wowww! Stack frame của chúng ta đang dần dần lớn lên rồi 😆

Sau khi `new_i()` gọi xong, nó sẽ kết thúc, các giá trị trong hàm sẽ được giải phóng (deallocated) và stack frame của chúng ta:
| Địa chỉ | Tên | Giá trị |
|-------|-------|-------|
| **0x1aa3** | **c** | **10011** |
| **0x1aa2** | **b** | **1001** |
| **0x1aa1** | **a** | **11** |
| 0x1aa0 | yoyo | 31 |

Sau đó, `more_numbers()` khi gọi xong thì các frame của hàm đó cũng sẽ được giải phóng, stack frame lúc này sẽ là:
| Địa chỉ | Tên | Giá trị |
|-------|-------|-------|
| 0x1aa0 | yoyo | 31 |

Sau khi xong chương trình, tất cả vùng nhớ và stack frame của chúng ta sẽ hoàn toàn được giải phóng -> RỖNG.

| Địa chỉ | Tên | Giá trị |
|-------|-------|-------|

## The Heap (Đống)

HIện tại, mọi thứ đều có vể hoạt động ổn định, nhưng kg phải tất cả mọi thứ đều hoạt động theo cách các ví dụ trên. Đôi khi, chúng ta cần truyền một số bộ nhớ hay biến (variable) giữa các hàm (function) với nhau, hoặc giữ các biến còn tồn tại (keep alive) lâu hơn. Đối với yêu cầu này, chúng ta có thể dùng Heap để phân bổ vùng nhớ.

Trong Rust, chúng ta có thể phân bố vùng nhớ ở Heap với việc khai báo kiểu dữ liệu [`Box<T>`](https://doc.rust-lang.org/rust-by-example/std/box.html). Dưới đây là 1 ví dụ:

```rs
fn main() {
    let age = Box::new(31);
    let another_value = 12;
}
```
Đây là những gì mà trong Stack Frame của chúng ta có thể hình dung:
| Địa chỉ | Tên | Giá trị |
|-------|-------|-------|
| 0x1aa1 | another_value | 12 |
| 0x1aa0 | age | ?????? |

Chúng ta sẽ phải cấp phát vùng nhớ cho 2 biến trên ngăn xếp. Trong đó `another_value` sẽ là 12 như mọi lúc, vậy còn `age` của chúng ta thì sao nào? 🤔 Thực ra `another_value` là một `Box<i32>`, và các Box (boxes) sẽ được cấp phát bộ nhớ ở Heap. Giá trị thực tế của `Box` là một cấu trúc (Struct) mà trong đó có một con trỏ (pointer) sẽ trỏ đến Heap.
Khi `main()` được thực thi và hàm `Box::new()` được gọi, máy tính sẽ cấp phát một bộ nhớ ở Heap và đặt giá trị `31` vào đó. Lúc này Stack Frame của chúng ta sẽ trông như sau:

| Địa chỉ | Tên | Giá trị |
|-------|-------|-------|
| 0x1000 |  | 31 |
| .... | .... | ... |
| .... | .... | ... |
| 0x1aa1 | another_value | 12 |
| 0x1aa0 | age | ->0x1000 |

Trong đó, `->0x1000` là một POINTER đang trỏ đến địa chỉ của vùng nhớ cấp phát cho `age` ở Heap.


> 📓 Một lưu ý nhỏ ở đây, chúng ta chưa nói về những gì đang thực sự diễn ra hoặc ý nghĩa thực sự của việc "allocate" hoặc "deallocate" bộ nhớ trong bài viết này, đi chi tiết vào nó sẽ thực sự rất dài và tốn nhiều thời gian, một cái nữa đó là không phải Heap mà mình đang đề cập ở bài viết này sẽ được lưu như dạng trong ví dụ là được bắt đầu từ phía trên cùng. Mình chỉ đang đưa ra một biểu diễn dễ hiểu cho các bạn 😄 Mình sẽ nói cụ thể về Heap được lưu ở đâu ở những bài viết sau trong series.

Ví dụ, chúng ta có 1GB RAM, thì bộ nhớ của chúng ta sẽ có (2^30) addresses. Và Stack của chúng ta bắt đầu từ vị trí 0 = `0x1aa0`, và kết thúc ở `0x1000`, hay nói cách khác giới hạn sẽ nằm trong `[0x1aa0...0x1000]`.

Trong trường hợp ở ví dụ của chúng ta, chúng ta đã phân bổ bộ nhớ cho `another_value` ở Heap, và sau khi `main()` được gọi và kết thúc, thì `age` sẽ được giải phóng bộ nhớ, trước tiên nó sẽ giải phóng bộ nhớ của Heap nếu có trước, và trong ví dụ của chúng ta sẽ là `0x1000`, lúc này stack frame của chúng ta sẽ trông như sau:

| Địa chỉ | Tên | Giá trị |
|-------|-------|-------|
| 0x1aa1 | another_value | 12 |
| 0x1aa0 | age | ?????? |

Ở trong Rust, `Box<T>` sẽ có một hàm (mình sẽ nói rõ hơn ở những phần sau) có tên là `Drop`, `<Box>` implement (`impl`) để giải phóng bộ nhớ.

Tuyệt vời 💯 , vì thế nên khi `age` dropped, nó sẽ giải phóng vùng nhớ đã được khỏi tạo (phân bổ) trên Heap 🏆 .
Chúng ta cũng có thể khiến cho vùng nhớ tồn tại (alive) lâu hơn bằng cách sử dụng "Transferring Ownership" hoặc đôi lúc một số bạn gọi là "moving out of the box" mà mình sẽ giải thích cụ thể ở 1 blog khác.

Yoyoooo! Đến lúc này thì stack frame của chúng ta sẽ được hoàn toàn giải phóng....

| Địa chỉ | Tên | Giá trị |
|-------|-------|-------|


## Arguments và borrowing

Chúng ta đã trải qua một số ví dụ có vẻ là cơ bản với Stack và Heap, nhưng sẽ như thế nào nếu đề cập về điều chúng ta đang nói với một hàm có sử dụng "arguments và borrowing"? Dưới đây là một ví dụ:

```rs
fn take_ref(i: &i32) {
    let x = 1991;
}

fn main() {
    let a = 1990;
    let b = &a;

    take_ref(b);
}
```

Chúng ta lại một lần nữa quay trở lại với Stack Frame... 🐐
Đầu tiên, khi hàm `main()` được gọi, stack fram lúc nay sẽ trông như sau:
| Địa chỉ | Tên | Giá trị |
|-------|-------|-------|
| 0x1aa1 | b | ->0x1aa0 |
| 0x1aa0 | a | 1990 |

`a` là một giá trị như chúng ta đã biết ở những ví dụ lần trước là một kiểu dữ liệu nguyên thủy (`primitive`) dạng 32-bit, `b` là một tham chiếu (reference) đến `a`, vì vậy giá trị của `b` sẽ là địa chỉ vùng nhớ mà giá trị `a` đang đứng ở trên Stack Frame, như ví dụ thì `b` sẽ có giá trị là `0x1aa0`.

Vậy điều gì sẽ xảy ra nếu hàm `take_ref()` được gọi và truyền vào một tham số `i`?
Lúc này stack frame của chúng ta sẽ như sau:

| Địa chỉ | Tên | Giá trị |
|-------|-------|-------|
| 0x1aa3 | x | 1991 |
| 0x1aa2 | i | ->0x1aa0 |
| 0x1aa1 | b | ->0x1aa0 |
| 0x1aa0 | a | 1990 |

## Kết luận

Phewphew! Như các bạn đã thấy ở stack frame trên, thì nó không chỉ chứa các giá trị cục bộ (local), mà cũng có thể chứa được các tham số truyền vào hàm.

Yooohooo! Chúng ta đã đi đến đoạn kết của bài viết ngày hôm nay, nếu như các bạn vẫn còn đọc những dòng này, thì hy vọng rằng các bạn sẽ tìm thấy một chút hữu ích từ bài viết của mình.

Bất kỳ câu hỏi nào hoặc yêu cầu giải thích về điều gì, các bạn có thể open issues tại [LINK](https://github.com/jimmymnt/blog-posts/issues), mình sẽ cố gắng hồi đáp nhanh nhất có thể.

