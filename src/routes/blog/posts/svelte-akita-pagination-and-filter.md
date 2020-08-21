# Управление состоянием в svelte 3 с akita.js

Проблема с документацией к akita в том, что все примеры написаны с исползованием TypeScript и декораторов, сходу и не поймешь как подступиться, если ты новичек. В этой статье мы разберем как реализовать компонент таблицы данных DataGrid с серверной пагинацией, фильтрацией и сортировкой.

Что такое akita.js  (написать про зависимость от rxjs, но всё ок, т.к. есть tree-shakong)
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
import { createEntityStore } from '@datorama/akita';

export function createMetricsEntity() {
  const entityStore = createEntityStore({}, { name: 'metrics' });
}
```

Теперь нужно научить сервис делать запросы к серверу в тот момент когда изменился один из параметров. Для этого добавим переменную `params` типа `BehaviorSubject` и метод для её обновления, что бы можно было _обновлять_ парамеры, без их перезаписи.

```javascript
// src/entities/metrics.js
import { createEntityStore } from '@datorama/akita';
import { get } from 'svelte/store';
import merge from 'lodash.merge'
import { BehaviorSubject } from 'rxjs';
import { combineLatest, switchMap } from 'rxjs/operators';

export function createMetricsEntity() {
  // хранилище состояния
  const entityStore = createEntityStore({}, { name: 'metrics' });
  // параметры запроса
  const params$ = new BehaviorSubject({ 
    start: null, 
    end: null,
    order: [{ by: 'date', direction: 'desc' }],
    offset: 0,
    limit: 10,
  });

  const updateParams = (params) => {
    // с помощью метода get из svelte/store можно работать с объектами rxjs
    const currentParams = get(params$); // todo: ссылку на get
    return merge({}, currentParams, params);
  };

  const pagination$ = combineLatest(
    paginator.pageChanges,
    params$.pipe(
      tap(() => paginator.clearCache()),
    ),
  ).pipe(
    switchMap(([nextPage, params]) => {
      return paginator.getPage(() => {
        let page = params.page > 0 ? params.page : nextPage;
        params.offset = (page - 1) * QUERY_LIMIT;
        delete params.page;
        const p = ds.getList(params).then(result => ({
          ...result,
          perPage: QUERY_LIMIT,
          currentPage: page,
        }));
        params$.next(params);
        return p;
      });
    }),
    startWith({ data: [], total: 0 }),
  );
}
```
Разберемся с импортами:
- [get](https://svelte.dev/docs#get) из `svelte/store` для того что бы получить значения params$ не подписываясь на него явным образом.
- [merge](https://lodash.com/docs/4.17.15#merge) Для того что бы объединить два старые и новые параметры. Можно было бы использовать object spread syntax, но из за того, что в параметрах есть сложное свойство order, запись была бы сложнее
- [BehaviorSubject] (https://www.learnrxjs.io/learn-rxjs/subjects/behaviorsubject) Для того 



(скрытый тест)
`BehaviorSubject` - это такой аналог writable store, его можно инициализировать определенным значением, подписаться и получить сразу же получить это значение, при первом же вызове обработчика подписки.



Ответить на вопрос почему название entity?