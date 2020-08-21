# Управление состоянием в svelte 3 с akita.js

Проблема с документацией к akita в том, что все примеры написаны с исползованием TypeScript и декораторов, сходу и не поймешь как подступиться, если ты новичек. В этой статье мы разберем как реализовать компонент таблицы данных DataGrid с серверной пагинацией, фильтрацией и сортировкой.

Что такое akita.js  
Что такое svelte 3  

Установка зависимостей и настройка проекта (скрытый текст)
Что такое Store
Что такое Query
Что такое EnityStore
Что такое EntityQuery

## Предметная область
В данном примере рассматривается создание компонента таблицы для отчета по рекламной компании.

## Слой данных
Предположим нам нужно интегрироваться с API вида   
```
POST /api​/reports​/query 

Schema
{
  "start": "2020-08-21",
  "end": "2020-08-21",
  "order": [
    {
      "direction": "asc"
    }
  ],
  "limit": 0,
  "offset": 0
}
```

Создадим сервис, который будет отвечать за взаимодействие с сервером и управление состоянием.

Для того что бы создать хранилище состояния существует фукция `createEntityStore`. В результате получится объект со следующими методами:

- [set()](https://datorama.github.io/akita/docs/entities/entity-store#set])
- add()
- update()
- remove()
- upsert()
- upsertMany()
- replace()
- move()
- setLoading()
- setError()

Мы будем его использовать для хранения ответа от сервера и выборки данных, которые необходимы компоненту. 

Создадим файл-заготовку `src/entities/metrics.js`: 

```javascript
// src/entities/metrics.js
export function createMetricsEntity() {
}
```

Создадим хранилище

```javascript
// src/entities/metrics.js
export function createMetricsEntity() {
  const entityStore = createEntityStore({}, { name: 'metrics' });
}
```

Теперь нужно научить сервис делать запросы к серверу. Запрос будет иметь такую структуру:

Ответить на вопрос почему название entity?