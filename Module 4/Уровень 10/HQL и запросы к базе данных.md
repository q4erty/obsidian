## 1. Что такое HQL?

**HQL** — это расширение объектно-ориентированного подхода на уровень запросов.

- В обычном **SQL** вы пишете: `SELECT * FROM user_table`.
    
- В **HQL** вы пишете: `FROM User`.
    

**Главное отличие:** HQL понимает связи между вашими объектами (ассоциации), наследование и полиморфизм. Hibernate сам переводит HQL-запрос в оптимальный SQL-код для конкретной базы данных.

---

## 2. Базовые запросы

### Простая выборка

Самый простой запрос — это выборка всех объектов класса:

- `FROM Employee` — вернет список всех объектов типа Employee.
    

### Выбор конкретных полей (Projection)

Если вам нужны не все объекты целиком, а только их имена:

- `SELECT name FROM User` — вернет список строк (`List<String>`).
    
- `SELECT DISTINCT name FROM User` — вернет только уникальные имена.
    

### Выполнение в коде:

Java

```
// Создаем объект запроса
Query<Employee> query = session.createQuery("from Employee", Employee.class);

// Получаем список результатов
List<Employee> results = query.list();

// Если ожидаем только одну запись
Employee emp = query.uniqueResult(); 
```

---

## 3. Фильтрация и работа со связями

В HQL не нужно вручную прописывать сложные `JOIN`, если у вас правильно настроены связи в Entity. Можно обращаться к полям связанных объектов через точку.

**Пример:** Найти задачи, закрепленные за Иваном Ивановичем.

- **SQL:** `SELECT t.* FROM task t JOIN employee e ON t.employee_id = e.id WHERE e.name = 'Иван Иванович'`
    
- **HQL:** `FROM EmployeeTask WHERE employee.name = 'Иван Иванович'`
    

Hibernate сам поймет, что нужно сделать `JOIN` с таблицей сотрудников.

---

## 4. Параметры в запросах и безопасность

Никогда не склеивайте строки в запросах (например, `"WHERE name = " + name`), так как это открывает путь для **SQL-инъекций**.

### Именованные параметры

Используйте двоеточие `:` для обозначения параметров:

Java

```
String hql = "from EmployeeTask where employee.name = :username";
Query<EmployeeTask> query = session.createQuery(hql, EmployeeTask.class);
query.setParameter("username", "Иван Иванович");
```

### Передача списков (IN clause)

HQL позволяет легко передавать коллекции или массивы:

Java

```
String hql = "from EmployeeTask where occupation IN (:occupation_list)";
query.setParameterList("occupation_list", new String[] {"Программист", "Тестировщик"});
```

---

## 5. Сортировка (Order By)

Сортировка работает так же, как в SQL, но по полям класса:

- `FROM Employee ORDER BY joinDate` (по умолчанию по возрастанию — ASC).
    
- `FROM Employee ORDER BY joinDate DESC, name ASC`.
    

---

## 6. Изменение данных (DML-операции)

Хотя основная мощь Hibernate — это автоматическое сохранение объектов, иногда нужно обновить много записей одним запросом.

- `UPDATE EmployeeTask SET employee = :user WHERE employee IS NULL`
    
- `DELETE FROM EmployeeTask WHERE deadline < :date`
    

**Важно:** Для таких запросов используется метод `executeUpdate()`, и они должны выполняться внутри транзакции.

---

## 7. Именованные запросы (Named Queries)

Если запрос сложный и часто повторяется, его лучше вынести из Java-кода в аннотации к классу. Это делает код чище и позволяет Hibernate заранее проверить синтаксис запроса при старте приложения.

Java

```
@NamedQueries({
    @NamedQuery(
        name = "Employee_FindAll",
        query = "from Employee",
        timeout = 1,        // Таймаут выполнения (в сек)
        fetchSize = 10,     // Размер порции данных
        cacheable = true    // Использовать ли кэш второго уровня
    )
})
@Entity
class Employee { ... }
```

**Вызов в коде:**

Java

```
List<Employee> list = session.createNamedQuery("Employee_FindAll", Employee.class).list();
```

---

## 8. Native SQL (Родной SQL)

Если HQL не поддерживает какую-то специфичную функцию вашей базы (например, хитрые оконные функции PostgreSQL), можно написать обычный SQL-запрос.

Java

```
NativeQuery<Employee> query = session.createNativeQuery("SELECT * FROM employee", Employee.class);
List<Employee> result = query.list();
```

_Минус этого подхода:_ Ваш код перестает быть универсальным и привязывается к конкретной базе данных.

---

## Дополнительно: Агрегатные функции и пагинация

### Агрегаты

HQL поддерживает стандартный набор: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.

- `SELECT COUNT(*) FROM Employee` — вернет общее количество сотрудников.
    

### Пагинация (Постраничный вывод)

Чтобы не грузить из базы сразу миллион записей, используйте методы ограничения:

Java

```
query.setFirstResult(0); // С какой записи начать (отступ)
query.setMaxResults(10); // Сколько записей взять
```

---

### Итог:

HQL позволяет писать запросы на языке доменной модели приложения. Это сокращает количество кода, делает его читаемым и безопасным, а также избавляет от необходимости писать сложные `JOIN` вручную.