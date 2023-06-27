## Изменение элемента

Обновить данные в БД можно следующими подходами:

* Метод ```save()```:

Вы можете изменить поля объекта модели, а затем вызвать метод save() для сохранения изменений в базе данных. 
Например:

```python
obj = Entry.objects.get(id=1)
print(obj.number_of_comments, obj.rating)
# 2 0.0
obj.number_of_comments += 1
obj.rating = 5.0
obj.save()

obj_new = Entry.objects.get(id=1)
print(obj.number_of_comments, obj.rating)
# 3 5.0
```

* Метод ```update()```: 

Этот метод позволяет обновить одно или несколько полей для множества объектов модели, соответствующих определенному 
условию фильтрации. Обновление выполняется непосредственно в базе данных без извлечения объектов.

Метод update() применяется к QuerySet и позволяет обновлять поля в базе данных для всех объектов, 
соответствующих определенному условию фильтрации. Он выполняет обновление данных непосредственно в базе данных 
и не извлекает объекты модели. 

Преимущества метода update() включают более эффективное выполнение, поскольку он работает непосредственно с базой данных, 
и возможность обновления нескольких полей одновременно. Однако, метод update() не вызывает сигналы модели и не выполняет 
обработку хуков модели, таких как переопределенные методы save().

Пример:

```python
print(Entry.objects.filter(blog__name='Путешествия по миру').values('id', 'rating'))
"""
<QuerySet [
{'id': 1, 'rating': 5.0}, 
{'id': 2, 'rating': 5.0}, 
{'id': 3, 'rating': 4.7}, 
{'id': 4, 'rating': 3.3}, 
{'id': 5, 'rating': 3.4}
]>
"""
Entry.objects.filter(blog__name='Путешествия по миру').update(rating=0.0)
print(Entry.objects.filter(blog__name='Путешествия по миру').values('id', 'rating'))
"""
<QuerySet [
{'id': 1, 'rating': 0.0}, 
{'id': 2, 'rating': 0.0}, 
{'id': 3, 'rating': 0.0}, 
{'id': 4, 'rating': 0.0}, 
{'id': 5, 'rating': 0.0}
]>
"""
```

Метод ```bulk_update()```:

```bulk_update()``` был добавлен в Django 3.2 и предоставляет альтернативный способ массового обновления множества 
объектов модели. В отличие от update(), bulk_update() работает с уже извлеченными объектами модели. 
Вы должны создать QuerySet объекты, изменить значения полей в этих объектах и передать их в метод bulk_update(). 
Метод bulk_update() выполняет обновление данных в базе данных с использованием оптимизированных SQL-запросов, 
что может привести к повышению производительности при обновлении большого количества объектов. 

Кроме того, bulk_update() может вызывать сигналы модели и обрабатывать хуки модели, что делает его более полезным 
в некоторых сценариях.

Таким образом, основные отличия между update() и bulk_update() заключаются в том, что update() работает непосредственно 
с QuerySet и выполняет обновление данных в базе данных без извлечения объектов, а bulk_update() работает 
с уже извлеченными объектами модели и может обрабатывать сигналы и хуки модели. 
Выбор между ними зависит от ваших потребностей и контекста использования.

Пример:

```python
objs = Entry.objects.filter(blog__name='Путешествия по миру')
print(objs.values('id', 'rating', "number_of_comments"))
"""
<QuerySet [
{'id': 1, 'rating': 0.0, 'number_of_comments': 4}, 
{'id': 2, 'rating': 0.0, 'number_of_comments': 14}, 
{'id': 3, 'rating': 0.0, 'number_of_comments': 7}, 
{'id': 4, 'rating': 0.0, 'number_of_comments': 2}, 
{'id': 5, 'rating': 0.0, 'number_of_comments': 4}
]>
"""
for obj, rating, number_of_comments in zip(objs, (1., 2., 3., 4., 5.), (2, 5, 3, 2, 1)):
    obj.rating = rating
    obj.number_of_comments = number_of_comments

Entry.objects.bulk_update(objs, ['rating', 'number_of_comments'])

print(Entry.objects.filter(blog__name='Путешествия по миру').values('id', 'rating', "number_of_comments"))
"""
<QuerySet [
{'id': 1, 'rating': 1.0, 'number_of_comments': 2}, 
{'id': 2, 'rating': 2.0, 'number_of_comments': 5}, 
{'id': 3, 'rating': 3.0, 'number_of_comments': 3}, 
{'id': 4, 'rating': 4.0, 'number_of_comments': 2}, 
{'id': 5, 'rating': 5.0, 'number_of_comments': 1}
]>
"""
```

## Удаление элемента

Метод удаления называется delete(). Этот метод немедленно удаляет объект и возвращает количество удаленных объектов и 
словарь с количеством удалений на тип объекта. Пример:
```python
obj = Entry.objects.get(id=1)
obj.delete()
```
Можно удалять как одиночные элементы так и группы элементов
```python
Entry.objects.filter(pub_date__year=2023).delete()
```

Есть еще множественное удаление ```bulk_delete()```

* Метод bulk_delete() позволяет массово удалять объекты моделей из базы данных.
* Он доступен только в Django 4.0 и более поздних версиях.
* Принимает в качестве аргумента queryset (запрос), который определяет набор объектов, которые нужно удалить.
* Метод выполняет массовое удаление объектов без получения каждого объекта отдельно.

* Пример использования:

```python
Entry.objects.filter(pub_date__year=2023).bulk_delete()
```

```Entry.objects.filter(pub_date__year=2023).delete()```, удалит все объекты, удовлетворяющие условию ```pub_date__year=2023```
из базы данных.

Однако, следует отметить, что ```delete()``` метод работает немного иначе по сравнению с ```bulk_delete()``` 

delete() метод:

* Удаляет объекты из базы данных в соответствии с условием фильтрации.
* Выполняет удаление объектов путем отправки отдельного запроса на удаление для каждого объекта.
* Возвращает количество удаленных объектов.

```bulk_delete()``` метод, с другой стороны, представляет собой новый метод, добавленный в Django 4.0, 
который оптимизирует удаление объектов, выполняя массовое удаление без получения каждого объекта отдельно. 
Он работает более эффективно для удаления большого количества объектов.