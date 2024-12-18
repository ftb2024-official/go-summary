### <span style="color: lime">Чем отличаются все виды контекста?</span>
В Go (Golang) есть специальный механизм - контекст(context), который используется для управления временем выполнения программы и для передачи данных между функциями. С помощью контекста можно передавать информацию о том, когда надо остановить выполнение задачи (например, если истек тайм-аут) или завершить выполнение функции при отмене (например, по нажатию на кнопку).

В Go есть несколько видов контекста, которые можно создать на основе базового контекста:
1. `context.Background()`:
    - Это пустой контекст, который обычно используется как `корень` для других контекстов. Он не содержит никаких данных, таймаутов или сигналов отмены.
    - Пример использования: в самом начале программы, когда нужно передать контекст в другие функции.

2. `context.TODO()`:
    - Этот контекст похож на `Background()`, но используется, когда вы `пока не знаете`, какой контекст будет нужен. Обычно он применяется как временное решение, когда вы планируете позже заменить его на другой контекст.
    - Пример использования: когда вы только пишете код, но ещё не решили, какой контекст использовать.

3. `context.WithCancel(parent)`:
    - Этот контекст создаётся на основе существующего контекста (например, `context.Background()`). Он добавляет возможность отменить выполнение программы. Когда вызывается функция отмены (например, `cancel()`), все функции, которые используют этот контекст, прекращают свою работу.
    - Пример использования: нужно остановить выполнение задачи, если, например, пользователь нажал кнопку `Отменить`.

4. `context.WithTimeout(parent, timeout)`:
    - Этот контекст автоматически завершает выполнение через заданное время. Он тоже строится на основе существующего контекста, но завершает задачи по истечении времени.
    - Пример использования: вы хотите завершить операцию, если она не закончилась в течение, например, 5 секунд.

5. `context.WithDeadline(parent, deadline)`:
    - Похож на `WithTimeout()`, но здесь указывается точное время (дедлайн), после которого контекст будет отменён. Например, если дедлайн — это 15:30, то задачи прекращают выполнение точно в это время.
    - Пример использования: завершить работу к определённому моменту времени (например, до конца рабочего дня).

6. `context.WithValue(parent, key, value)`:
	- Этот контекст позволяет хранить и передавать значения (пары `ключ-значение`) между функциями. Это полезно, если нужно передать что-то вроде ID пользователя или другую информацию.
	- Важно: контекст `не рекомендуется` использовать для хранения больших данных или данных, которые нужны для бизнес-логики (например, базовые параметры функции).

Основные различия:
- `Background()` и `TODO()` - это базовые контексты, с которых начинают.
- `WithCancel()` позволяет вручную отменять выполнение.
- `WithTimeout()` и `WithDeadline()` ограничивают время выполнения.
- `WithValue()` используется для передачи данных через контекст.

### <span style="color: lime">Как правильно обрабатывать ошибки?</span>
Oсновные подходы для правильной обработки ошибок в Go:
1. Проверка ошибки сразу после вызова функции: Большинство функций в Go возвращают два значения: результат и ошибку. Типичный подход — проверять наличие ошибки сразу после вызова функции.
```go
file, err := os.Open("path_to_file")
if err != nil {
	// обработка ошибки
	log.Fatal("Не удалось открыть файл: %v", err")
}
defer file.Close()
```

2. Использование `errors.Is()` и `errors.As()`: В Go можно проверять конкретные типы ошибок, используя функции `errors.Is()` и `errors.As()`. Это полезно, если вы хотите выяснить, какая именно ошибка произошла.
```go
if err != nil {
	if errors.Is(err, os.ErrNotExist) {
		fmt.Println("File doesn't exist")
	} else {
		fmt.Println("error occured:", err)
	}
}
// В этом примере мы проверяем, является ли ошибка `err` конкретной ошибкой `os.ErrNotExist`, которая указывает на то, что файл не существует.
```

