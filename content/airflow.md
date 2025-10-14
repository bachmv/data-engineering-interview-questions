# Apache Airflow

- [Giới thiệu Airflow](#giới-thiệu-airflow)
- [Những vấn đề Airflow giải quyết](#những-vấn-đề-airflow-giải-quyết)
- [Giải thích cách thiết kế workflow trong Airflow](#giải-thích-cách-thiết-kế-workflow-trong-airflow)
- [Kiến trúc Airflow và các thành phần của nó](#Kiến-trúc-airflow-và-các-thành-phần-của-nó)
- [Phân loại Executor trong Airflow](#phân-loại-executor-trong-airflow)
- [Ưu và nhược điểm của SequentialExecutor](#ưu-và-nhược-điểm-của-sequentialexecutor)
- [Ưu và nhược điểm của LocalExecutor](#ưu-và-nhược-điểm-của-localexecutor)
- [Ưu và nhược điểm của CeleryExecutor](#ưu-và-nhược-điểm-của-celeryexecutor)
- [Ưu và nhược điểm của KubernetesExecutor](#ưu-và-nhược-điểm-của-kubernetesexecutor)
- [Cách định nghĩa một workflow trong Airflow](#cách-định-nghĩa-một-workflow-trong-airflow)
- [Làm thế nào để module có thể được Airflow nhận diện khi bạn sử dụng Docker Compose](#làm-thế-nào-để-module-có-thể-được-airflow-nhận-diện-khi-bạn-sử-dụng-docker-compose)
- [Cách lên lịch DAG trong Airflow](#cách-lên-lịch-dag-trong-airflow)
- [XComs trong Airflow](#xcoms-trong-airflow)
- [xcom_pull trong XCom Airflow](#xcom_pull-trong-xcom-airflow)
- [Jinja templates](#jinja-templates)
- [Cách sử dụng Airflow XComs trong Jinja templates](#cách-sử-dụng-airflow-xcoms-trong-jinja-templates)

# Giới thiệu Airflow

Apache Airflow là một nền tảng quản lý workflow mã nguồn mở. Nó được bắt đầu vào tháng 10 năm 2014 tại Airbnb như một giải pháp để quản lý các workflow ngày càng phức tạp của công ty. Việc Airbnb tạo ra Airflow cho phép họ lập trình tự động, lên lịch và giám sát các workflow thông qua giao diện người dùng Airflow tích hợp sẵn. Airflow là một công cụ điều phối workflow cho pipeline ETL (Extract, Transform, Load – Trích xuất, Biến đổi, Tải dữ liệu).

[Table of Contents](#apache-airflow)

# Những vấn đề Airflow giải quyết

Cron là một kỹ thuật lập lịch tác vụ cũ. Cron có khả năng mở rộng yêu cầu sự hỗ trợ từ bên ngoài để ghi lại, theo dõi và quản lý các tác vụ. Giao diện Airflow (Airflow UI) được sử dụng để theo dõi và giám sát việc thực thi workflow. Việc tạo và duy trì mối quan hệ giữa các tác vụ trong cron là một thách thức, trong khi việc này lại đơn giản khi viết mã Python trong Airflow. Các tác vụ Cron không thể tái tạo cho đến khi chúng được cấu hình bên ngoài. Airflow duy trì một bản ghi kiểm toán (audit trail) của tất cả các tác vụ đã hoàn thành.

[Table of Contents](#apache-airflow)

# Giải thích cách thiết kế workflow trong Airflow

Một đồ thị có hướng acyclic (DAG – Directed Acyclic Graph) được sử dụng để thiết kế workflow trong Airflow. Điều này có nghĩa là khi tạo workflow, bạn cần xem xét cách chia nhỏ nó thành các tác vụ có thể hoàn thành độc lập. Các tác vụ sau đó có thể được kết hợp thành một đồ thị để tạo thành một tổng thể logic. Logic tổng thể của workflow dựa trên hình dạng của đồ thị. Một DAG trong Airflow có thể có nhiều nhánh, và bạn có thể chọn nhánh nào để theo dõi hoặc nhánh nào bỏ qua trong quá trình thực thi workflow. Pipeline DAG trong Airflow có thể hoàn toàn dừng lại và các workflow có thể tiếp tục chạy từ tác vụ chưa hoàn thành cuối cùng. Điều quan trọng là phải nhớ rằng các toán tử (operator) trong Airflow có thể được chạy nhiều lần khi thiết kế, và mỗi tác vụ nên có tính idempotent, nghĩa là có thể thực hiện nhiều lần mà không gây ra hậu quả không mong muốn.

[Table of Contents](#apache-airflow)

# Kiến trúc Airflow và các thành phần của nó

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

[Table of Contents](#apache-airflow)

# Phân loại Executor trong Airflow

Executor là các thành phần thực sự thực thi các task, trong khi Scheduler điều phối chúng. Airflow có các loại executor khác nhau, bao gồm SequentialExecutor, LocalExecutor, CeleryExecutor và KubernetesExecutor. Mọi người thường chọn executor phù hợp nhất với trường hợp sử dụng của họ. Các loại Executor:

## SequentialExecutor
- Chỉ một task được thực thi tại một thời điểm bởi SequentialExecutor. Cả scheduler và worker đều sử dụng cùng một máy.

## LocalExecutor
- LocalExecutor giống như Sequential Executor, ngoại trừ việc nó có thể chạy nhiều task cùng một lúc.

## CeleryExecutor
- Celery là một framework Python để chạy các task bất đồng bộ phân tán. Do đó, CeleryExecutor đã từ lâu là một phần của Airflow, thậm chí trước cả Kubernetes. CeleryExecutor có một số lượng worker cố định ở chế độ chờ để đảm nhận các task khi chúng trở nên khả dụng.

## KubernetesExecutor
- Mỗi task được chạy bởi KubernetesExecutor trong pod Kubernetes riêng của nó. Không giống như Celery, nó tạo các worker pod theo yêu cầu, cho phép sử dụng tài nguyên hiệu quả nhất.

[Table of Contents](#apache-airflow)

# Ưu và nhược điểm của SequentialExecutor 

**Ưu điểm:**

- Đơn giản và dễ dàng để thiết lập.
- Là cách tốt để kiểm thử các DAG trong khi chúng đang được phát triển.

**Nhược điểm:** Nó không có khả năng mở rộng. Không thể thực hiện nhiều task cùng một lúc. Không phù hợp để sử dụng trong môi trường production.

[Table of Contents](#apache-airflow)

# Ưu và nhược điểm của LocalExecutor 

**Ưu điểm:**

- Có khả năng thực hiện nhiều task.
- Có thể được sử dụng để chạy các DAG trong quá trình phát triển.

**Nhược điểm:**

- Sản phẩm không có khả năng mở rộng.
- Chỉ có một điểm lỗi duy nhất.
- Không phù hợp để sử dụng trong môi trường production.

[Table of Contents](#apache-airflow)

# Ưu và nhược điểm của CeleryExecutor

**Ưu điểm:**

- Cho phép khả năng mở rộng.
- Celery chịu trách nhiệm quản lý các worker. Celery tạo một worker mới trong trường hợp có lỗi xảy ra.

**Nhược điểm:**

- Celery yêu cầu RabbitMQ/Redis để xếp hàng task, điều này trùng lặp với những gì Airflow đã hỗ trợ sẵn.
- Việc thiết lập cũng phức tạp do các phụ thuộc được đề cập ở trên.

[Table of Contents](#apache-airflow)

# Ưu và nhược điểm của KubernetesExecutor 

**Ưu điểm:**

- Kết hợp lợi ích của CeleryExecutor và LocalExecutor về khả năng mở rộng và tính đơn giản.
- Kiểm soát chi tiết các tài nguyên phân bổ cho task. Ở cấp độ task, lượng CPU/bộ nhớ cần thiết có thể được cấu hình.

**Nhược điểm:**

- Airflow còn mới với Kubernetes, và tài liệu khá phức tạp.

[Table of Contents](#apache-airflow)

# Cách định nghĩa một workflow trong Airflow

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

Biểu thức `schedule_interval='30 8 * * 1-5'` là một **biểu thức cron** được sử dụng trong Airflow (và các hệ thống giống Unix) để định nghĩa lịch trình cụ thể cho việc chạy các tác vụ. Dưới đây là phân tích chi tiết:

## Cấu trúc Biểu thức Cron

Một biểu thức cron bao gồm 5 trường được phân tách bởi dấu cách:

| Trường | Vị trí | Giá trị cho phép | Mô tả |
|--------|--------|------------------|-------|
| Phút | 1 | 0-59 | Phút trong giờ |
| Giờ | 2 | 0-23 | Giờ trong ngày |
| Ngày trong tháng | 3 | 1-31 | Ngày của tháng |
| Tháng | 4 | 1-12 hoặc JAN-DEC | Tháng |
| Ngày trong tuần | 5 | 0-6 hoặc SUN-SAT | Ngày trong tuần (0 = Chủ nhật) |

## Giải thích Chi tiết về `30 8 * * 1-5`

1. **30** (Phút):
   - Tác vụ sẽ chạy vào **phút thứ 30** của giờ.
   - Ví dụ: Nếu giờ là 8, tác vụ sẽ thực thi vào lúc 08:30.

2. **8** (Giờ):
   - Tác vụ sẽ chạy trong **giờ thứ 8 trong ngày**.
   - Ví dụ: Nó sẽ thực thi vào lúc 08:30 AM.

3. **\*** (Ngày trong tháng):
   - Dấu sao có nghĩa là **"mọi ngày"**.
   - Tác vụ sẽ chạy vào bất kỳ ngày nào trong tháng.

4. **\*** (Tháng):
   - Dấu sao có nghĩa là **"mọi tháng"**.
   - Tác vụ sẽ chạy vào tất cả các tháng trong năm.

5. **1-5** (Ngày trong tuần):
   - Tác vụ sẽ chạy từ **Thứ Hai đến Thứ Sáu** (1 = Thứ Hai, 5 = Thứ Sáu).
   - Nó sẽ **không chạy vào cuối tuần** (Thứ Bảy và Chủ nhật).
  
## Trường hợp Sử dụng Thực tế

Bạn có thể sử dụng lịch trình này cho các tác vụ chỉ nên chạy trong giờ làm việc vào các ngày trong tuần, chẳng hạn như:

- Gửi báo cáo hàng ngày cho nhóm.
- Cập nhật cơ sở dữ liệu với dữ liệu từ ngày hôm trước.
- Chạy các pipeline dữ liệu trong thời gian không cao điểm.

[Table of Contents](#apache-airflow)

# Làm thế nào để module có thể được Airflow nhận diện khi bạn sử dụng Docker Compose

Nếu chúng ta đang sử dụng Docker Compose, thì chúng ta sẽ cần sử dụng một image tùy chỉnh với các dependency bổ sung của riêng mình để làm cho module có sẵn cho Airflow.

[Table of Contents](#apache-airflow)

# Cách lên lịch DAG trong Airflow

DAG có thể được lên lịch bằng cách truyền một timedelta hoặc một biểu thức cron (hoặc một trong các preset @), điều này hoạt động đủ tốt cho các DAG cần chạy theo định kỳ, nhưng có nhiều trường hợp sử dụng khác hiện đang khó thể hiện "tự nhiên" trong Airflow, hoặc yêu cầu một số giải pháp phức tạp. Bạn có thể tham khảo Airflow Improvements Proposals (AIP). Chỉ cần sử dụng lệnh sau để khởi động scheduler:

- `airflow scheduler`

[Table of Contents](#apache-airflow)

# XComs trong Airflow

XCom (viết tắt của cross-communication) là các thông điệp cho phép dữ liệu được gửi giữa các task. Key, value, timestamp và task/DAG id đều được định nghĩa.

[Table of Contents](#apache-airflow)

# xcom_pull trong XCom Airflow 

Các phương thức xcom push và xcom pull trên Task Instances được sử dụng để tường minh "push" và "pull" XCom đến và từ storage của chúng. Trong khi nếu tham số do_xcom_push được đặt thành True (như mặc định), nhiều operator và sensor sẽ tự động push kết quả của chúng vào một XCom key được đặt tên là return_value. Nếu không có key nào được cung cấp cho xcom pull, nó sẽ sử dụng key này theo mặc định, cho phép bạn viết code như thế này: Pull return_value XCOM từ "pushing_task" value = task_instance.xcom_pull(task_ids='pushing_task')

[Table of Contents](#apache-airflow)

# Jinja templates

Jinja là một template engine nhanh, biểu cảm và có thể mở rộng. Template có các placeholder đặc biệt cho phép bạn viết code trông giống như cú pháp Python. Sau đó, dữ liệu được truyền vào template để render ra tài liệu cuối cùng.

[Table of Contents](#apache-airflow)

# Cách sử dụng Airflow XComs trong Jinja templates

Chúng ta có thể sử dụng XComs trong Jinja templates như sau:

- `SELECT * FROM {{ task_instance.xcom_pull(task_ids='foo', key='table_name') }}`

[Table of Contents](#apache-airflow)






