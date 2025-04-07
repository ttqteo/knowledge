***Mục tiêu**: Hiểu S3 là gì và cách nó vận hành tính năng cơ bản*
Lý thuyết
- [x] Amazon S3 là gì và trường hợp sử dụng (use cases)
- [x] Bucket và Object (cấu trúc cơ bản)
- [x] S3 storage classes (Standard, IA, Glacier, etc)
- [x] Cách đặt tên Bucket
- [ ] Tải lên/về tệp thông qua
	- [ ] AWS Console
	- [ ] AWS CLI
	- [ ] SDKs (như Python Boto3 hoặc JavaScript SDK)
Thực hành
- [ ] Tạo Bucket
- [ ] Tải lên / tải về / xoá object
- [ ] Tổ chức tệp bằng prefix (tiền tố, kiến trúc folder-like)
- [ ] Bật / tắt truy cập công khai

## Amazon S3 là gì?

- S3 - Simple Storage Service
- Dịch vụ lưu trữ đối tượng (object storage) do AWS cung cấp
	- File (ảnh, video, tài liệu, dữ liệu backup, log...) dưới dạng **object**
	- Object gồm
		- File (dữ liệu)
		- Metadata (thông tin phụ)
		- Key (khoá định danh)
	- Object lưu vào **Bucket** (như thư mục)
## Use cases

| Ứng dụng phổ biến                 | Mô tả                                                               |
| --------------------------------- | ------------------------------------------------------------------- |
| Lưu trữ ảnh, video, file          | Lưu trữ media cho web/mobile                                        |
| Hosting static web                | HTML/CSS/JS lên S3 như web                                          |
| Backup và Restore                 | Backup dữ liệu tự động (log, DB dump, snapshot...)                  |
| Data lake                         | Lưu dữ liệu lớn để xử lý bằng Big Data (Athena, Redshift, Spark...) |
| Chia sẻ file bảo mật              | Pre-signed URL                                                      |
| CDN - Phân phối nội dung toàn cầu | Kết hợp với CloudFront để làm CDN                                   |

> Dùng MinIO thay cho Amazon S3 để self-hosted

## Quy tắc đặt tên Bucket

1. Chỉ dùng chữ thường, số,  dấu gạch ngang và dấu chấm
--> không dùng chữ in hoa, ký tự đặc biệt và khoảng trắng
2. Độ dài 3 --> 63 ký tự
3. Không bắt đầu và kết thúc bằng dấu chấm và gạch ngang
4. Không dùng --
5. Không đặt như địa chỉ IP
6. Tên duy nhất
7. Nên: Dùng cấu trúc có tổ chức
```
<company-name>-<env>-<usecase>
ttqteo-user-avatar
```

Ví dụ

| Hợp lệ          | Không hợp lệ | Lỗi            |
| --------------- | ------------ | -------------- |
| my-bucket       | MyBucket     | chữ in hoa     |
| log-backup-2025 | log_backup   | gạch dưới      |
| data.site.name  | -mybucket    | bắt đầu bằng - |
| abc-xyz123      | abc..xyz     | .. liền nhau   |
| my-bucket-01    | 192.168.0.1  | như IP         |
Tips
* Nếu dùng Bucket với CloudFront (CDN) thì nên tránh dấu chấm vì dễ gây lỗi SSL/TLS (CNAME không khớp)

### Tạo Bucket hợp lệ bằng AWS CLI hoặc Python

1. AWS CLI
```sh
aws s3api create-bucket --bucket ttqteo-dev-user-avatar --region ap-southeast-1 --create-bucket-configuration LocationConstraint=ap-southeast-1
```
2. Python (Boto3)
```python
import boto3
from botocore.exceptions import ClientError

def create_bucket(bucket_name, region='ap-southeast-1'):
    try:
        s3 = boto3.client('s3', region_name=region)
        if region == 'us-east-1':
            response = s3.create_bucket(Bucket=bucket_name)
        else:
            response = s3.create_bucket(
                Bucket=bucket_name,
                CreateBucketConfiguration={'LocationConstraint': region}
            )
        print(f"✅ Bucket '{bucket_name}' created successfully.")
    except ClientError as e:
        print(f"❌ Error: {e}")

# Gọi hàm
create_bucket('ttqteo-dev-user-avatar')
```

