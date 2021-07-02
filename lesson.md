









[TOC]















---













📽️ - посмотри воркшоу

⚗️ - проведи эксперимент

🔬 - изучи внимательно

📖 - прочитай документация

🪙 - подумай о сложности

🐞 - запомни ошибку

🔨 - запомни решение

🏔️ - обойди камень предкновенья

⏰ - сделай перерыв

🏡 - попробуй дома

💡 - обсуди светлые идеи

🙋 - задай вопрос

🌩️ - запомни панику















---























## Protobuf 🙋























---

### Proto файл

```protobuf
syntax = "proto3";

option go_package = "github.com/ozoncp/ocp-task-api/pkg/ocp-task-api;ocp_task_api";

package ocp.task.api;

// Описание задачи
message Task {
    uint64 task_id = 1;
    string description = 2;
    
    string comment = 20;
}
```

**syntax** определяет версию proto-спецификации

- 3
- 2 (default)

**package** - наименование пакета, пространство имен



**// comment** - используются в документации



 **;** - часто встречаемая ошибка 🐞 - забыл поставить ;



**option** - для разных случаев, можно и свои (полный список [тут](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/descriptor.proto))

- `string description = 2 [deprecated = true]`; 
- `option optimize_for = SPEED, CODE_SIZE, LITE_RUNTIME`



Расположение:

- **./api** директория для proto файлов, описывающих API сервисов

- **./vendor.protogen** директория для вендоринга proto файлов, включая proto файлы из директории **api**

---

### Service

```protobuf
// ...

// Cервис для работы с задачами
service OcpTaskApi {
    // Возвращает описание задачи по ее идентификатору
    rpc DescribeTask(DescribeTaskRequest) returns (DescribeTaskResponse) {
    }
}

message DescribeTaskRequest {
		uint64 task_id = 1;
		bool verbose = 2;
}

message DescribeTaskResponse {
	Task task = 1;
	Error error 
}
```

Наименования сервисов

Наименование R**P**C ручек - **verb** + **subject**

Множественное определение сервисов 🏔️

Потоковое передача данных: когда и зачем ?

Общие типы запросов и ответов 🏔️

Порядок расположения сообщений и сервисов 💡

---

### Генерация

Подготовка окружения

```sh
➜ brew install protobuf
➜ which protoc
➜ protoc --version

➜ export GO111MODULE=on
➜ go get google.golang.org/protobuf/cmd/protoc-gen-go \
         google.golang.org/grpc/cmd/protoc-gen-go-grpc
         
# export PATH="$PATH:$(go env GOPATH)/bin"

➜ tmpdir=$(mktemp -d); cd $tmpdir && go mod init temp && go get -d github.com/envoyproxy/protoc-gen-validate@v0.1.0 \
&& go build -o $GOBIN/protoc-gen-validate $GOPATH/src/github.com/envoyproxy/protoc-gen-validate/main.go && cd -
```

Генерация кода из proto-файлов

```sh
➜ protoc -I vendor.protogen \
				--go_out=pkg/ocp-task-api --go_opt=paths=import \
				--go-grpc_out=pkg/ocp-task-api --go-grpc_opt=paths=import \
				api/ocp-task-api/ocp-task-api.proto
				
➜ ls -l pkg/ocp-task-api

ocp-task-api.pb.go      # types
ocp-task-api_grpc.pb.go # grpc
```

---

### Имплементация обработчика

Добавим в internal/api/api.go код обработки запроса для ручки **DescribeTaskV1**:

```go
package api

import (
	"context"

	desc "github.com/ozoncp/ocp-task-api/pkg/ocp-task-api"

	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
)

const (
	errTaskNotFound = "task not found"
)

type api struct {
	desc.UnimplementedOcpTaskApiServer
}

func (a *api) DescribeTask(
	ctx context.Context,
	req *desc.DescribeTaskRequest,
) (*desc.DescribeTaskResponse, error) {

	err := status.Error(codes.NotFound, errTaskNotFound)
	return nil, err
}

func NewOcpTaskApi() desc.OcpTaskApiServer {
	return &api{}
}
```

---

### Запуск сервиса

```go
package main

import (
	"log"
	"net"

	"google.golang.org/grpc"

	api "github.com/ozoncp/ocp-task-api/internal/api"
	desc "github.com/ozoncp/ocp-task-api/pkg/ocp-task-api"
)


const (
	grpcPort = ":82"
)

func run() error {
	listen, err := net.Listen("tcp", grpcPort)
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	s := grpc.NewServer()
	desc.RegisterOcpTaskApiServer(s, api.NewOcpTaskApi())

	if err := s.Serve(listen); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}

	return nil
}

```



---

### Gateway

```protobuf
syntax = "proto3";

import "google/api/annotations.proto";

service OcpTaskApi {
    // Возвращает описание задачи по ее идентификатору
    rpc DescribeTask(DescribeTaskRequest) returns (DescribeTaskResponse) {
        option (google.api.http) = {
            post: "/tasks/{task_id}"
            body: *
        };
    }
}
```

