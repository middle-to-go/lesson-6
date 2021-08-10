---
customTheme : "presentation"

---

## Лекция 6. Protobuf и gRPC
### OZON

### Москва, 2021


---

### Лекции

1. <span style="color:gray">Введение. Рабочее окружение. Структура программы. Инструментарий.</span>
2. <span style="color:gray">Базовые конструкции и операторы. Встроенные типы и структуры данных.</span>
3. <span style="color:gray">Структуры данных, отложенные вызовы, обработка ошибок и основы тестирования</span>
4. <span style="color:gray">Интерфейсы, моки и тестирование с ними</span>
5. <span style="color:gray">Асинхронные сущности и паттерны в Go</span>
6. Protobuf и gRPC
7. <span style="color:gray">Работа с БД в Go</span>
8. <span style="color:gray">Брокеры сообщений. Трассировка. Метрики. </span>

---

### Темы

Сегодня мы поговорим про:

1. Мандат Безоса<!-- .element: class="fragment" -->
1. Зачем нужны Protobuf и gRPC<!-- .element: class="fragment" -->
1. Как построить создать микросервис на Go<!-- .element: class="fragment" -->
1. Как подключить REST API<!-- .element: class="fragment" -->
1. Как обратиться к сервису<!-- .element: class="fragment" -->
1. Makefile и Docker для окружения (но можем не успеть)<!-- .element: class="fragment" -->


---

### Обозначения

* 📽️ - посмотри воркшоу
* ⚗️ - проведи эксперимент
* 🔬 - изучи внимательно
* 📖 - прочитай документация
* 🪙 - подумай о сложности
* 🐞 - запомни ошибку
* 🔨 - запомни решение
* 🏔️ - обойди камень предкновенья
* ⏰ - сделай перерыв
* 🏡 - попробуй дома
* 💡 - обсуди светлые идеи
* 🙋 - задай вопрос
* ⚡ - запомни панику

---

### Мандат Безоса

Оригинал: http://jesusgilhernandez.com/2012/10/18/jeff-bezos-mandate-amazon-and-web-services/

1. Отныне все команды будут предоставлять свои данные и функциональные возможности через сервисные интерфейсы. <!-- .element: class="fragment" -->
2. Команды должны взаимодействовать друг с другом через эти интерфейсы. <!-- .element: class="fragment" -->
3. Не будет разрешена никакая другая форма межпроцессной связи: никаких прямых ссылок, никаких прямых считываний хранилища данных другой команды, никакой модели общей памяти, никаких "задних дверей" вообще. Единственная разрешенная связь-это вызовы интерфейса службы по сети. <!-- .element: class="fragment" -->


---

### Мандат Безоса

4. Не имеет значения, какую технологию они используют. HTTP, Corba, Pubsub, пользовательские протоколы — не имеет значения.
5. Все интерфейсы обслуживания, без исключения, должны быть спроектированы с нуля, чтобы быть внешними. То есть команда должна планировать и проектировать, чтобы иметь возможность предоставлять интерфейс разработчикам во внешнем мире. Никаких исключений. <!-- .element: class="fragment" -->
6. Любой, кто этого не сделает, будет уволен. <!-- .element: class="fragment" -->
7. Спасибо, хорошего вам дня! <!-- .element: class="fragment" -->


---

### Protobuf

Protocol Buffers — протокол сериализации (передачи) структурированных данных, предложенный Google как эффективная бинарная альтернатива текстовому формату XML. 

Разработчики сообщают, что Protocol Buffers проще, компактнее и быстрее, чем XML, поскольку осуществляется передача бинарных данных, оптимизированных под минимальный размер сообщения <!-- .element: class="fragment" -->

🙋 Какие ещё методы сериализации вы знаете? <!-- .element: class="fragment" -->

---

### gRPC

gRPC (Remote Procedure Calls) — это система удалённого вызова процедур (RPC) с открытым исходным кодом, первоначально разработанная в Google в 2015 году. В качестве транспорта используется HTTP/2, в качестве языка описания интерфейса — буферы протоколов

gRPC предоставляет такие функции как аутентификация, двунаправленная потоковая передача и управление потоком, блокирующие или неблокирующие привязки, а также отмена и тайм-ауты.  <!-- .element: class="fragment" -->


---

### gRPC


Генерирует кроссплатформенные привязки клиента и сервера для многих языков. Чаще всего используется для подключения служб в микросервисном стиле архитектуры и подключения мобильных устройств и браузерных клиентов к серверным службам.