Nhớ AWS credentials `~/.aws/credentials`
```sh
aws configure
```

### Tạo Bucket hợp lệ với MinIO

- http://localhost:9000
- access key: minioadmin
- secret key: minioadmin
```python
import boto3
from botocore.client import Config
from botocore.exceptions import ClientError

def create_minio_bucket(bucket_name):
    try:
        s3 = boto3.client(
            's3',
            endpoint_url='http://localhost:9000',
            aws_access_key_id='minioadmin',
            aws_secret_access_key='minioadmin',
            config=Config(signature_version='s3v4'),
            region_name='us-east-1'  # bắt buộc nhưng MinIO không thực sự dùng đến
        )

        s3.create_bucket(Bucket=bucket_name)
        print(f"✅ Bucket '{bucket_name}' created on MinIO.")
    except ClientError as e:
        print(f"❌ Error: {e}")

# Gọi hàm
create_minio_bucket('ttqteo-dev-user-avatar')
```

# Bucket và Object

## Bucket
- **Thư mục** chứa các file
- Tên duy nhất trong hệ thống (với Amazon S3 toàn cầu)
- Object PHẢI THUỘC Bucket
- Không thể tạo Object nếu không có Bucket
## Object
- Gồm:
	- Key (tên file/đường dẫn; ví dụ avatar/user1.jpg)
	- Data (dữ liệu từ file)
	- Metadata (dữ liệu mô tả: extension, thời gian tạo, quyền truy cập etc)
```txt
S3 (hoặc MinIO)
 └── Bucket: ttqteo-dev-images
       ├── Object: avatar/user1.jpg
       ├── Object: avatar/user2.jpg
       └── Object: document/manual.pdf
```

# Storage Class

Tóm tắt
- Avatar, download file -> Standard
- Monthly report, backup code -> Standard-IA
- System log -> Glacier / Deep Archive

>- IA = Infrequent Access = Ít truy cập
>- HA = High Availability = Độ bền cao
>- (1) (2) (3) là thứ tự ưu tiên hiểu biết

| Storage Class               | Đặc điểm                                                                                                                                                                                                              |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **S3 Standard** (1)         | - Truy cập thường xuyên, tính sẵn sàng cao<br>**--> Dữ liệu quan trọng hay được truy cập**<br>**--> Avatar, config file, used documents**<br>--> Độ sẵn sàng 99.99%<br>--> Độ bền 99.9999999%<br>--> Chi phí cao nhất |
| **S3 Standard-IA** (2)      | **--> Backup, logging**<br>--> Rẻ hơn standard nhưng **tốn phí mỗi lần truy cập**                                                                                                                                     |
| S3 One Zone-IA              | - Lưu 1 vùng duy nhất<br>--> không cần HA<br>--> AZ hỏng thì mất dữ liệu<br>--> phù hợp cache hoặc backup                                                                                                             |
| **S3 Glacier** (3)          | - Lưu trữ lạnh<br>**--> Lưu trữ policy, old document**<br>--> Phục hồi mất thời gian (vài phút đến và giờ)<br>--> Chi phí cực thấp, dành cho lưu trữ dài hạn                                                          |
| S3 Glacier Deep Archive     | - Cực kì rẻ, phục hồi 12-48h<br>--> Dữ liệu giữ lâu (vài năm)                                                                                                                                                         |
| S3 Intelligent-Tiering      | - Tự động chuyển class<br>--> Không rõ tần suất truy cập                                                                                                                                                              |
| S3 Reduced Redundancy (RRS) | Không dùng                                                                                                                                                                                                            |

> Nếu bạn làm hệ thống thật (upload/download file hoặc quản lý dữ liệu logs, report...), nên thiết lập [[S3 - Lifecycle Rule]] tự động chuyển object sang lớp rẻ hơn sau vài ngày không truy cập.