```sh
➜ protoc -I vendor.protogen \
  --go_out=pkg/ocp-task-api --go_opt=paths=import \
  --go-grpc_out=pkg/ocp-task-api --go-grpc_opt=paths=import \
  --grpc-gateway_out=pkg/ocp-task-api \
  --grpc-gateway_opt=logtostderr=true \
  --grpc-gateway_opt=paths=import \
  api/ocp-task-api/ocp-task-api.proto
```



task_id

что берется из query **?** 🙋

что берется из url **?** 🙋





---

### Версионирование

| Before               | After                      |
| -------------------- | -------------------------- |
| DescribeTask         | DescribeTask**V1**         |
| DescribeTaskRequest  | DescribeTask**V1**Request  |
| DescribeTaskResponse | DescribeTask**V1**Response |
| "/tasks/{task_id}"   | "**/v1**/tasks/{task_id}"  |

> #обычные-сообщения #cовместное-обновление




Добавление новой версии ручки

- Задеплоить релиз с новой версией ручки
- Перевести всех нужных клиентов на новую версию



Переход на новую версию ручки

- Пометить, как deprecated
- Задеплоить релиз с новой версией ручки
- Перевести всех клиентов на новую версию
- Задеплоить релиз без старой версии ручки, если не планируется поддерживать



**internal/models** 

- единая кодовая база

- отображение модели данных в БД

---

### Деплой пакета

```sh
➜ cd pkg/ocp-task-api/
➜ go mod init && go mod tidy
➜ git tag pkg/ocp-task-api/v0.1.0 # v1.0.0
➜ git push origin --tags

➜ z ocp-facade-api 
➜ go get github.com/ozoncp/ocp-task-api # v1.0.0
```

```go
import ( 
  taskApi "github.com/ozoncp/ocp-task-api/pkg/ocp-task-api"
  
  "google.golang.org/grpc"
)

conn, err := grpc.Dial(taskApiAddress, grpc.WithInsecure())

client := taskApi.NewOcpTaskApiClient(conn)

req := &taskApi.DescribeTaskV1Request{
	TaskId: taskId,
}

resp, err := client.DescribeTaskV1(ctx, req)
```

- Использование replace для отладки
- Использование go get для обновления
- Версионирование **semver**
- Генерация пакетов на других языках
  - Пример
  - Совпадение версий

- Максимальный размер передаваемых данных **?**
- Время ожидания ответа по умолчанию **?**

---

### Валидация

```protobuf
// ...
import "github.com/envoyproxy/protoc-gen-validate/validate/validate.proto";
// ...

message DescribeTaskRequest {
		uint64 task_id = 1 [(validate.rules).uint64.gt = 0];
}
```

```sh
➜ protoc -I vendor.protogen \
  --go_out=pkg/ocp-task-api --go_opt=paths=import \
  --go-grpc_out=pkg/ocp-task-api --go-grpc_opt=paths=import \
  --grpc-gateway_out=pkg/ocp-task-api \
  --grpc-gateway_opt=logtostderr=true \
  --grpc-gateway_opt=paths=import \
  --validate_out lang=go:pkg/ocp-task-api \ # +
  api/ocp-task-api/ocp-task-api.proto
```

```sh
➜ ls -l pkg/ocp-task-api
ocp-task-api.pb.go
ocp-task-api.pb.gw.go
ocp-task-api.pb.validate.go # new file
ocp-task-api_grpc.pb.go 
```

- Инсталирование плагина
- Генерация кода
- Добавление проверки в обработчики
- Добавление проверки в вызывающий код

```go
import (
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
)

if err := req.Validate(); err != nil {
		return nil, status.Error(codes.InvalidArgument, err.Error())
}
```

валидация в вызывающий код

---

### Пользовательские типы

```protobuf
message Author {
  uint64 user_id = 1;
	string name = 2;
	string display_name = 3;
}
```

```protobuf
message Task {
	uint64 id = 1;
  string description = 2;
  Author author = 3;
}
```



Вложенные сообщения 🏔️

Все поля по умолчанию optional **?**

Наименование полей - **snake_case**

Идентификаторы только на расширение

Поля можно переименовывать ?

Важен ли порядок размещения полей ?

> The " = 1", " = 2" markers on each element identify the unique "tag" that field uses in the binary encoding. Tag numbers 1-15 require one less byte to encode than higher numbers, so as an optimization you can decide to use those tags for the commonly used or repeated elements, leaving tags 16 and higher for less-commonly used optional elements

---

### Встроенные типы

| Protobuf     | C++    | Go         |
| ------------ | ------ | ---------- |
| double       | double | float64    |
| float        | float  | float32    |
| int32 🔑      | int32  | int32      |
| int64 🔑      | int64  | int64      |
| uint32 🔑     | uint32 | uint32     |
| uint64 🔑     | uint64 | uint64     |
| sint32 🔑     | int32  | int32      |
| sint64 🔑     | int64  | int64      |
| bool 🔑 **?** | bool   | bool       |
| string 🔑     | string | string     |
| bytes        | string | **[]byte** |