Сложное использование HTTP/2 в gRPC делает невозможным реализацию клиента gRPC в браузере - вместо этого требуется прокси. <!-- .element: class="fragment" -->

🔬 Больше про разницу в версиях HTTP: https://cheapsslsecurity.com/p/http2-vs-http1/ <!-- .element: class="fragment" -->

---

### Proto файл

```go
syntax = "proto3";
​
option go_package = "github.com/ozoncp/ocp-task-api/pkg/ocp-task-api;ocp_task_api";
​
package ocp.task.api;
​
// Описание задачи
message Task {
    uint64 task_id = 1;
    string description = 2;
    
    string comment = 20;
}
```

syntax определяет версию proto-спецификации: <!-- .element: class="fragment" -->
* 3 <!-- .element: class="fragment" -->
* 2 (default) <!-- .element: class="fragment" -->


package - наименование пакета, пространство имен <!-- .element: class="fragment" -->

// comment - используются в документации <!-- .element: class="fragment" -->

---

### Proto файл

; - часто встречаемая ошибка 🐞 - забыл поставить ;

`option` - для разных случаев, можно и свои (полный список [тут](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/descriptor.proto))

```go
string description = 2 [deprecated = true]; 
option optimize_for = SPEED, CODE_SIZE, LITE_RUNTIME
```

Расположение:

`./api` -- директория для proto файлов, описывающих API сервисов
`./vendor.protogen` -- директория для вендоринга proto файлов, включая proto файлы из директории api.

---

### Service

```go
syntax = "proto3";

import "google/api/annotations.proto";
import "google/protobuf/empty.proto";

package siriusfreak.lecture_6_demo;

option go_package = "gitlab.com/siriusfreak/lecture-6-demo/pkg/lecture-6-demo;lecture_6_demo";

// Lecture6Demo сервис для обработки текстов
service Lecture6Demo {
  // AddV1 добавляет задание в очередь на обработку
  rpc AddV1(AddRequestV1) returns (google.protobuf.Empty) {
  }
}

message AddRequestV1 {
  int64 id = 1;
  string text = 2;
  bool result = 3;
  string callback_url = 4; 
}
```

---

### Service

Что надо обсудить заранее:

* Наименования сервисов <!-- .element: class="fragment" -->

* Наименование RPC ручек - verb + subject <!-- .element: class="fragment" -->

* Множественное определение сервисов 🏔️ <!-- .element: class="fragment" -->
 
* Потоковое передача данных: когда и зачем ? <!-- .element: class="fragment" -->

* Общие типы запросов и ответов 🏔️ <!-- .element: class="fragment" -->

* Порядок расположения сообщений и сервисов 💡 <!-- .element: class="fragment" -->

* Свой пустой тип или `google.protobuf.Empty`? <!-- .element: class="fragment" -->

---

### Service

И помните, что часы планирования лишают нас недель разработки не того, чего мы хотели / не того, чего надо.

---

### Генерация

Установка компилятора protoc:

```
➜ brew install protobuf # или apt-get или просто отсюда: https://github.com/protocolbuffers/protobuf/releases
➜ which protoc
➜ protoc --version
```

Установка зависимостей (часть Makefile):
```
    LOCAL_BIN:=$(CURDIR)/bin

    .PHONY: deps
    deps: install-go-deps

    .PHONY: .install-go-deps
    .install-go-deps:
		ls go.mod || go mod init gitlab.com/siriusfreak/lecture-6-demo
		GOBIN=$(LOCAL_BIN) go get -u github.com/grpc-ecosystem/grpc-gateway/protoc-gen-grpc-gateway
		GOBIN=$(LOCAL_BIN) go get -u github.com/golang/protobuf/proto
		GOBIN=$(LOCAL_BIN) go get -u github.com/golang/protobuf/protoc-gen-go
		GOBIN=$(LOCAL_BIN) go get -u google.golang.org/grpc
		GOBIN=$(LOCAL_BIN) go get -u google.golang.org/grpc/cmd/protoc-gen-go-grpc
		GOBIN=$(LOCAL_BIN) go install google.golang.org/grpc/cmd/protoc-gen-go-grpc
		GOBIN=$(LOCAL_BIN) go install github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger
```

```
    make deps
```
    

---

### Makefile

В общем случае: файл, который описывает как собрать проект на сях, со своими плюшками. <!-- .element: class="fragment" -->

В нашем случае: удобная форма записи консольных команд. <!-- .element: class="fragment" -->

