# Диаграмма вариантов использования (use case)

Диаграмма вариантов использования отображает функциональные возможности рекомендательной системы книг на основе RAG, реализованной в виде Telegram-бота для ЦБС Петроградского района. На диаграмме представлены два основных актора — Пользователь и Администратор ЦБС, а также внешняя RAG-система, обеспечивающая генерацию рекомендаций.

```
@startuml
left to right direction
skinparam packageStyle rectangle
skinparam actorStyle awesome

actor "Пользователь\n(Telegram)" as User
actor "Администратор\nЦБС" as Admin
actor "RAG-система" as RAG <<Внешняя система>>

rectangle "Рекомендательная система книг\n(Telegram Bot ЦБС Петроградского района)" {

  package "Авторизация" {
    usecase "Начать работу с ботом\n(/start)" as UC1
    usecase "Зарегистрироваться" as UC2
  }

  package "Каталог" {
    usecase "Просмотреть карточку книги" as UC4
    usecase "Фильтровать по жанру / автору" as UC5
    usecase "Применить фильтры ЦБС\n(наличие, филиал и др.)" as UC9
  }

  package "Рекомендации" {
    usecase "Получить рекомендации книг" as UC6
    usecase "Настроить предпочтения" as UC7
    usecase "Просмотреть историю запросов" as UC8
    usecase "Добавить книгу в избранное" as UC10
    usecase "Просмотреть избранное" as UC14
  }

  package "Администрирование" {
    usecase "Управлять каталогом книг" as UC11
    usecase "Просматривать статистику" as UC12
    usecase "Управлять пользователями" as UC13
  }
}

' Пользователь
User --> UC1
User --> UC4
User --> UC5
User --> UC6
User --> UC7
User --> UC8
User --> UC9
User --> UC10
User --> UC14

' Администратор
Admin --> UC1
Admin --> UC11
Admin --> UC12
Admin --> UC13

' Зависимости
UC1 .> UC2 : <<extend>>
UC6 .> UC7 : <<extend>>
UC6 .> UC9 : <<extend>>
UC4 .> UC10 : <<extend>>

' RAG
UC6 --> RAG
@enduml
```

<img width="814" height="1380" alt="image" src="https://github.com/user-attachments/assets/656060fd-f826-4b20-a0ee-b357074c5e82" />



# Диаграмма деятельности (activity)

Диаграмма деятельности отображает основной сценарий работы рекомендательной системы книг на основе RAG, реализованной в виде Telegram-бота для ЦБС Петроградского района. Диаграмма описывает полный поток взаимодействия между тремя участниками — пользователем, ботом и RAG-системой — от момента запуска бота до получения персонализированных рекомендаций.


```
@startuml
skinparam backgroundColor WhiteSmoke
skinparam activity {
  BackgroundColor LightSkyBlue
  BorderColor SteelBlue
  ArrowColor DimGray
  FontSize 13
}
skinparam activityDiamond {
  BackgroundColor LightYellow
  BorderColor Goldenrod
}

|#LightCyan|Пользователь|
start
:Отправить команду /start;

|#LightGreen|Telegram Bot|
if (Пользователь\nзарегистрирован?) then (нет)
  :Создать профиль пользователя;
  :Сохранить в базе данных;
endif

:Отобразить приветственное сообщение;

(A)

|#LightGreen|Telegram Bot|
:Выбрать действие;

if (Открыть избранное?) then (да)
  :Показать список\nизбранных книг;
  |Пользователь|
  :Просмотреть книги;
  (A)
endif

|#LightGreen|Telegram Bot|
:Предложить ввести запрос;

|#LightCyan|Пользователь|
:Ввести запрос\n(тема / жанр / автор);

|#LightGreen|Telegram Bot|
if (Запрос корректен?) then (нет)
  :Запросить уточнение;
  |Пользователь|
  :Уточнить запрос;
  |#LightGreen|Telegram Bot|
endif

:Проверить предпочтения пользователя;
:Предложить применить фильтры ЦБС;

|Пользователь|
if (Хочет применить\nфильтры?) then (да)
  :Выбрать фильтры\n(наличие, филиал и др.);
  |#LightGreen|Telegram Bot|
  :Добавить фильтры\nв контекст запроса;
endif

|#LightGreen|Telegram Bot|
:Сформировать контекстный запрос к RAG;

|#LightSalmon|RAG-система|
:Принять запрос от бота;
:Выполнить векторный поиск\nпо базе знаний;

if (Релевантные книги\nнайдены?) then (нет)
  :Вернуть ответ:\n"Ничего не найдено";
  |#LightGreen|Telegram Bot|
  :Сообщить пользователю\nоб отсутствии результатов;
  |Пользователь|
  :Изменить запрос;
  (A)
else (да)
  |#LightSalmon|RAG-система|
  :Ранжировать результаты\nпо релевантности;
  :Сформировать список\nрекомендаций с описанием;
  :Передать результаты боту;
endif

|#LightGreen|Telegram Bot|
:Получить список рекомендаций;
:Отформатировать ответ;
:Отправить рекомендации;
:Сохранить запрос в историю;

|Пользователь|
:Получить список\nрекомендованных книг;

if (Хочет добавить\nв избранное?) then (да)
  :Выбрать книгу;
  |#LightGreen|Telegram Bot|
  :Сохранить книгу\nв избранное пользователя;
  :Подтвердить сохранение;
  |Пользователь|
endif

if (Хочет узнать\nбольше о книге?) then (да)
  :Запросить карточку книги;
  |#LightGreen|Telegram Bot|
  :Отобразить подробную\nкарточку книги;
  |Пользователь|
endif

if (Хочет новые\nрекомендации?) then (да)
  :Ввести новый запрос;
  (A)
else (нет)
  :Завершить сессию;
endif

stop
@enduml
```

