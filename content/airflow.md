# Apache Airflow

- [Airflow la gi](#airflow-la-gi)
- [Airflow giải quyết những vấn đề nào?](#)
- [Giải thích cách thiết kế workflow trong Airflow?](#)
- [Trình bày kiến trúc của Airflow và các thành phần của nó?](#)
- [Có những loại Executor nào trong Airflow?](#)
- [Ưu và nhược điểm của SequentialExecutor là gì?](#)
- [Ưu và nhược điểm của LocalExecutor là gì?](#)
- [Ưu và nhược điểm của CeleryExecutor là gì?](#)
- [Ưu và nhược điểm của KubernetesExecutor là gì?](#)
- [Cách định nghĩa một workflow trong Airflow?](#)
- [Làm thế nào để module có thể được Airflow nhận diện khi bạn sử dụng Docker Compose?](#)
- [Cách lên lịch DAG trong Airflow?](#)
- [XComs trong Airflow là gì?](#)
- [xcom_pull trong XCom Airflow là gì?](#)
- [Jinja templates là gì?](#)
- [Cách sử dụng Airflow XComs trong Jinja templates?](#)

# Airflow la gi

Apache Airflow là một nền tảng quản lý workflow mã nguồn mở. Nó được bắt đầu vào tháng 10 năm 2014 tại Airbnb như một giải pháp để quản lý các workflow ngày càng phức tạp của công ty. Việc Airbnb tạo ra Airflow cho phép họ lập trình tự động, lên lịch và giám sát các workflow thông qua giao diện người dùng Airflow tích hợp sẵn. Airflow là một công cụ điều phối workflow cho pipeline ETL (Extract, Transform, Load – Trích xuất, Biến đổi, Tải dữ liệu).

[Table of Contents](#)

# Airflow giải quyết những vấn đề gì?

Cron là một kỹ thuật lập lịch tác vụ cũ. Cron có khả năng mở rộng yêu cầu sự hỗ trợ từ bên ngoài để ghi lại, theo dõi và quản lý các tác vụ. Giao diện Airflow (Airflow UI) được sử dụng để theo dõi và giám sát việc thực thi workflow. Việc tạo và duy trì mối quan hệ giữa các tác vụ trong cron là một thách thức, trong khi việc này lại đơn giản khi viết mã Python trong Airflow. Các tác vụ Cron không thể tái tạo cho đến khi chúng được cấu hình bên ngoài. Airflow duy trì một bản ghi kiểm toán (audit trail) của tất cả các tác vụ đã hoàn thành.

[Table of Contents](#)

# Giải thích cách thiết kế workflow trong Airflow

Một đồ thị có hướng acyclic (DAG – Directed Acyclic Graph) được sử dụng để thiết kế workflow trong Airflow. Điều này có nghĩa là khi tạo workflow, bạn cần xem xét cách chia nhỏ nó thành các tác vụ có thể hoàn thành độc lập. Các tác vụ sau đó có thể được kết hợp thành một đồ thị để tạo thành một tổng thể logic. Logic tổng thể của workflow dựa trên hình dạng của đồ thị. Một DAG trong Airflow có thể có nhiều nhánh, và bạn có thể chọn nhánh nào để theo dõi hoặc nhánh nào bỏ qua trong quá trình thực thi workflow. Pipeline DAG trong Airflow có thể hoàn toàn dừng lại và các workflow có thể tiếp tục chạy từ tác vụ chưa hoàn thành cuối cùng. Điều quan trọng là phải nhớ rằng các toán tử (operator) trong Airflow có thể được chạy nhiều lần khi thiết kế, và mỗi tác vụ nên có tính idempotent, nghĩa là có thể thực hiện nhiều lần mà không gây ra hậu quả không mong muốn.

[Table of Contents](#)

# Giải thích Kiến trúc Airflow và các thành phần của nó

Airflow có bốn thành phần chính.

## Kiến trúc

- **Webserver**  
  - Đây là giao diện người dùng (UI) của Airflow được xây dựng trên Flask, cung cấp cái nhìn tổng quan về sức khỏe tổng thể của các DAG và giúp trực quan hóa các thành phần và trạng thái của từng DAG. Đối với thiết lập Airflow, Web Server cũng cho phép quản lý người dùng, vai trò và các cấu hình khác nhau.

- **Scheduler**  
  - Mỗi n giây, Scheduler sẽ duyệt các DAG và lập lịch cho các task được thực thi.

- **Executor**  
  - Executor là một thành phần nội bộ khác của Scheduler.  
  - Executor là các thành phần thực sự thực thi các task, trong khi Scheduler chịu trách nhiệm điều phối chúng. Airflow có nhiều loại Executor khác nhau, bao gồm SequentialExecutor, LocalExecutor, CeleryExecutor và KubernetesExecutor. Người dùng thường chọn Executor phù hợp nhất với trường hợp sử dụng của họ.

- **Worker**  
  - Worker chịu trách nhiệm thực thi các task mà Executor giao cho chúng.

- **Metadata Database**  
  - Airflow hỗ trợ nhiều loại cơ sở dữ liệu lưu trữ metadata. Cơ sở dữ liệu này chứa thông tin về các DAG, các lần chạy của chúng và các cấu hình khác của Airflow như người dùng, vai trò và kết nối. Trạng thái và các lần chạy của DAG được Web Server hiển thị từ cơ sở dữ liệu. Thông tin này cũng được Scheduler cập nhật vào cơ sở dữ liệu metadata.

[Table of Contents](#)

# Các loại Executor trong Airflow là gì?

Executor là các thành phần thực sự thực thi các task, trong khi Scheduler điều phối chúng. Airflow có các loại executor khác nhau, bao gồm SequentialExecutor, LocalExecutor, CeleryExecutor và KubernetesExecutor. Mọi người thường chọn executor phù hợp nhất với trường hợp sử dụng của họ. Các loại Executor:

## SequentialExecutor
- Chỉ một task được thực thi tại một thời điểm bởi SequentialExecutor. Cả scheduler và worker đều sử dụng cùng một máy.

## LocalExecutor
- LocalExecutor giống như Sequential Executor, ngoại trừ việc nó có thể chạy nhiều task cùng một lúc.

## CeleryExecutor
- Celery là một framework Python để chạy các task bất đồng bộ phân tán. Do đó, CeleryExecutor đã từ lâu là một phần của Airflow, thậm chí trước cả Kubernetes. CeleryExecutor có một số lượng worker cố định ở chế độ chờ để đảm nhận các task khi chúng trở nên khả dụng.

## KubernetesExecutor
- Mỗi task được chạy bởi KubernetesExecutor trong pod Kubernetes riêng của nó. Không giống như Celery, nó tạo các worker pod theo yêu cầu, cho phép sử dụng tài nguyên hiệu quả nhất.

[Table of Contents](#)

# Ưu và nhược điểm của SequentialExecutor là gì?

**Ưu điểm:**

- Đơn giản và dễ dàng để thiết lập.
- Là cách tốt để kiểm thử các DAG trong khi chúng đang được phát triển.

**Nhược điểm:** Nó không có khả năng mở rộng. Không thể thực hiện nhiều task cùng một lúc. Không phù hợp để sử dụng trong môi trường production.

[Table of Contents](#)

# Ưu và nhược điểm của LocalExecutor là gì?

**Ưu điểm:**

- Có khả năng thực hiện nhiều task.
- Có thể được sử dụng để chạy các DAG trong quá trình phát triển.

**Nhược điểm:**

- Sản phẩm không có khả năng mở rộng.
- Chỉ có một điểm lỗi duy nhất.
- Không phù hợp để sử dụng trong môi trường production.

[Table of Contents](#)

# Ưu và nhược điểm của CeleryExecutor là gì?

**Ưu điểm:**

- Cho phép khả năng mở rộng.
- Celery chịu trách nhiệm quản lý các worker. Celery tạo một worker mới trong trường hợp có lỗi xảy ra.

**Nhược điểm:**

- Celery yêu cầu RabbitMQ/Redis để xếp hàng task, điều này trùng lặp với những gì Airflow đã hỗ trợ sẵn.
- Việc thiết lập cũng phức tạp do các phụ thuộc được đề cập ở trên.

[Table of Contents](#)

# Ưu và nhược điểm của KubernetesExecutor là gì?

**Ưu điểm:**

- Kết hợp lợi ích của CeleryExecutor và LocalExecutor về khả năng mở rộng và tính đơn giản.
- Kiểm soát chi tiết các tài nguyên phân bổ cho task. Ở cấp độ task, lượng CPU/bộ nhớ cần thiết có thể được cấu hình.

**Nhược điểm:**

- Airflow còn mới với Kubernetes, và tài liệu khá phức tạp.

[Table of Contents](#)

# Làm thế nào để định nghĩa một workflow trong Airflow?

Các file Python được sử dụng để định nghĩa workflow. DAG (Directed Acyclic Graph - Đồ thị có hướng không chu trình). Class Python DAG trong Airflow cho phép bạn tạo một Directed Acyclic Graph, đây là biểu diễn của workflow.
```python
from Airflow.models import DAG
from airflow.utils.dates import days_ago

args = {
'start_date': days_ago(0),
}

dag = DAG(
dag_id='bash_operator_example',
default_args=args,
schedule_interval='* * * * *',
)
```
Bạn có thể sử dụng start date để khởi chạy một task vào một ngày cụ thể.
Schedule interval chỉ định tần suất mỗi workflow được lên lịch chạy. '* * * * *' cho biết các task phải chạy mỗi phút.

[Table of Contents](#)