---

### Генерация

Генерация кода из proto-файлов

```
➜ 	GOBIN=$(LOCAL_BIN) protoc -I vendor.protogen \
        --go_out=pkg/lecture-6-demo --go_opt=paths=import \
        --go-grpc_out=pkg/lecture-6-demo --go-grpc_opt=paths=import \
        api/lecture-6-demo/lecture-6-demo.proto
        
➜ ls -l pkg/lecture-6-demo/gitlab.com/siriusfreak/lecture-6-demo/pkg/lecture-6-demo
```
​
```
lecture-6-demo.pb.go    # types
lecture-6-demo_grpc.pb.go # grpc
```

---

### Имплементация обработчика

internal/app/lecture-6-demo/service.go

```go
package lecture_6_demo

import desc "gitlab.com/siriusfreak/lecture-6-demo/pkg/lecture-6-demo"

type Lecture6DemoAPI struct {
	desc.UnimplementedLecture6DemoServer
}


func NewLecture6DemoAPI() desc.Lecture6DemoServer {
	return &Lecture6DemoAPI{
	}
}
```

---

### Имплементация обработчика

internal/app/lecture-6-demo/add_v1.go

```go
package lecture_6_demo

import (
	"context"

	desc "gitlab.com/siriusfreak/lecture-6-demo/pkg/lecture-6-demo"
	"google.golang.org/protobuf/types/known/emptypb"
)

func (a *Lecture6DemoAPI)AddV1(ctx context.Context, req *desc.AddRequestV1) (*emptypb.Empty, error) {
	return &emptypb.Empty{}, nil
}
```

---

### Запуск сервиса

```go
const (
	grpcPort = ":82"
	grpcServerEndpoint = "localhost:82"
)


func run() error {
	listen, err := net.Listen("tcp", grpcPort)
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	s := grpc.NewServer()
	desc.RegisterLecture6DemoServer(s, api.NewLecture6DemoAPI())

	if err := s.Serve(listen); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}

	return nil
}
```

---

### Запуск сервиса

```go

package main

import (
	"log"
	"net"

	api "gitlab.com/siriusfreak/lecture-6-demo/internal/app/lecture-6-demo"
	"google.golang.org/grpc"

	desc "gitlab.com/siriusfreak/lecture-6-demo/pkg/lecture-6-demo"
)

func main() {
	if err := run(); err != nil {
		log.Fatal(err)
	}
}

```

---

### Проксирование 

Добавляем определения для шлюза.

```go
syntax = "proto3";

import "google/api/annotations.proto";
import "google/protobuf/empty.proto";

package siriusfreak.lecture_6_demo;

option go_package = "gitlab.com/siriusfreak/lecture-6-demo/pkg/lecture-6-demo;lecture_6_demo";

// Lecture6Demo сервис для обработки текстов
service Lecture6Demo {
  // AddV1 добавляет задание в очередь на обработку
  rpc AddV1(AddRequestV1) returns (google.protobuf.Empty) {
    option (google.api.http) = {
      post: "/v1/add"
    };
  }
}

message AddRequestV1 {
  int64 id = 1;
  string text = 2;
  bool result = 3;
  string callback_url = 4; 
}
```

---

### Проксирование 

Добавляем генерацию шлюза и swagger.

```go
protoc -I vendor.protogen \
        --go_out=pkg/lecture-6-demo --go_opt=paths=import \
        --go-grpc_out=pkg/lecture-6-demo --go-grpc_opt=paths=import \
        --grpc-gateway_out=pkg/lecture-6-demo \
        --grpc-gateway_opt=logtostderr=true \
        --grpc-gateway_opt=paths=import \
        --swagger_out=allow_merge=true,merge_file_name=api:swagger \
        api/lecture-6-demo/lecture-6-demo.proto
```

---

### Проксирование

Запускаем обработчик REST API:

```go

func runJSON() {
	ctx := context.Background()
	ctx, cancel := context.WithCancel(ctx)
	defer cancel()

	mux := runtime.NewServeMux()
	opts := []grpc.DialOption{grpc.WithInsecure()}

	err := desc.RegisterLecture6DemoHandlerFromEndpoint(ctx, mux, grpcServerEndpoint, opts)
	if err != nil {
		panic(err)
	}

	err = http.ListenAndServe(":8081", mux)
	if err != nil {
		panic(err)
	}
}

func main() {
	go runJSON()

	if err := run(); err != nil {
		log.Fatal(err)
	}
}

```