3. Создание собственных ошибок: Если вам нужно создать свою ошибку, вы можете воспользоваться пакетом `errors` и функцией `errors.New()` для создания новой ошибки.
```go
var ErrInvalidInput = errors.New("Invalid input")

func process(input int) error {
	if input < 0 { return ErrInvalidInput }
	return nil
}

func main() {
	err := process(-1)
	if err != nil {
		fmt.Println(err) // Invalid input
	}
}
// Здесь мы создаём свою ошибку `ErrInvalidInput`, которая возвращается, если введённое значение не соответствует условиям.
```

4. Форматирование ошибок с `fmt.Errorf()`: Если нужно добавить к ошибке дополнительную информацию, можно использовать функцию `fmt.Errorf()`.
```go
func readFile(filename string) error {
	file, err := os.Open(filename)
	if err != nil {
		return fmt.Errorf("не удалось открыть файл %s: %w", filename, err)
	}
	defer file.Close()
	// reading file
	return nil
}

func main() {
	err := readFile("non-existent.txt")
	if err != nil {
		fmt.Println(err)
	}
}
// Здесь `%w` используется для обёртывания оригинальной ошибки, чтобы её можно было потом извлечь и проверить с помощью `errors.Is()`.
```

5. Использование `panic` и `recover`: `panic` и `recover` используются для обработки критических ошибок, которые нельзя или не стоит обрабатывать обычным способом. Однако их использование нужно ограничить крайними случаями, когда программа не может продолжать выполнение.
```go
func riskyFunction() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("critical error occured:", r)
		}
	}()
	panic("smth went wrong")
}

func main() {
	riskyFunction()
	fmt.Println("Программа продолжает выполнение")
}
// Здесь `panic` прерывает обычное выполнение программы, а `recover` позволяет "поймать" панику и продолжить выполнение программы.
```

6. Обработка ошибок в циклах: Если у вас несколько операций, которые могут вызвать ошибку (например, цикл с запросами к API), то рекомендуется проверять ошибки на каждом шаге.
```go
func main() {
	urls := []string{"https://www.example.com", "https://www.invalid-url"}
	for _, url := range urls {
		resp, err := http.Get(url)
		if err != nil {
			log.Printf("Ошибка при запросе к %s: %v", url, err)
			continue
		}
		defer resp.Body.Close()
		// обрабатываем ответ
	}
}
```

Основные советы по обработке ошибок:
- **Проверяйте ошибки сразу**: это упрощает отладку и повышает ясность кода.

- **Используйте `log.Fatal()` или `panic()` только в случае критических ошибок**, когда программа не может продолжить работу.

- **Создавайте собственные ошибки** для более точного описания проблемы.

### <span style="color: lime">Зачем врапать ошибки?</span>
`Враппинг (оборачивание)` ошибок в Go делается для того, чтобы:
1. **Сохранить исходную ошибку** и её контекст.

2. **Добавить больше информации** о том, что пошло не так, в каком месте программы произошла ошибка и почему.

Пример, когда враппинг полезен: Допустим, у вас есть функция для чтения файла. Если файл не удаётся открыть, полезно знать не только, что операция не удалась, но и почему она не удалась (например, из-за отсутствия файла) и где в коде это произошло.
Пример без `враппинга`:
```go
func readFile(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    // Обработка файла...
    return nil
}

func main() {
	err := readFile("file.txt")
	if err != nil {
	    fmt.Println(err) // Выведет только основную ошибку
	}
}

// В этом случае ошибка будет просто передана вверх по стеку. Мы узнаем только то, что что-то пошло не так, но не узнаем контекст, где именно произошла ошибка, и что за файл мы пытались открыть.
```

Пример c `враппингом`:
```go
func readFile(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return fmt.Errorf("не удалось открыть файл %s: %w", filename, err)
    }
    defer file.Close()
    // Обработка файла...
    return nil
}

func main() {
	err := readFile("file.txt")
	if err != nil {
	    fmt.Println(err) // Теперь вывод будет содержать больше информации
	}
}

// Здесь `%w` в функции `fmt.Errorf` используется для обёртывания исходной ошибки (`err`). Теперь мы не только получаем информацию о самой ошибке, но и можем добавить пояснение, например, указав, что ошибка произошла при открытии конкретного файла.
```

