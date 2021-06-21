# Создание дашборда с метриками взаимодействия пользователей с карточками статей Яндекс.Дзена

## Данные
Входные данные — агрегированная таблица на основе сырых данных о событиях взаимодействия пользователей с карточками:
- идентификатор карточки;
- тематика карточки;
- тематика источника;
- возрастная группа пользователей;
- дата;
- количество взаимодействий.

## Задачи
- **Образовательная** - отработать навыки выгрузки данных, создания дашбордов и подготовки презентаций.
- **Практическая** - используя данные Яндекс.Дзена построить дашборд с метриками взаимодействия пользователей с карточками статей.
    - Ответить на вопросы:
        - Cколько взаимодействий пользователей с карточками происходит в системе с разбивкой по темам карточек?
        - Как много карточек генерируют источники с разными темами?
        - Как соотносятся темы карточек и темы источников?

## Используемые библиотеки
- `pandas`
- `sqlalchemy`
## Используемая система BI
- `Tableau`