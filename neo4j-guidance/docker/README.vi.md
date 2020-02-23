## Instructions for working with Neo4j database with docker
This guidance is used for neo4j version 3.5

## Hướng dẫn thao tác với Neo4j và docker

## Nội dung
* [Cài đặt](#cài-đặt)
* [Sao lưu](#sao-lưu)
* [Khôi phục](#khôi-phục)

## Cài đặt
### Các tham số cài đặt
Neo4j Docker image đã đi kèm cùng với một vài tham số cài đặt mặc định. Các tham số này được liệt kê tại [đây](https://github.com/neo4j/docker-neo4j/blob/master/docker-image-src/3.5/docker-entrypoint.sh).<br>
Neo4j Docker images sẽ expose mặc định 3 cổng sau cho việc truy cập từ xa:
* `7474` for HTTP
* `7473` for HTTPS
* `7687` for Bolt
Các cổng này sẽ được dùng để truy cập đến Neo4j trong container thông qua Neo4j browser hoặc các phương thức khác.

Có 3 cách để tuỳ chỉnh cấu hình trong Neo4j
* Cài đặt thông qua biến môi trường.
* Mount `/conf` volume.
* Build một image mới.

#### Cài đặt thông qua biến môi trường
Truyền biến môi trường khi tạo và chạy container. Ví dụ:
```
docker run \
    --detach \
    --publish=7474:7474 --publish=7687:7687 \
    --volume=$HOME/neo4j/data:/data \
    --volume=$HOME/neo4j/logs:/logs \
    --env NEO4J_dbms_memory_pagecache_size=4G \
    neo4j:3.5.9-enterprise
```
Với câu lệnh trên `NEO4J_dbms_memory_pagecache_size=4G` là một biến môi trường được truyền vào để thay đổi cấu hình `dbms.memory.pagecache.size` trong Neo4j. Các cấu hình khác([Configuration settings](https://neo4j.com/docs/operations-manual/current/reference/configuration-settings/)) có thể được truyền truyền tới container sử dụng quy ước sau:
* Prefix cấu hình cần thay đổi với NEO4J_.
* Dấu gạch dưới (underscore) phải được ghi 2 lần: _ sẽ chuyển thành __.
* Dấu chấm sẽ chuyển thành dấu gạch dưới: . chuyển thành _.
Ví dụ, cấu hình `dbms.tx_log.rotation.size` được truyền vào Docker container như sau:
```
--env NEO4J_dbms_tx__log_rotation_size
```
##### Neo4j Enterprise Edition
Những biến môi trường sau đây là dành riêng cho Causal Clustering và có sẵn trong Neo4j Enterprise Edition:
* `NEO4J_dbms_mode:` database mode, giá trị mặc định là `SINGLE`. Đặt là `CORE` hoặc `READ_REPLICA` trong trường hợp triển khai Causal Clustering.
* `NEO4J_causal__clustering_expected__core__cluster__size`: số lượng core trong cụm tại thời điểm bắt đầu triển khai cụm.
* `NEO4J_causal__clustering_initial__discovery__members`: Địa chỉ mạng của các core đầu tiên trong cụm.
* `NEO4J_causal__clustering_discovery__advertised__address`: Địa chỉ IP/domain và port cho quá trình phát hiện các thành phần trong cụm.
* `NEO4J_causal__clustering_transaction__advertised__address`: Địa chỉ IP/doamin và port để tham gia vào quá trình xử lý transaction
* `NEO4J_causal__clustering_raft__advertised__address`: Địa chỉ IP/domain và port cho quá trình giao tiếp trong cụm.
#### Mount a /conf volume
Để có thể chỉnh sửa nhiều cấu hình cùng một lúc, container có thể bind mount với volume `/conf`
```
docker run \
    --detach \
    --publish=7474:7474 --publish=7687:7687 \
    --volume=$HOME/neo4j/data:/data \
    --volume=$HOME/neo4j/logs:/logs \
    --volume=$HOME/neo4j/conf:/conf \
    neo4j:3.5.9-enterprise
```
Các file cấu hình trong thư mục `/conf` sẽ ghi đè cấu hình cung cấp bởi image. Vì vậy, nếu người dùng muốn thay đổi một giá trị trong một file thì cần phải đảm bảo rằng các giá trị còn lại là chính xác.
Biến môi trường truyền tới container sẽ ghi đè các cấu hình được cung cấp trong thư mục `/conf`.

#### Build một image mới
Để biết thêm chi tiết, xem [tài liệu](https://neo4j.com/docs/operations-manual/current/docker/configuration/#docker-new-image).

### Single Database
Để chạy một Neo4j instance trong một Docker container, chỉ cần thực hiện câu lệnh `docker run` với neo4j image kèm theo các tuỳ chọn và phiên bản. Một vài tuỳ chọn khi chạy câu lệnh `docker run` được mô tả như bảng dưới đây

| Tuỳ chọn | Mô tả | Ví dụ |
| --- | --- | --- |
| --name | Đặt tên cho container | `docker run --name name-of-container-what-you-want neo4j` |
| -p | Xác định cổng của container sẽ ánh xạ với cổng của host | `docker run -p7687:7687 neo4j` |
| -d | Chạy container ở chế độ background | `docker run -d neo4j` |
| -v | Bind mount một volume | `docker run -v $HOME/neo4j/data:/data neo4j` |
| --env | Cài đặt biến môi trường cho Neo4j database | `docker run --env NEO4J_AUTH=neo4j/test` |
| --help | Liệt kê các tuỳ chọn của câu lệnh `docker run` | `docker run --help` |

Mặc định, Neo4j yêu cầu người dùng đăng nhập lần đầu với tài khoản `neo4j/neo4j` và phải đặt password mới. Quá trình này có thể được bỏ qua bằng cách đặt password khi tạo Docker container sử dụng tuỳ chọn `--env NEO4J_AUTH=neo4j/<password>`
Ví dụ với câu lệnh dưới đây:
```
docker run \
    --name testneo4j \
    -p7474:7474 -p7687:7687 -p7473:7474 \
    -d \
    -v $HOME/neo4j/data:/data \
    -v $HOME/neo4j/logs:/logs \
    -v $HOME/neo4j/import:/import \
    -v $HOME/neo4j/plugins:/plugins \
    --env NEO4J_AUTH=neo4j/test \
    neo4j:latest
```
Trong câu lệnh trên:
* Tạo và khởi chạy một container có tên là `testneo4j`
* Tuỳ chọn `-d`, biểu thị rằng container này sẽ chạy background.
* Tuỳ chọn `-v`, biểu thị rằng các thư mục trong container sẽ được bind mount vào các thư mục local, do đó có thể truy cập nội dung các thư mục này từ máy host. (Để biết thêm thông tin về `bind mount`, tham khảo [tài liệu](https://docs.docker.com/storage/bind-mounts/)).
  - Dòng đầu tiên là bind mount cho thư mục /data - đây là nơi lưu chữ việc xác thực và quyền hạn cho mỗi database cũng như dữ liệu của mỗi database instance (trong thư mục graph.db)
  - Dòng thứ hai bind mount cho thư mục /logs. Logs của Neo4j sẽ được ghi ra thư mục host, cho phép người quản trị có thể phát hiện lỗi khi chạy Neo4j, thậm chí là container crash.
  - Dòng thứ ba bind mount thư mục /import. Các file csv có thể được sao chép vào đây cho việc thực hiện import trong Neo4j. Script cho việc thực hiện import cũng có thể được đặt vào thư mục này để thực hiện import.
  - Tuỳ chọn -v cuối cùng bind mount thư mục /plugins. Các extension, thư viện như Neo4j APOC hoặc các thư viện giải thuật đồ thị có thể được thêm vào đây (dưới dạng các file jars) để Neo4j trong Docker container có thể truy cập được.
* Tuỳ chọn `--env`. Dòng này đặt tài khoản để đăng nhập vào Neo4j instance với tên người dùng và mật khẩu. Tài khoản đăng nhập mặc định là `neo4j/neo4j`. Vì Neo4j sẽ yêu cầu người dùng thay đổi password trong lần đầu đăng nhập, do đó việc sử dụng tuỳ chọn này sẽ bỏ qua quá trình đó.
* Dòng cuối cùng của câu lệnh biểu thị rằng container sẽ chạy với Docker image `neo4j` và có phiên bản là `lastest`.
Khi chạy câu lệnh trên, nó sẽ tạo và khởi chạy một container<br>Chạy câu lệnh `docker ps` trên terminal để kiểm tra các container đang chạy.

### Cluster
Ví dụ dưới đây minh học việc cài đặt một cụm với 3 Core Server thông qua Docker.

#### Ví dụ
Trong ví dụ này, cụm sẽ được cài đặt với 3 Core Server với domain có tên lần lượt là: `core01.example.com`, `core02.example.com` and `core03.example.com`.<br>
Chạy câu lệnh sau đối với mỗi server
```
docker run --name=$NAME_OF_CONTAINER --detach \
  --network=host \
  --env NEO4J_dbms_mode=CORE \
  --env NEO4J_causal__clustering_expected__core__cluster__size=$EXPECTED_CORE_CLUSTER_SIZE \
  --env NEO4J_causal__clustering_initial__discovery__members=$INITIAL_CORE_MEMBERS \
  --env NEO4J_causal__clustering_discovery__advertised__address=$ADDRESS:5000 \
  --env NEO4J_causal__clustering_transaction__advertised__address=$ADDRESS:6000 \
  --env NEO4J_causal__clustering_raft__advertised__address=$ADDRESS:7000 \
  --env NEO4J_dbms_connectors_default__advertised__address=$ADDRESS \
  --env NEO4J_dbms_connector_bolt_advertised__address=$ADDRESS:7687 \
  --env NEO4J_dbms_connector_http_advertised__address=$ADDRESS:7474 \
  --env NEO4J_ACCEPT_LICENSE_AGREEMENT=yes \
  neo4j:enterprise
```
Trong đó:
* `$NAME_OF_CONTAINER` là tên của container.
* `$EXPECTED_CORE_CLUSTER_SIZE` là số lượng core mong đợi sẽ có trong cụm. Giá trị mặc định là 3, giá trị nhỏ nhất là 2. Để biết thêm chi tiết, xem [configuration-settings](https://neo4j.com/docs/operations-manual/current/reference/configuration-settings/).
* `$ADDRESS` là địa chỉ IP hoặc domain của mỗi server, trong ví dụ này, domain lần lượt là `core01.example.com`, `core02.example.com` and `core03.example.com`

Sau khi cluster khởi chạy thành công, chạy câu lệnh `sysinfo` trên [neo4j browser](http://localhost:7474) để kiểm tra trạng thái của cluser.

## Sao lưu
Các tham số cấu hình thực hiện sao lưu, vui lòng tham khảo phần `common/README.vi.md`

Có 3 tình huống thực hiện sao lưu khi sử dụng với docker.
1. Backup server chạy trên docker, backup client chạy trên docker.
2. Backup server chạy trên docker, backup client chạy trên server thông thường.
3. Backup server chạy trên máy tính thông thường, backup client chạy trên docker.

Quy trình thực hiện sao lưu khi sử dụng với docker gồm 3 bước:
1. Tạo một volume trong docker bằng cách sử dụng câu lệnh sau: (ví dụ `neo4j-backups`)
```
docker volume create neo4j-backups
```
2. Trong tất cả các trường hợp, backup server đều phải được cài đặt 2 cấu hình:
```
dbms.backup.enabled=true
dbms.backup.address=0.0.0.0:6362
```
Để cài đặt cấu hình cho Neo4j chạy trên docker, tham khảo chương [Các tham số cài đặt](#các-tham-số-cài-đặt)

3. Tham khảo `common/README.vi.md` trong trường hợp backup client chạy trên server thông thường.<br>
Thực hiện câu lệnh sau trên terminal đối với trường hợp backup client chạy trên docker
```
docker run --name=neo4j-backup -d \
  -v neo4j-backups:/data \
  -v $HOME/neo4j/logs:/logs \
  -v $HOME/neo4j/import:/import \
  -v $HOME/neo4j/plugins:/plugins \
  --env NEO4J_AUTH=neo4j/test \
  --env NEO4J_ACCEPT_LICENSE_AGREEMENT=yes \
  neo4j:enterprise /bin/bash -c "neo4j-admin backup --from=backup-server@example.com \
    --backup-dir=/data --name=graph.db-2020/02/03"
```
Câu lệnh trên gồm 2 phần:
* Phần khởi tạo container:
  - Tạo và khởi chạy một container có tên là `neo4j-backup`
  - Tuỳ chọn `-d`, biểu thị rằng container này sẽ chạy background.
  - Tuỳ chọn `-v`, biểu thị rằng các thư mục trong container sẽ được mount/bind mount vào các thư mục local, do đó có thể truy cập nội dung các thư mục này từ máy host. (Để biết thêm thông tin về `bind mount`, tham khảo [tài liệu](https://docs.docker.com/storage/bind-mounts/), `mount/volume`, tham khảo [tài liệu](https://docs.docker.com/storage/volumes/)).
    - Dòng đầu tiên là bind mount cho thư mục `/data` - đây là nơi lưu chữ việc xác thực và quyền hạn cho mỗi database cũng như dữ liệu của mỗi database instance (trong thư mục graph.db)
    - Dòng thứ hai bind mount cho thư mục `/logs`. Logs của Neo4j sẽ được ghi ra thư mục host, cho phép người quản trị có thể phát hiện lỗi khi chạy Neo4j, thậm chí là container crash.
    - Dòng thứ ba bind mount thư mục `/import`. Các file csv có thể được sao chép vào đây cho việc thực hiện import trong Neo4j. Script cho việc thực hiện import cũng có thể được đặt vào thư mục này để thực hiện import.
    - Tuỳ chọn -v cuối cùng bind mount thư mục `/plugins`. Các extension, thư viện như Neo4j APOC hoặc các thư viện giải thuật đồ thị có thể được thêm vào đây (dưới dạng các file jars) để Neo4j trong Docker container có thể truy cập được.
  - Tuỳ chọn `--env`. Dòng này đặt tài khoản để đăng nhập vào Neo4j instance với tên người dùng và mật khẩu. Tài khoản đăng nhập mặc định là `neo4j/neo4j`. Vì Neo4j sẽ yêu cầu người dùng thay đổi password trong lần đầu đăng nhập, do đó việc sử dụng tuỳ chọn này sẽ bỏ qua quá trình đó.
  - Dòng cuối cùng của câu lệnh với nội dung: `neo4j:enterprise` rằng container sẽ chạy với Docker image `neo4j` và có phiên bản là `enterprise`. Phần còn lại của dòng này liên quan đến việc khởi chạy container.
* Phần khởi chạy container: Mục tiêu của việc khởi chạy container này sao lưu dữ liệu từ một backup server (container này đóng vai trò backup client) do đó, cần phải cung cấp câu lệnh thực hiện sao lưu để container này thực hiện. Nội dung của câu lệnh này là:
```
bin/bash -c "neo4j-admin backup --from=$ADDRESS \
--backup-dir=/data --name=$NAME
```
  - `bin/bash -c` biểu thị việc thực hiện câu lệnh trong môi trường bash/shell của container.
  - `neo4j-admin backup --from=backup-server@example.com --backup-dir=/data --name=graph.db-2020/02/03` là câu lệnh thực hiện sao lưu online của Neo4j.
    - Tuỳ chọn `--from=backup-server@example.com` xác định địa chỉ backup server là máy tính có domain là `backup-server@example.com`.
    - Tuỳ chọn `--backup-dir=/data` chỉ ra thư mục sẽ lưu dữ liệu sao lưu trên container này là `/data` (do đã mount thư mục `/data` với volume `neo4j-backups` nên dữ liệu được sao lưu ra thư mục `/data` cũng sẽ có trong volume `neo4j-backups`)
    - Tuỳ chọn `--name=grpah.db-2020/02/03` chỉ ra tên của bản sao lưu này.

## Khôi phục
**Yêu cầu**: phải chuẩn bị sẵn một volume chứa dữ liệu sao lưu. Ví dụ volume `neo4j-backups`  đã tồn tại và đã chứa dữ liệu sao lưu<br>
Thực hiện câu lệnh sau trên terminal:
```
docker run --name neo4j -d \
  -p7474:7474 -p7687:7687 -p7473:7473 \
  -v neo4j-backups:/backups:ro \
  -v $HOME/neo4j/data:/data \
  -v$HOME/neo4j/logs:/logs \
  -v $HOME/neo4j/import:/import \
  -v $HOME/neo4j/plugins:/plugins \
  --env NEO4J_AUTH=neo4j/test \
  neo4j:enterprise /bin/bash -c "neo4j-admin restore \
  --from=/backups/ --database=graph.db-2020/02/03 --force; \
  neo4j console"
```
Câu lệnh trên gồm 2 phần:
* Phần khởi tạo container:
  - Tạo và khởi chạy một container có tên là `neo4j`
  - Tuỳ chọn `-d`, biểu thị rằng container này sẽ chạy background.
  - Tuỳ chọn `-v`, biểu thị rằng các thư mục trong container sẽ được mount/bind mount vào các thư mục local, do đó có thể truy cập nội dung các thư mục này từ máy host. (Để biết thêm thông tin về `bind mount`, tham khảo [tài liệu](https://docs.docker.com/storage/bind-mounts/), `mount/volume`, tham khảo [tài liệu](https://docs.docker.com/storage/volumes/)).
    - Dòng thứ nhất là mount thư mục `/backups` trong container với volume `neo4j-backups`, điều này cho phép container này có thể nhận được dữ liệu sao lưu trong volume `neo4j-backups`.
    - Dòng thứ hai là bind mount cho thư mục `/data` - đây là nơi lưu chữ việc xác thực và quyền hạn cho mỗi database cũng như dữ liệu của mỗi database instance (trong thư mục graph.db)
    - Dòng thứ ba bind mount cho thư mục `/logs`. Logs của Neo4j sẽ được ghi ra thư mục host, cho phép người quản trị có thể phát hiện lỗi khi chạy Neo4j, thậm chí là container crash.
    - Dòng thứ tư bind mount thư mục `/import`. Các file csv có thể được sao chép vào đây cho việc thực hiện import trong Neo4j. Script cho việc thực hiện import cũng có thể được đặt vào thư mục này để thực hiện import.
    - Tuỳ chọn -v cuối cùng bind mount thư mục `/plugins`. Các extension, thư viện như Neo4j APOC hoặc các thư viện giải thuật đồ thị có thể được thêm vào đây (dưới dạng các file jars) để Neo4j trong Docker container có thể truy cập được.
  - Tuỳ chọn `--env`. Dòng này đặt tài khoản để đăng nhập vào Neo4j instance với tên người dùng và mật khẩu. Tài khoản đăng nhập mặc định là `neo4j/neo4j`. Vì Neo4j sẽ yêu cầu người dùng thay đổi password trong lần đầu đăng nhập, do đó việc sử dụng tuỳ chọn này sẽ bỏ qua quá trình đó.
  - Dòng cuối cùng của câu lệnh với nội dung: `neo4j:enterprise` rằng container sẽ chạy với Docker image `neo4j` và có phiên bản là `enterprise`. Phần còn lại của dòng này liên quan đến việc khởi chạy container.
* Phần khởi chạy container: Mục tiêu của việc khởi chạy container này khôi phục dữ liệu từ dữ liệu sao lưu. Do đó, khi khởi chạy, container này sẽ phải thực hiện 2 quá trình: 1. Khôi phục dữ liệu từ dữ liệu sao lưu; 2. Khởi chạy Neo4j instance với dữ liệu đã được khôi phục. Câu lệnh để thực hiện 2 nhiệm vụ trên như sau:
```
/bin/bash -c "neo4j-admin restore \
--from=/backups/ --database=$NAME --force;
neo4j console \
```
  - `bin/bash -c` biểu thị việc thực hiện câu lệnh trong môi trường bash/shell của container.
  - `neo4j-admin restore --from=/backups/ --database=$graph.db --force` là câu lệnh thực hiện sao lưu online của Neo4j.
    - Tuỳ chọn `--from=/backups/` chỉ ra đường dẫn tới thư mục chứa dữ liệu là `/backups` - đây là thư mục đã được mount với volume chứa dữ liệu sao lưu.
    - Tuỳ chọn `--database=graph.db-2020/02/03` là tên của dữ liệu sao lưu.
  - `neo4j console` là câu lệnh khởi chạy Neo4j.