`Враппинг` ошибок делает ваш код более **читаемым** и **диагностируемым**. Он помогает видеть полную картину ошибки — что произошло, где это произошло и почему. Это особенно полезно в больших системах или при работе с внешними ресурсами (файлы, базы данных, сети), где причину ошибки может быть не очевидно сразу.

### <span style="color: lime">Способы врапинга ошибок:</span>
1. Использование `fmt.Errorf` с `%w`: Это один из самых популярных и простых способов враппинга ошибок. С помощью `%w` можно обернуть исходную ошибку и добавить больше контекста.
```go
func openFile(filename string) error {
	file, err := os.Open(filename)
	if err != nil {
		// Враппинг ошибки с добавлением информации о файле
		return fmt.Errorf("ошибка при открытии файла %s: %w", filename, err)
	}
	defer file.Close()
	// Работа с файлом...
	return nil
}
	
func main() {
	err := openFile("config.json")
	if err != nil { fmt.Println(err) }
}

// В этом примере оригинальная ошибка, возвращённая `os.Open`, обёрнута с помощью `%w` в `fmt.Errorf`. Это позволяет не только сохранить оригинальную ошибку, но и добавить к ней дополнительную информацию.
```

2. Использование `errors.Join` (Go 1.20+): Начиная с Go 1.20, появилась новая функция `errors.Join`, которая позволяет комбинировать несколько ошибок в одну. Это полезно, если у вас есть несколько независимых ошибок, которые нужно вернуть вместе.
```go
func doSomething() error {
    err1 := errors.New("ошибка 1")
    err2 := errors.New("ошибка 2")
    
    // Объединяем несколько ошибок
    return errors.Join(err1, err2)
}

func main() {
    err := doSomething()
    if err != nil {
        fmt.Println("Произошли ошибки:", err)
    }
}

// С помощью `errors.Join` можно объединить несколько ошибок и передать их как одну. При этом оригинальные ошибки будут сохранены и доступны для дальнейшей проверки.
```

3. Создание пользовательских типов ошибок: Иногда бывает полезно создать собственный тип ошибки, который включает дополнительную информацию. Это можно сделать, определив структуру, которая реализует интерфейс `error`.
```go
type MyError struct {
    Op  string
    Err error
}

func (e *MyError) Error() string {
    return fmt.Sprintf("операция %s: ошибка %v", e.Op, e.Err)
}

func (e *MyError) Unwrap() error {
    return e.Err
}

func doSomething() error {
    return &MyError{Op: "чтение файла", Err: fmt.Errorf("файл не найден")}
}

func main() {
    err := doSomething()
    if err != nil {
        fmt.Println(err)
    }
}

// Здесь создаётся пользовательский тип ошибки `MyError`, который сохраняет оригинальную ошибку и добавляет контекст операции. Метод `Unwrap()` позволяет использовать стандартные функции `errors.Is()` и `errors.As()` для извлечения оригинальной ошибки.
```

4. Использование `errors.Unwrap` для извлечения оригинальной ошибки: После враппинга ошибок с помощью `%w` или других методов, можно использовать функцию `errors.Unwrap` для извлечения исходной ошибки. Это полезно, если нужно узнать, что вызвало исходную ошибку.
```go
func openFile(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return fmt.Errorf("ошибка при открытии файла %s: %w", filename, err)
    }
    defer file.Close()
    return nil
}

func main() {
    err := openFile("config.json")
    if err != nil {
        // Извлечение оригинальной ошибки
        if errors.Is(err, os.ErrNotExist) {
            fmt.Println("Файл не существует")
        } else {
            fmt.Println(err)
        }
    }
}

// Функция `errors.Is()` позволяет проверить, является ли обёрнутая ошибка конкретной ошибкой, например, `os.ErrNotExist`.
```