---

### Как дёргать ручки?

gRPC: https://github.com/uw-labs/bloomrpc

Gateway: https://www.postman.com/

---

### Версионирование

| Before | After |
|-|-|
|DescribeTask|	DescribeTaskV1|
|DescribeTaskRequest	|DescribeTaskV1Request|
|DescribeTaskResponse|	DescribeTaskV1Response|
|"/tasks/{task_id}"|	"/v1/tasks/{task_id}"|

> #обычные-сообщения #cовместное-обновление

---

### Версионирование

Добавление новой версии ручки

* Задеплоить релиз с новой версией ручки <!-- .element: class="fragment" -->
* Перевести всех нужных клиентов на новую версию <!-- .element: class="fragment" -->
 

Переход на новую версию ручки <!-- .element: class="fragment" -->

* Пометить, как deprecated <!-- .element: class="fragment" -->
* Задеплоить релиз с новой версией ручки <!-- .element: class="fragment" -->
* Перевести всех клиентов на новую версию <!-- .element: class="fragment" -->
* Задеплоить релиз без старой версии ручки, если не планируется поддерживать <!-- .element: class="fragment" -->

---

### Деплой пакета

```go

➜ cd pkg/lecture-6-demo/
➜ go mod init && go mod tidy
➜ git tag pkg/lecture-6-demo/v0.1.0 # v1.0.0
➜ git push origin --tags
​
➜ z lecture-6-demo
➜ go get gitlab.com/siriusfreak/lecture-6-demo/pkg/lecture-6-demo# v1.0.0
```

---

### Деплой пакета

```go
import ( 
  taskApi "github.com/ozoncp/ocp-task-api/pkg/ocp-task-api"
  
  "google.golang.org/grpc"
)
​
conn, err := grpc.Dial(taskApiAddress, grpc.WithInsecure())
​
client := taskApi.NewOcpTaskApiClient(conn)
​
req := &taskApi.DescribeTaskV1Request{
  TaskId: taskId,
}
​
resp, err := client.DescribeTaskV1(ctx, req)
```


---

### Деплой пакета

1. Использование replace для отладки

2. Использование go get для обновления <!-- .element: class="fragment" -->
 
3. Версионирование semver <!-- .element: class="fragment" -->

4. Генерация пакетов на других языках <!-- .element: class="fragment" -->

---

### Валидация

```go

// ...
import "github.com/envoyproxy/protoc-gen-validate/validate/validate.proto";
// ...
​
message AddRequestV1 {
  int64 id = 1 [(validate.rules).uint64.gt = 0];
  string text = 2;
  bool result = 3;
  string callback_url = 4;
}
```


---

### Валидация

```go
protoc -I vendor.protogen \
        --go_out=pkg/lecture-6-demo --go_opt=paths=import \
        --go-grpc_out=pkg/lecture-6-demo --go-grpc_opt=paths=import \
        --grpc-gateway_out=pkg/lecture-6-demo \
        --grpc-gateway_opt=logtostderr=true \
        --grpc-gateway_opt=paths=import \
        --swagger_out=allow_merge=true,merge_file_name=api:swagger \
        --validate_out lang=go:pkg/lecture-6-demo \ # +
        api/lecture-6-demo/lecture-6-demo.proto
```


---

### Валидация

```
➜ ls -l pkg/lecture-6-demo
lecture-6-demo.pb.go
lecture-6-demo.pb.gw.go
lecture-6-demo.pb.validate.go # new file
lecture-6-demo_grpc.pb.go 
```

---

### Валидация


* Инсталирование плагина
* Генерация кода
* Добавление проверки в обработчики
* Добавление проверки в вызывающий код

```go
import (
  "google.golang.org/grpc/codes"
  "google.golang.org/grpc/status"
)
​
if err := req.Validate(); err != nil {
    return nil, status.Error(codes.InvalidArgument, err.Error())
}
```

валидация в вызывающий код

---

### Пользовательские типы

```go
message Author {
  uint64 user_id = 1;
  string name = 2;
  string display_name = 3;
}
```

```go
message Task {
  uint64 id = 1;
  string description = 2;
  Author author = 3;
}
```

---

### Пользовательские типы

Вопросы:
1. Вложенные сообщения 🏔️ <!-- .element: class="fragment" -->
1. Все поля по умолчанию optional ? <!-- .element: class="fragment" -->
1. Наименование полей - snake_case <!-- .element: class="fragment" -->
1. Идентификаторы только на расширение <!-- .element: class="fragment" -->
1. Поля можно переименовывать ? <!-- .element: class="fragment" -->
1. Важен ли порядок размещения полей ? <!-- .element: class="fragment" -->

