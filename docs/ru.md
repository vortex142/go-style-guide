# Vortex Go Style Guide

Стандарт от **03.10.2026**.

---

Данный документ фиксирует единый стандарт разработки и оформления кода на языке **Go** в рамках экосистемы **Vortex**.
Соблюдение изложенных правил является обязательным для всех сервисов и компонентов организации.

Документ базируется на рекомендациях [Effective Go](https://golang.org/doc/effective_go) и [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments).

**Иерархия приоритетов:**
1. Правила и архитектурные требования, явно описанные в настоящем Стандарте (**Vortex Go Style Guide**), имеют абсолютный приоритет.
2. В вопросах, не затронутых данным документом, следует строго руководствоваться положениями *Effective Go* и *Go Code Review Comments*.

## Содержание

- [Vortex Go Style Guide](#vortex-go-style-guide)
  - [Содержание](#содержание)
  - [1. Общие правила и сигнатуры функций](#1-общие-правила-и-сигнатуры-функций)
    - [1.1. Запрет использования функции init()](#11-запрет-использования-функции-init)
    - [1.2. Передача контекста (`context.Context`)](#12-передача-контекста-contextcontext)
    - [1.3. Позиция ошибки в возвращаемых значениях](#13-позиция-ошибки-в-возвращаемых-значениях)
  - [2. Структура файла](#2-структура-файла)
    - [2.1. Порядок объявления элементов в файле](#21-порядок-объявления-элементов-в-файле)
    - [2.2. Конструкторы и структуры параметров (Params)](#22-конструкторы-и-структуры-параметров-params)
  - [3. Именование (Naming Conventions)](#3-именование-naming-conventions)
    - [3.1. Ошибки](#31-ошибки)
    - [3.2. CRUD-функции](#32-crud-функции)
  - [4. Комментарии](#4-комментарии)
    - [4.1. Общие требования к комментариям](#41-общие-требования-к-комментариям)
    - [4.2. Гиперссылки](#42-гиперссылки)
    - [4.3. Структура комментариев к функциям и методам](#43-структура-комментариев-к-функциям-и-методам)
    - [4.4. Исключения и послабления](#44-исключения-и-послабления)
      - [Отсутствие аргументов/результатов](#отсутствие-аргументоврезультатов)
      - [Короткие функции](#короткие-функции)
      - [Очевидные аргументы](#очевидные-аргументы)
      - [Логические блоки и перенос длинных строк](#логические-блоки-и-перенос-длинных-строк)
    - [4.5. Комментирование возвращаемых ошибок](#45-комментирование-возвращаемых-ошибок)
    - [4.6. Комментирование переменных ошибок](#46-комментирование-переменных-ошибок)
  - [5. Тестирование](#5-тестирование)
    - [5.1. Ограничение видимости](#51-ограничение-видимости)
    - [5.2. Порядок полей в Table-Driven тестах](#52-порядок-полей-в-table-driven-тестах)
  - [6. Работа с ошибками (error)](#6-работа-с-ошибками-error)
    - [6.1. Обертывание и логирование ошибок](#61-обертывание-и-логирование-ошибок)
    - [6.2. Агрегация ошибок через errors.Join()](#62-агрегация-ошибок-через-errorsjoin)
    - [6.3. Использование errors.New() для статичных ошибок](#63-использование-errorsnew-для-статичных-ошибок)
    - [6.4. Сбор ошибок в `Validate()`](#64-сбор-ошибок-в-validate)
    - [6.5. Текст ошибок](#65-текст-ошибок)
      - [Ошибки отсутствия обязательного ресурса](#ошибки-отсутствия-обязательного-ресурса)
      - [Ошибки ненайденного ресурса](#ошибки-ненайденного-ресурса)


---

## 1. Общие правила и сигнатуры функций

### 1.1. Запрет использования функции init()

Использование функции `init()` в коде строго запрещено. Инициализация переменных, зависимостей и конфигураций должна происходить явно через конструкторы или вызовы инициализаторов в пакетах `main` / `app`.

> Причина: `init()` скрывает порядок инициализации, усложняет тестирование и создает неявные побочные эффекты при импорте пакета

### 1.2. Передача контекста (`context.Context`)

Если функция или метод принимает `context.Context`, он **всегда** должен передаваться первым аргументом.

```go
// Good
func (s *Service) FetchUser(ctx context.Context, userID string) (*User, error)

// Bad
func (s *Service) FetchUser(userID string, ctx context.Context) (*User, error)
```

### 1.3. Позиция ошибки в возвращаемых значениях

Если функция возвращает значение ошибки (`error`), оно всегда должно находиться на последней позиции в списке возвращаемых значений.

```go
// Good
func Load(path string) (*Config, error)

// Bad
func Load(path string) (error, *Config)
```

## 2. Структура файла

### 2.1. Порядок объявления элементов в файле

Каждый `.go` файл должен соблюдать строгую последовательность объявления элементов кода:

1. Глобальные константы (`const`) и переменные (`var`) файла.
2. Кастомные типы и интерфейсы (`type`).
3. Публичные методы типов (`func (t *Type) PublicMethod()`).
4. Приватные методы типов (`func (t *Type) privateMethod()`).
5. Публичные функции файла (`func PublicFunction()`).
6. Приватные функции файла (`func privateFunction()`).

```go
package example

// 1. Глобальные const и var
const defaultTimeout = 5

var packageVersion = "1.0"

// 2. Типы
type Service struct {
    timeout time.Duration
}

// 3. Публичные методы типов
func (s *Service) Timeout() time.Duration { return s.timeout }

// 4. Приватные методы типов
func (s *Service) setTimeout(t time.Duration) {
    s.timeout = t
}

// 5. Публичные функции файла
func PrintPackageVersion() { 
    fmt.Printf("version: %s", packageVersion) 
}

// 6. Приватные функции файла
func formatVersion() string {
    return "v" + packageVersion
}
```

### 2.2. Конструкторы и структуры параметров (Params)

Типы, используемые в качестве контейнера параметров для сборки структуры через конструктор *(паттерн Params)*, объявляются в конце блока типов.

Сразу после структуры `Params` и её методов обязан следовать сам конструктор типа:

```go
type Service struct {
    logger Logger
    db     Database
}

// Params объявляется в конце блока кастомных типов
type ServiceParams struct {
    Logger Logger
    DB     Database
}

func (p *ServiceParams) Validate() error {
    if p.Logger == nil {
        return errors.New("logger is required")
    }
    return nil
}

// Конструктор располагается строго после ServiceParams
func NewService(params *ServiceParams) (*Service, error) {
    if err := params.Validate(); err != nil {
        return nil, fmt.Errorf("invalid service params: %w", err)
    }

    return &Service{
        logger: params.Logger,
        db:     params.DB,
    }, nil
}
```

## 3. Именование (Naming Conventions)

### 3.1. Ошибки

Все переменные ошибок (как экспортируемые, так и приватные) обязаны начинаться с префикса `Err` (или `err`).

```go
// Good
var (
    ErrEmptyURL    = errors.New("URL is required")
    ErrUserNotFound = errors.New("user is required")
)

// Bad
var (
    BadURLErr = errors.New("URL cannot be empty")
)
```

### 3.2. CRUD-функции

Для функций и методов, взаимодействующих с базами данных и хранилищами, устанавливается явный префиксный стандарт.

> В отличие от общих рекомендаций *Effective Go* (где префикс **Get** не рекомендуется для обычных методов геттеров), в контексте **CRUD-операций** использование префикса **Get** разрешено и рекомендуется для повышения читаемости слоя данных

**Обязательные префиксы для CRUD:**

- `Get` — выгрузка или чтение существующих записей из БД (**GetByID**, **GetByName**).
- `Insert` — вставка и создание новых записей в БД (**InsertUser**, **InsertSession**).
- `Update` — модификация и обновление имеющихся данных (**UpdateStatus**, **UpdateProfile**).
- `Delete` — физическое или логическое удаление записей (**DeleteByID**, **DeleteSoft**).

```go
// Good: Четкая префиксация CRUD-операций в слое данных
func (r *Repository) GetUserByID(ctx context.Context, id string) (*User, error)
func (r *Repository) InsertUser(ctx context.Context, user *User) error
func (r *Repository) UpdateStatus(ctx context.Context, id string, status Status) error
func (r *Repository) DeleteByID(ctx context.Context, id string) error

// Bad: Использование нестандартных глаголов для работы с БД
func (r *Repository) SaveUser(ctx context.Context, user *User) error   // Используйте Insert
func (r *Repository) RemoveUser(ctx context.Context, id string) error  // Используйте Delete
func (r *Repository) FetchUser(ctx context.Context, id string) error   // Используйте Get
```

## 4. Комментарии

Документирование кода подчиняется строгим стандартам **GoDoc**. Главная цель комментариев — объяснить **ПОЧЕМУ** было принято то или иное **архитектурное/бизнес-решение**, а не пересказывать **КАК** работает код.

### 4.1. Общие требования к комментариям
1. **Язык:** Все комментарии пишутся строго на английском языке.
2. **Покрытие (100% Coverage):** Комментарии **обязаны** присутствовать у всех сущностей файла *(экспортируемых и приватных)* — **функций**, **методов**, **структур**, **интерфейсов**, **методов интерфейсов**, **кастомных типов**, **констант** и **переменных**.
3. **Форматирование:** Документирующий комментарий к экспортируемой *(и желательно приватной)* сущности обязан начинаться с её имени.
4. **Пунктуация**: Обязательно нужно ставить точку в конце предложения, будь то описание сущности или комментарий внутри функции.

### 4.2. Гиперссылки

Для создания гиперссылок оборачивайте типы в квадратные скобки: **[Type]** или **[pkg.Type]**.
**НЕ НУЖНО** оборачивать в гиперссылку сущность, которую вы описываете данным комментарием:

```go
// Good comment:

// ServiceParams encapsulates the necessary dependencies for creating a [Service] instance.
type ServiceParams struct{}

// Bad comment:

// [ServiceParams] encapsulates the necessary dependencies for creating a [Service] instance.
type ServiceParams struct{}
```

### 4.3. Структура комментариев к функциям и методам

Документирование функций выполняется по строгой построчной структуре:

- **Строка 1:** Краткое описание назначения **функции/метода** *(начинается с имени функции/метода)*.
- **Строка 2:** Описание аргументов и их необходимости (записывается как одно связное предложение, аргументы разделяются запятыми, без двоеточий :).
- **Строка 3:** Описание результата функции, начинающееся с фразы *"It returns..."* или аналогичной по смыслу.
- **Дополнительные примечания (Notes):** При необходимости добавить примечания, после **Строки 3** ставится пустая строка **//** и каждое примечание пишется с новой строки.

```go
// ConvertMedia transcodes the incoming media stream into the target format.
// Takes ctx for boundary cancellation, payload containing raw video bytes, and opts for configuration.
// It returns the converted [MediaResult] or an error if processing fails.
//
// This operation is resource-intensive and triggers CPU-bound workers in Vortex Engine.
func (s *Service) ConvertMedia(ctx context.Context, payload []byte, opts *ConvertOptions) (*MediaResult, error)
```

### 4.4. Исключения и послабления

#### Отсутствие аргументов/результатов

Если функция не принимает аргументов или ничего не возвращает, соответствующие строки в структуре комментария опускаются.

#### Короткие функции

Для простых функций и методов, чья реализация занимает до 5 строк кода, разрешен однострочный GoDoc-комментарий. Детализация аргументов и возвращаемых значений в нем может быть объединена, либо же отсутствовать полностью.

```go
// Good:

// threadsArgs returns FFmpeg CLI flags to configure multi-threaded processing.
func (c *ffmpegConverter) threadsArgs() []string {
	return []string{"-threads", strconv.Itoa(int(c.maxThreads)), "-filter_threads", strconv.Itoa(int(c.maxThreads))}
}

// Bad:

// threadsArgs Creates a slice of FFmpeg CLI arguments to control the number of threads used.
// The method takes no arguments.
// Returns a slice containing the FFmpeg arguments.
func (c *ffmpegConverter) threadsArgs() []string {
	return []string{"-threads", strconv.Itoa(int(c.maxThreads)), "-filter_threads", strconv.Itoa(int(c.maxThreads))}
}
```

#### Очевидные аргументы

Написание отдельной строки с описанием аргументов *(Строка 2)* не требуется, если назначение и необходимость параметров однозначно очевидны из их названий.

```go
// Good:

// Load reads and parses the JSON configuration at path.
// It returns the pointer to the parsed [Config] instance and an error if path is empty or reading or parsing fails.
func Load(path string) (*Config, error) {...}

// Bad:

// Load reads and parses the JSON configuration at path.
// Takes path as the path to the configuration file.
// It returns the pointer to the parsed [Config] instance and an error if path is empty or reading or parsing fails.
func Load(path string) (*Config, error) {...}
```

#### Логические блоки и перенос длинных строк

Обозначения *(Строка 1, Строка 2, Строка 3)* задают логические блоки **GoDoc-комментария**, а не его строгое физическое форматирование в одну линию кода.

Если описание аргументов *(Блок 2)* или возвращаемых значений *(Блок 3)* превышает комфортный лимит длины строки *(120–150 символов)*, описание разрешается и рекомендуется переносить на следующую строку с сохранением контекста.


### 4.5. Комментирование возвращаемых ошибок

Если функция или метод возвращает конкретные ошибки, которые не создаются внутри самой функции, они обязаны быть зафиксированы в GoDoc с использованием **гиперссылок** в **3-й строке** комментария.

> Это необходимо для того, чтобы вызывающий код точно знал, какие типы и переменные ошибок можно обрабатывать через `errors.Is()` или `errors.As()`

```go
var ErrMediaNotFound = errors.New("media not found")

var ErrAccessDenied = errors.New("access denied")

// FetchMedia retrieves the processed media content by its unique identifier.
// Takes ctx for timeout management and id representing the target media resource.
// It returns an [ErrMediaNotFound] if the record is missing, [ErrAccessDenied] if permissions are insufficient, any errors on validation failure, or the target [Media] if there are no errors.
func (s *Service) FetchMedia(ctx context.Context, id string) (*Media, error) {}
```

### 4.6. Комментирование переменных ошибок

Комментарии к переменным ошибок *(как публичным, так и приватным)* обязаны строго следовать шаблону: **"... is triggered when ..."** с четким указанием условий их возникновения.

```go
// ErrEmptyURL is triggered when there is an empty URL string in the Source payload structure.
var ErrEmptyURL = errors.New("URL is required")

// errTimeoutExceeded is triggered when the Vortex Engine fails to respond within the allocated deadline.
var errTimeoutExceeded = errors.New("timeout exceeded")
```

## 5. Тестирование

### 5.1. Ограничение видимости

В файлах тестов `(*_test.go)` разрешено объявлять **ТОЛЬКО** не экспортируемые типы, методы и функции (начинающиеся со строчной буквы).

```go
// Good
type mockUserRepo struct{}
func (m *mockUserRepo) getUser() {}
func createTestUser() *User {}

// Bad: Экспортируемые типы запрещены в тестовых файлах
type MockUserRepo struct{}
```

### 5.2. Порядок полей в Table-Driven тестах

При использовании табличных тестов структура кейса должна соблюдать строгий порядок полей:

1. `name string` — всегда идет первым полем в структуре кейса.
2. `wantErr bool` — всегда идет вторым полем **(если тестируемый метод возвращает error)**.
3. `Context (ctx)` — объявляется строго в конце структуры тестового кейса **(если тестируемый метод требует контекст)**.

## 6. Работа с ошибками (error)

### 6.1. Обертывание и логирование ошибок

- При передаче ошибки вверх по стеку вызовов всегда обертывайте её через `fmt.Errorf()` с использованием спецификатора `%w`.
- Спецификатор `%v` используется только для форматирования текста ошибки при логировании или выводе в консоль.

```go
err := service.CreateUser(id)

// Передача ошибки вверх по стеку
if err != nil {
    return fmt.Errorf("failed to create user: %w", err)
}

// Логирование ошибки
log.Printf("failed to create user: %v", err)
```

### 6.2. Агрегация ошибок через errors.Join()

Если функция накапливает ошибки в срез `[]error` для их последующего объединения, возвращайте результат напрямую через `errors.Join(errs...)`. **Проверка if len(errs) > 0 не требуется.**

```go
func (s *Service) Validate() error {
    var errs []error

    if s.logger == nil {
        errs = append(errs, errors.New("logger is required"))
    }

    if s.db == nil {
        errs = append(errs, errors.New("db is required"))
    }

    return errors.Join(errs...)
}
```

### 6.3. Использование errors.New() для статичных ошибок

Везде, где создаётся ошибка без встраивания в неё динамической информации *(форматированных переменных)*, она обязана создаваться через `errors.New()`. **Использование `fmt.Errorf()` для статичного текста запрещено.**

```go
// Good
var ErrInvalidToken = errors.New("token is invalid")

// Bad
var ErrInvalidPassword = fmt.Errorf("password is invalid")
```

### 6.4. Сбор ошибок в `Validate()`

Если метод `Validate()` у структуры проверяет несколько условий и потенциально может выявить больше одной ошибки, он обязан собирать все найденные ошибки в срез `[]error` и возвращать их через `errors.Join(errs...)`.

Завершать метод на первой же найденной ошибке *(через ранний return err)* запрещено, так как клиентский код должен получать полную картину невалидности объекта за один вызов.

```go
// Good #1
func (p *ServiceParams) Validate() error {
    var errs []error

    if p.Logger == nil {
        errs = append(errs, errors.New("logger is required"))
    }

    if p.DB == nil {
        errs = append(errs, errors.New("database is required"))
    }

    return errors.Join(errs...)
}

// Good #2
// В данном случае проверяется одно поле, поэтому срез ошибок []error избыточен
func (s *Service) Validate() error {
    if s.db == nil {
        return errors.New("database is required")
    }
    return nil
}

// Bad
func (p *ServiceParams) Validate() error {
    if p.Logger == nil {
        return errors.New("logger is required")
    }

    if p.DB == nil {
       return errors.New("database is required")
    }

    return nil
}
```

### 6.5. Текст ошибок

#### Ошибки отсутствия обязательного ресурса

Текст валидационной ошибки, обозначающий отсутствие обязательного поля или ресурса, обязан заканчиваться на **"is required"**.

```go
// Good
var ErrEmptyURL = errors.New("URL is required")

// Bad
var ErrEmptyUser = errors.New("user cannot be nil")
```

#### Ошибки ненайденного ресурса

Текст валидационной ошибки, обозначающий ненайденный ресурс, обязан заканчиваться на **"not found"**.

> Пустой переданный аргумент в **функцию/метод** не считается ненайденным ресурсом!

В качестве примера ненайденного ресурса можно считать **отсутствие** нужного документа в базе данных.

```go
// Good
var ErrUserNotFound = errors.New("user not found")

// Bad
var ErrResourceNotFound = errors.New("resource is required")
```