Резюме:
1. **`fmt.Errorf("%w")`** — основной способ враппинга ошибок в стандартной библиотеке Go. Используйте его для сохранения исходной ошибки и добавления контекста.

2. **`errors.Join` (Go 1.20+)** — объединение нескольких ошибок в одну.

3. **Пользовательские типы ошибок** — создание собственных типов для более гибкой обработки ошибок.

4. **`errors.Unwrap` и `errors.Is()`** — извлечение оригинальной ошибки и проверка типа ошибки, чтобы понять её природу.

### <span style="color: lime">Зачем нужен go mod?</span>
`go mod` — это инструмент управления зависимостями и версионирования в Go, который был введён в версии Go 1.11 (и стал стандартом с Go 1.13). Его основная цель — упростить работу с внешними библиотеками (зависимостями), управление версиями этих библиотек и создание модульных приложений. Он заменил старые механизмы управления зависимостями, такие как GOPATH и `dep`.

Основные задачи `go mod`:
1. **Управление зависимостями**: `go mod` позволяет вашей программе автоматически загружать и отслеживать зависимости (другие пакеты, которые ваш код использует).

2. **Версионирование**: Вы можете фиксировать определённые версии зависимостей, чтобы избежать проблем с несовместимыми изменениями, которые могут быть внесены в будущие версии внешних библиотек.

3. **Упрощение сборки и развёртывания**: Программы и библиотеки с использованием `go mod` становятся самодостаточными и не зависят от конкретной структуры проекта (например, необходимости хранить проект внутри `GOPATH`).

Основные функции `go mod`:
1. `go mod init`: Команда для инициализации нового модуля в проекте. Она создаёт файл `go.mod`, который будет хранить информацию о модуле и его зависимостях.

2. `go get`: Команда для добавления зависимостей в проект. Например, если вы хотите подключить библиотеку, вы можете использовать `go get`.

3. `go mod tidy`: Команда для очистки `go.mod` от неиспользуемых зависимостей и для проверки, что все используемые зависимости определены. Она также обновляет файл `go.sum`, который содержит контрольные суммы всех загруженных модулей для валидации.

4. `go.mod` и `go.sum`:
	- **`go.mod`** - основной файл, в котором хранится информация о модуле и его зависимостях (их названия и версии).
	
	- **`go.sum`** - файл, который автоматически создаётся и обновляется при загрузке зависимостей. В нём хранятся контрольные суммы (хэши) всех зависимостей, что позволяет защититься от несанкционированных изменений в библиотеке при последующих сборках.

### <span style="color: lime">Из чего состоит файл go.mod?</span>
Структура файла `go.mod`:
1. **`module`** — описание имени модуля.

2. **`go`** — версия Go, используемая для разработки и сборки.

3. **`require`** — список зависимостей с указанием версий.

4. **`replace`** — замена одной зависимости на другую (например, для локальной разработки или исправления).

5. **`exclude`** — исключение определённых версий зависимостей.

6. **`retract`** — аннулирование определённых версий модуля (Go 1.16+).

Пример полного файла `go.mod`:
```go
module github.com/username/project

go 1.20

require (
    github.com/gin-gonic/gin v1.7.0
    github.com/jinzhu/gorm v1.9.16
)

replace github.com/old/package => github.com/new/package v1.0.0

exclude github.com/bad/package v1.2.3
```

### <span style="color: lime">Что значит indirect в go.mod?</span>
В файле `go.mod` Go использует пометку **`indirect`** для зависимостей, которые ваш проект использует **косвенно** (через другие библиотеки), но напрямую вы их не импортируете в своём коде.

Когда возникает пометка `indirect`?:
- Если ваша программа напрямую зависит от библиотеки `A`, и библиотека `A` в свою очередь зависит от другой библиотеки `B`, то библиотека `B` будет добавлена в ваш файл `go.mod` с пометкой `indirect`.
- Это означает, что библиотека `B` — не явная зависимость вашего кода, но она необходима для работы вашей программы, потому что она требуется для библиотеки `A`.