---

### Встроенные типы

|Protobuf|	C++	|Go|
|-|-|-|
|double	|double|	float64|
|float|	float	|float32|
|int32 🔑	|int32|	int32|
|int64 🔑|	int64	|int64|
|uint32 🔑|	uint32|	uint32|
|uint64 🔑| uint64|	uint64|
|sint32 🔑|	int32| int32|
|sint64 🔑	|int64	|int64|
|bool 🔑 ?|	bool|	bool|
|string 🔑|	string|	string|
|bytes|	string|	[]byte|

---

### Встроенные типы

отображения map и повторяемые repeated

> The value_type can be any type except another map

```go
message MutliDescribeTaskRequest {
  repeated uint64 ids = 1;
}
​
message MutliDescribeTaskResponse {
  map<uint64, Task> tasks = 1;
}
```

Для совместимости массив пар: ключ - значение  <!-- .element: class="fragment" -->


---

### Известные типы

|wrappers.proto	|Field	|Type|	Description|
|-|-|-|-|
|BoolValue|	value|	bool|	The bool value|
|BytesValue|	value	|bytes	|The bytes value|
|DoubleValue	|value|	double|The double value|
|FloatValue|	value	|float|	The float value|
|Int32Value|	value	|int32	|The int32| value|
|Int64Value|	values|	int64|	The int64 value|
|StringValue|	values|	string|	The string value|

---

### Известные типы


|timestamp.proto|	Field|	Type	|Description|
|-|-|-|-|
|Timestamp|	seconds	|int64|	Represents seconds of UTC time since Unix epoch 1970-01-01T00:00:00Z. Must be from 0001-01-01T00:00:00Z to 9999-12-31T23:59:59Z inclusive|
 |	|nanos|	int32	|Non-negative fractions of a second at nanosecond resolution. Negative second values with fractions must still have non-negative nanos values that count forward in time. Must be from 0 to 999,999,999 inclusive|


---

### Известные типы

|duration.proto	|Field|Type	|Description|
|-|-|-|-|
|Duration	|seconds|	int64	|Signed seconds of the span of time. Must be from -315,576,000,000 to +315,576,000,000 inclusive|
| 	|nanos|	int32	|Signed fractions of a second at nanosecond resolution of the span of time. Durations less than one second are represented with a 0 seconds field and a positive or negative nanos field. For durations of one second or more, a non-zero value for the nanos field must be of the same sign as the seconds field. Must be from -999,999,999 to +999,999,999 inclusive|


---

### Известные типы

В `empty.proto` определен `Empty`.

```go
import "google/protobuf/empty.proto";
​
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

### Repeated

```go
message Task {
    uint64 task_id = 1;
    string description = 2;
    repeated string tags = 3;
    repeated Author authors = 4;
}
```

---

### Перечисления

```go
enum TaskDifficulty {
    Begin = 0;
    Easy = 1;
    Normal = 2;
    Hard = 3;
}
​
// Описание задачи
message Task {
    uint64 task_id = 1;
    string description = 2;
    TaskDifficulty difficulty = 3; // +
}
```

---

### Вопросы

* Минимальное значение для первого элемента ?
* Максимальное значение для первого элемента ? <!-- .element: class="fragment" -->
* Unknown со значение 0 💡 <!-- .element: class="fragment" -->
* Разрывы в нумерации 💡 <!-- .element: class="fragment" -->
* Наименование Type + Value <!-- .element: class="fragment" -->
* Что может быть ключом для отображения ? 🙋 <!-- .element: class="fragment" -->
* Когда что лучше использовать ? 🙋 <!-- .element: class="fragment" -->

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

---

### gRPC коды

* OK
* Canceled
* Unknown
* InvalidArgument
* DeadlineExceeded
* NotFound
* AlreadyExists
* PermissionDenied
* ResourceExhausted
* FailedPrecondition
* Aborted
* OutOfRange
* Unimplemented
* Internal
* Unavailable
* DataLoss
* Unauthenticated


---

### Makefile

Фальшивая (PHONY) цель-это цель, которая на самом деле не является именем файла; скорее, это просто имя рецепта, который будет выполняться при выполнении явного запроса. 

Есть две причины использовать фальшивую цель: избежать конфликта с файлом с тем же именем и повысить производительность. <!-- .element: class="fragment" -->
 

---

### Makefile

Сборка, тесты, линтер:

```makefile