отображения **map** и повторяемые **repeated**

> The `value_type` can be any type except <u>another map</u>

```protobuf
message MutliDescribeTaskRequest {
	repeated uint64 ids = 1;
}

message MutliDescribeTaskResponse {
	map<uint64, Task> tasks = 1;
}
```

Для совместимости массив пар: ключ - значение 

---

### Известные типы

| **wrappers.proto** | Field  | Type   | Description      |
| ------------------ | ------ | ------ | ---------------- |
| BoolValue          | value  | bool   | The bool value   |
| BytesValue         | value  | bytes  | The bytes value  |
| DoubleValue        | value  | double | The double value |
| FloatValue         | value  | float  | The float value  |
| Int32Value         | value  | int32  | The int32 value  |
| Int64Value         | values | int64  | The int64 value  |
| StringValue        | values | string | The string value |

| **timestamp.proto** | Field     | Type  | Description                                                  |
| ------------------- | --------- | ----- | ------------------------------------------------------------ |
| Timestamp           | `seconds` | int64 | Represents seconds of UTC time since Unix epoch 1970-01-01T00:00:00Z. Must be from 0001-01-01T00:00:00Z to 9999-12-31T23:59:59Z inclusive |
|                     | `nanos`   | int32 | Non-negative fractions of a second at nanosecond resolution. Negative second values with fractions must still have non-negative nanos values that count forward in time. Must be from 0 to 999,999,999 inclusive |

---

| **duration.proto** | Field     | Type  | Description                                                  |
| ------------------ | --------- | ----- | ------------------------------------------------------------ |
| Duration           | `seconds` | int64 | Signed seconds of the span of time. Must be from -315,576,000,000 to +315,576,000,000 inclusive |
|                    | `nanos`   | int32 | Signed fractions of a second at nanosecond resolution of the span of time. Durations less than one second are represented with a 0 `seconds` field and a positive or negative `nanos` field. For durations of one second or more, a non-zero value for the `nanos` field must be of the same sign as the `seconds` field. Must be from -999,999,999 to +999,999,999 inclusive |

В **empty.proto** определен Empty

```protobuf
import "google/protobuf/empty.proto";

service OcpTaskApi {
    // Удаляет задачу по ее идентификатору
    rpc RemoveTaskV1(RemoveTaskV1Request) returns (google.protobuf.Empty) {
        option (google.api.http) = {
            delete: "/tasks/{task_id}"
        };
    }
}
```

---























## ⏰

























---

### Repeated

```protobuf
message Task {
    uint64 task_id = 1;
    string description = 2;
    repeated string tags = 3;
    repeated Author authors = 4;
}
```



Было одно стало много

Генерация для встроенных типов

Генерерация для пользовательских типов

Проект **gogo**

Было много стало мало ?

### Перечисления

```go
enum TaskDifficulty {
    Begin = 0;
    Easy = 1;
    Normal = 2;
    Hard = 3;
}

// Описание задачи
message Task {
    uint64 task_id = 1;
    string description = 2;
    TaskDifficulty difficulty = 3; // +
}
```

На расширение

Минимальное значение для первого элемента **?**

Максимальное значение для первого элемента **?**

**Unknown** со значение 0 💡

Разрывы в нумерации 💡

Наименование **Type** + **Value**

Может быть ключом для отображения ? 🙋

Когда лучше использовать ? 🙋

---

### Swagger

```sh
➜ go install github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger

➜ protoc -I vendor.protogen \
  --go_out=pkg/ocp-task-api --go_opt=paths=import \
  --go-grpc_out=pkg/ocp-task-api --go-grpc_opt=paths=import \
  --grpc-gateway_out=pkg/ocp-task-api \
  --grpc-gateway_opt=logtostderr=true \
  --grpc-gateway_opt=paths=import \
  --validate_out lang=go:pkg/ocp-task-api \
  --swagger_out=allow_merge=true,merge_file_name=api:swagger \ # +
  api/ocp-task-api/ocp-task-api.proto
```

```sh
➜ docker run -p 80:8080 \
  -e BASE_URL=/swagger -e SWAGGER_JSON=/swagger/api.swagger.json \
	-v `pwd`/swagger:/swagger swaggerapi/swagger-ui
➜ open http://localhost:80
```

















---

### gRPC коды

```go
	resp, err := a.client.DescribeTaskV1(ctx, req)
	if err != nil {
		status, ok := status.FromError(err)
		if !ok {
			return nil, fmt.Errorf("get status from to error: %w", err)
		}
		if status.Code() == codes.NotFound {
			return nil, ErrTaskNotFound
		}
		return nil, fmt.Errorf("describe task: %w", err)
	}

```

- OK
- **Canceled**
- Unknown
- **InvalidArgument**
- **DeadlineExceeded**
- **NotFound**
- AlreadyExists
- PermissionDenied
- ResourceExhausted
- **FailedPrecondition**
- Aborted
- OutOfRange
- **Unimplemented**
- **Internal**
- Unavailable
- DataLoss
- Unauthenticated
