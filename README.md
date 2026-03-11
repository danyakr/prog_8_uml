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
  }
  package "Рекомендации" {
    usecase "Получить рекомендации книг" as UC6
    usecase "Настроить предпочтения" as UC7
    usecase "Просмотреть историю запросов" as UC8
  }
  package "Администрирование" {
    usecase "Управлять каталогом книг" as UC11
    usecase "Просматривать статистику" as UC12
    usecase "Управлять пользователями" as UC13
  }
}

' Связи пользователя
User --> UC1
User --> UC4
User --> UC5
User --> UC6
User --> UC7
User --> UC8

' Зависимости
UC1 .> UC2 : <<extend>>
UC6 .> UC7 : <<extend>>

' RAG-система
UC6 --> RAG

' Администратор
Admin --> UC1
Admin --> UC11
Admin --> UC12
Admin --> UC13

@enduml
```

<img width="752" height="1124" alt="image" src="https://github.com/user-attachments/assets/95c5a72e-a07f-49ac-ae54-eb1c3bbd6244" />