LOCAL_BIN:=$(CURDIR)/bin

.PHONY: run
run:
	GOBIN=$(LOCAL_BIN) go run cmd/lecture-6-demo/main.go

.PHONY: lint
lint:
	GOBIN=$(LOCAL_BIN) golint ./...

.PHONY: test
test:
	GOBIN=$(LOCAL_BIN) go test -v ./...

.PHONY: .build
.build:
	GOBIN=$(LOCAL_BIN) go build -o $(LOCAL_BIN)/lecture-6-demo cmd/lecture-6-demo/main.go
```


---

### Makefile

Генерация кода:

```makefile
.PHONY: build
build: vendor-proto .generate .build

.PHONY: .generate
.generate:
		mkdir -p swagger
		mkdir -p pkg/lecture-6-demo
		GOBIN=$(LOCAL_BIN) protoc -I vendor.protogen \
				--go_out=pkg/lecture-6-demo --go_opt=paths=import \
				--go-grpc_out=pkg/lecture-6-demo --go-grpc_opt=paths=import \
				--grpc-gateway_out=pkg/lecture-6-demo \
				--grpc-gateway_opt=logtostderr=true \
				--grpc-gateway_opt=paths=import \
				--swagger_out=allow_merge=true,merge_file_name=api:swagger \
				api/lecture-6-demo/lecture-6-demo.proto
		mv pkg/lecture-6-demo/gitlab.com/siriusfreak/lecture-6-demo/pkg/lecture-6-demo/* pkg/lecture-6-demo/
		rm -rf pkg/lecture-6-demo/gitlab.com
		mkdir -p cmd/lecture-6-demo
		cd pkg/lecture-6-demo && ls go.mod || go mod init gitlab.com/siriusfreak/lecture-6-demo/pkg/lecture-6-demo && go mod tidy

.PHONY: generate
generate: .vendor-proto .generate
```

---

### Makefile

Скачивание протофайлов:

```makefile
.PHONY: vendor-proto
vendor-proto: .vendor-proto

.PHONY: .vendor-proto
.vendor-proto:
		mkdir -p vendor.protogen
		mkdir -p vendor.protogen/api/lecture-6-demo
		cp api/lecture-6-demo/lecture-6-demo.proto vendor.protogen/api/lecture-6-demo/lecture-6-demo.proto
		@if [ ! -d vendor.protogen/google ]; then \
			git clone https://github.com/googleapis/googleapis vendor.protogen/googleapis &&\
			mkdir -p  vendor.protogen/google/ &&\
			mv vendor.protogen/googleapis/google/api vendor.protogen/google &&\
			rm -rf vendor.protogen/googleapis ;\
		fi
```

🏡 Добавить скачивание для: `github.com/envoyproxy/protoc-gen-validate/validate/validate.proto`


---

### Makefile

Установка зависимостей: 

```makefile

.PHONY: deps
deps: install-go-deps

.PHONY: install-go-deps
install-go-deps: .install-go-deps

.PHONY: .install-go-deps
.install-go-deps:
		ls go.mod || go mod init gitlab.com/siriusfreak/lecture-6-demo
		GOBIN=$(LOCAL_BIN) go get -u github.com/grpc-ecosystem/grpc-gateway/protoc-gen-grpc-gateway
		GOBIN=$(LOCAL_BIN) go get -u github.com/golang/protobuf/proto
		GOBIN=$(LOCAL_BIN) go get -u github.com/golang/protobuf/protoc-gen-go
		GOBIN=$(LOCAL_BIN) go get -u google.golang.org/grpc
		GOBIN=$(LOCAL_BIN) go get -u google.golang.org/grpc/cmd/protoc-gen-go-grpc
		GOBIN=$(LOCAL_BIN) go install google.golang.org/grpc/cmd/protoc-gen-go-grpc
		GOBIN=$(LOCAL_BIN) go install github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger

```

---

### Docker

Чтобы ваш код работал не только дома.

Установка: 
* Mac и Windows: https://www.docker.com/products/docker-desktop
* Linux: https://docs.docker.com/engine/install/ubuntu/

Docker-compose для БД `docker-compose.yml`:

```yaml
# Use postgres/example user/password credentials
version: '3.1'

services:

  db:
    image: postgres
    restart: always
    environment:
      POSTGRES_PASSWORD: example

  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
```

```docker compose up```