<img width="1214" height="2991" alt="image" src="https://github.com/user-attachments/assets/550a338b-56f9-47df-8138-5f2976eb5620" />

# Диаграмма классов (classes)

Архитектура построена по принципу разделения ответственности: Telegram-бот отвечает за взаимодействие с пользователем, RAG-сервис — за генерацию рекомендаций, а база данных — за хранение пользователей, книг и избранного.

```
@startuml
class User {
  +id: int
  +telegramId: string
  +preferences: string
}

class Book {
  +id: int
  +title: string
  +author: string
  +genre: string
  +description: string
  +available: bool
}

class Favorite {
  +id: int
  +userId: int
  +bookId: int
}

class Query {
  +text: string
  +filters: string
}

class TelegramBot {
  +handleStart()
  +handleQuery()
  +sendRecommendations()
}

class RAGService {
  +retrieve(query)
  +rank(results)
  +generateAnswer()
}

class Database {
  +saveUser()
  +saveQuery()
  +getBooks()
  +saveFavorite()
}

' Связи
User "1" -- "0..*" Favorite
Book "1" -- "0..*" Favorite

TelegramBot --> RAGService
TelegramBot --> Database
RAGService --> Database

TelegramBot --> User
TelegramBot --> Query
@enduml
```

<img width="758" height="496" alt="image" src="https://github.com/user-attachments/assets/84f5615f-616d-4878-886d-303ca5737cf9" />


# Диаграмма последовательности (sequence)

Диаграмма отражает взаимодействие между пользователем, ботом, RAG-системой и базой данных в рамках одного запроса рекомендаций.

```
@startuml
actor User
participant "Telegram Bot" as Bot
participant "RAG-система" as RAG
database DB

User -> Bot: /start
Bot -> DB: проверить пользователя
DB --> Bot: результат

User -> Bot: ввод запроса
Bot -> DB: получить предпочтения
Bot -> RAG: отправить запрос

RAG -> DB: поиск книг
DB --> RAG: список книг

RAG -> RAG: ранжирование
RAG --> Bot: рекомендации

Bot -> DB: сохранить запрос
Bot --> User: отправить рекомендации

User -> Bot: добавить в избранное
Bot -> DB: сохранить книгу
@enduml
```

<img width="514" height="603" alt="image" src="https://github.com/user-attachments/assets/b25a8d2a-bd86-4713-a517-bcf55a569f3f" />


# Диаграмма состояний (state)

Диаграмма отражает состояния пользователя от момента входа в систему до получения рекомендаций и взаимодействия с ними. Показывает жизненный цикл пользователя в системе

```
@startuml

[*] --> Неавторизован

Неавторизован --> Авторизован : /start

Авторизован --> ВводЗапроса : начать поиск
ВводЗапроса --> Обработка : отправка запроса
Обработка --> ПолучениеРекомендаций : успех
Обработка --> Ошибка : нет результатов

ПолучениеРекомендаций --> ПросмотрКниги
ПросмотрКниги --> ДобавлениеВИзбранное

ДобавлениеВИзбранное --> Авторизован
Ошибка --> ВводЗапроса

@enduml
```

<img width="426" height="880" alt="image" src="https://github.com/user-attachments/assets/fa89f48b-f514-4116-bfac-4d13bf9a3b7c" />


# Прототип интерфейса (wireframe)

Прототип отражает пользовательский интерфейс системы рекомендаций книг на основе RAG: от ввода свободного запроса до получения списка релевантных книг и объяснения от LLM. Показывает ключевые экраны, с которыми пользователь работает для решения задачи подбора книги.

Ссылка на [прототип](https://github.com/danyakr/prog_8_uml/blob/main/PROTOTYPE.md) с пояснениями в Markdown и Mermaid.

https://www.figma.com/proto/evu65LQFKqeuhvvxHQZhj7/%D0%92%D0%9A%D0%A0-%E2%80%94-RAG-Book-Recommender-%E2%80%94-Wireframes?node-id=0-1&t=nsgK4x3n0oGHh9Mm-1

<img width="1287" height="753" alt="image" src="https://github.com/user-attachments/assets/af439460-61a1-4964-96d6-96a4aa2e2b9c" />
