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
:Предложить ввести запрос;

|Пользователь|
:Ввести запрос\n(тема / жанр / автор);

|#LightGreen|Telegram Bot|
if (Запрос корректен?) then (нет)
  :Запросить уточнение;
  |Пользователь|
  :Уточнить запрос;
  |#LightGreen|Telegram Bot|
endif

:Проверить предпочтения пользователя;
:Сформировать контекстный запрос к RAG;

|#LightSalmon|RAG-система|
:Принять запрос от бота;
:Выполнить векторный\nпоиск по базе знаний;

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
:Отформатировать ответ для Telegram;
:Отправить рекомендации пользователю;
:Сохранить запрос в историю;

|Пользователь|
:Получить список рекомендованных книг;

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
  |Пользователь|
  :Завершить сессию;
endif

stop
@enduml
```

<img width="1214" height="2075" alt="image" src="https://github.com/user-attachments/assets/d90b0e90-5a6b-4fc5-a836-2bdecc1310b8" />


