## Домашнее задание № 2

- установить MongoDB одним из способов: ВМ, докер;
- заполнить данными;
- написать несколько запросов на выборку и обновление данных.

### Выполнение ДЗ № 2

Запускаем сервер mongodb в докере (см. файл docker-compose.yml) командой:

```
docker compose up -d
```

(либо без "-d", если хотим наблюдать за "жизнью" контейнера)

В docker-compose.yml также прописано поднятие mongo-express, чтобы можно было зайти через браузер на сервер mongo (http://localhost:8081).  
Также можно использовать GUI, например:

MongoDB Compass (https://www.mongodb.com/products/tools/compass)  
Robo 3T (https://robomongo.org/)

Запустим bash в контейнере (где `lesson02-mongo-1` - имя контейнера, в котором запущена mongodb):

```
docker exec -it lesson02-mongo-1 bash
```

и загрузим данные из папки /data.  
В папке /data находится файл, скачанный отсюда: https://github.com/ozlerhakan/mongodb-json-files/blob/master/datasets/books.json

Для загрузки файла в БД выполним команду:

```
mongoimport -h localhost:27017 -u root -p example --authenticationDatabase admin -d db_lesson02 -c books --file /home/data/books.json
```

получим сообщения:

```
connected to: mongodb://localhost:27017/
431 document(s) imported successfully. 0 document(s) failed to import.
```

Закрываем этот терминал.

Открываем новый, запускаем `mongosh` командой:

```
docker exec -it lesson02-mongo-1 mongosh -u root -p example
```

Переключаемся на нашу БД `db_lesson02`:

```
use db_lesson02
```

1. Поиск данных

- найдем книгу с заголовком "Hello! Python":

```
db.getCollection('books').find({'title': 'Hello! Python'})
```

mongodb вернет даныне по одной книге.

- найдем все книги, с количеством страниц более (включительно) 1050 страниц (должно вернуться 3 книги):

```
db.getCollection('books').find({'pageCount': { '$gte': 1050 }})
```

- найти все книги с категорией "Mobile" (1 книга):

```
db.getCollection('books').find({categories: "Mobile"})
```

2. Обновление данных:

- добавим категорию "Web" для книги с _id = 1:

```
db.getCollection('books').update({_id: 1}, {$push: {categories: "Web"}})
```

проверим:

```
db.getCollection('books').find({_id: 1})
```

получим 

```
...
categories: [ 'Open Source', 'Mobile', 'Web' ]
...
```

- прибавить к количеству страниц 1000000 всем книгам:

```
db.getCollection('books').update({}, {$inc: {pageCount: 1000000}}, {multi: true})
```

- обновить книгу с названием "Война и мир", если ее нет, то вставить в коллекцию:

```
db.getCollection('books').update({"title": "Война и мир"}, {$set: {"title":"Война и мир","isbn":"9785447237509","pageCount":1300,"publishedDate":{"$date":"1865-01-01T00:00:00.000-0000"},"thumbnailUrl":"https://www.test.com","shortDescription":"Роман-эпопея Льва Николаевича Толстого","longDescription":"Роман-эпопея Льва Николаевича Толстого, описывающий русское общество в эпоху войн против Наполеона в 1805—1812 годах. Эпилог романа доводит повествование до 1820 года.","status":"PUBLISH","authors":["Лев Николаевич Толстой"],"categories":["Роман-эпопея"]} }, {upsert: true})
```

проверим

```
db.getCollection('books').find({'title': 'Война и мир'})
```

вернется 1 добавленный документ.

3. Удаление данных

- удалим все книги, у которых количество страниц > 10000
(останется 1 книга - "Война и мир". У нее поставили кол. страниц 1300, а остальным книгам увеличивали ранее количество страниц на 1000000):

```
db.getCollection('books').deleteMany({'pageCount': {$gt: 10000}})
```

проверим

```
db.getCollection('books').count()
```

получим

```
1
```
