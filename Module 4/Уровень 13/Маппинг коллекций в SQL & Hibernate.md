## 1. Базовый маппинг коллекций (Collection Mapping)

В Java мы часто используем коллекции для хранения наборов данных. Hibernate позволяет сохранять эти коллекции в базу данных. Объекты могут быть сгруппированы в 5 основных типов:

- **Array** (массивы)
    
- **List** (списки объектов)
    
- **Set** (множества объектов)
    
- **Map** (словари объектов)
    
- **Collection** (общие коллекции объектов)
    

### Как это хранится в БД

Реляционные базы данных не умеют хранить списки напрямую в одной ячейке. Поэтому, если класс содержит коллекцию элементов, для ее хранения в базе данных создается **специальная вспомогательная таблица**.

Эта вспомогательная таблица обычно содержит несколько колонок:

- **Foreign Key (Внешний ключ):** Ссылка на ID родительского объекта.
    
- **Column:** Само значение элемента коллекции.
    
- **Index / Key:** Для упорядоченных коллекций (`List`) хранится индекс (позиция), а для словарей (`Map`) — ключ.
    

### Аннотация `@ElementCollection`

Если ваша коллекция содержит базовые типы (например, `String`, `Integer` или встраиваемые объекты `@Embeddable`), а не отдельные сущности (`@Entity`), используется аннотация `@ElementCollection`. Имя вспомогательной таблицы можно задать с помощью `@CollectionTable(name="имя_таблицы")`. Если ее не указать, Hibernate сгенерирует имя автоматически.

**Пример из презентации:**

Java

```
@Entity 
@Table(name="user") 
class User {
    @Id 
    @Column(name="id") 
    public Integer id; [cite: 8, 9]

    @ElementCollection [cite: 10]
    // Можно добавить @CollectionTable(name="user_messages")
    public List<String> messages; [cite: 11]
}
```

_Дополнение:_ В этом случае Hibernate создаст таблицу `User_messages` с колонками `User_id` (ссылка на пользователя) и `messages` (текст сообщения).

---

## 2. Связи между таблицами (Entity Relationships)

Если коллекция содержит не базовые типы, а другие полноценные сущности (Entity), используются аннотации связей.

### 2.1. Связь «Один ко Многим» и «Многие к Одному» (`@OneToMany` / `@ManyToOne`)

Это самая частая связь. Например, один сотрудник может иметь много задач.

**Класс EmployeeTask (сторона "Многие"):** Здесь используется `@ManyToOne` и `@JoinColumn`, чтобы указать, по какой колонке происходит связь (внешний ключ `employee_id`).

Java

```
@Entity 
@Table(name="user") // Прим: вероятно, тут должна быть таблица task, как в других примерах [cite: 24]
class EmployeeTask {
    @Column(name="id") public Integer id; [cite: 25, 26]
    @Column(name="name") public String description; [cite: 27]

    @ManyToOne [cite: 28]
    @JoinColumn(name = "employee_id") 
    public Employee employee; [cite: 29]
}
```

**Класс Employee (сторона "Один"):** Здесь мы указываем коллекцию задач с помощью `@OneToMany`.

Java

```
@Entity 
@Table(name="user") // Прим: обычно таблица employee [cite: 34]
class Employee {
    @Column(name="id") public Integer id; [cite: 35]

    @OneToMany(cascade = CascadeType.ALL) [cite: 36]
    @JoinColumn(name = "employee_id") [cite: 37]
    private Set<EmployeeTask> tasks = new HashSet<EmployeeTask>(); [cite: 38]
}
```

_Дополнение:_ Лучшей практикой в двунаправленных связях является использование атрибута `mappedBy` на стороне `@OneToMany` (например, `@OneToMany(mappedBy = "employee")`), вместо повторного объявления `@JoinColumn`.

**HQL запросы по связям:**

Hibernate позволяет легко делать запросы, не прописывая сложные SQL JOIN вручную:

- Поиск задач по имени сотрудника: `from EmployeeTask where employee.name = "Иван Иванович"`
    
- Выбор уникальных сотрудников, у которых есть просроченные задачи: `select distinct employee from EmployeeTask where deadline < CURDATE();`
    
- Массовое обновление: `update EmployeeTask set employee = :user where employee is null`
    

### 2.2. Связь «Многие ко Многим» (`@ManyToMany`)

Используется, когда несколько сотрудников могут работать над несколькими общими задачами. В этом случае _всегда_ требуется вспомогательная таблица (Join Table), так как ни одна из таблиц не может хранить внешний ключ другой.

В коде это реализуется через `@JoinTable`, где описываются колонки для обеих сторон связи:

Java

```
// В классе Employee
@ManyToMany(cascade = CascadeType.ALL) 
@JoinTable(name="employee_task", [cite: 47]
    joinColumns= @JoinColumn(name="employee_id", referencedColumnName="id"), [cite: 48]
    inverseJoinColumns= @JoinColumn(name="task_id", referencedColumnName="id") ) [cite: 49]
private Set<EmployeeTask> tasks = new HashSet<EmployeeTask>(); [cite: 50]

// В классе EmployeeTask
@ManyToMany(cascade = CascadeType.ALL) 
@JoinTable(name="employee_task", [cite: 55]
    joinColumns= @JoinColumn(name="task_id", referencedColumnName="id"), [cite: 56]
    inverseJoinColumns= @JoinColumn(name=" employee_id", referencedColumnName="id") ) [cite: 57]
private Set<Employee> employees = new HashSet<Employee>(); [cite: 58]
```

### 2.3. Связь «Один к Одному» (`@OneToOne`)

Связь один на один. Например, один сотрудник выполняет строго одну задачу, и эта задача принадлежит только ему. Одна из сторон является главной (содержит внешний ключ), другая — зависимой (использует `mappedBy`).

Java

```
// Класс EmployeeTask (Главная сторона, хранит employee_id)
@OneToOne [cite: 70]
@JoinColumn(name = "employee_id") 
public Employee employee; [cite: 71]

// Класс Employee (Зависимая сторона)
@OneToOne(cascade = CascadeType.ALL, mappedBy="task") [cite: 78]
private EmployeeTask task; [cite: 78]
```

---

## 3. Каскадные изменения (Cascading)

В презентации упоминаются каскадные операции (Каскадные изменения). Это механизм, который указывает Hibernate, нужно ли распространять действия, выполненные над родительской сущностью, на её дочерние сущности.

**Примеры из презентации:**

- `@OneToOne(cascade = CascadeType.ALL)`
    
- `@ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})`
    
- `@Cascade({ org.hibernate.annotations.CascadeType.ALL })` (Аннотация специфичная для Hibernate, не JPA)
    

_Дополнение (расшифровка типов каскадирования):_

- `PERSIST`: Если вы сохраняете родителя, автоматически сохраняются и новые дети.
    
- `MERGE`: Обновление состояния родителя обновит и детей.
    
- `REMOVE`: Удаление родителя повлечет удаление связанных записей в БД.
    
- `ALL`: Включает в себя все вышеперечисленные операции.
    

---

## 4. Удаление «Сирот» (Orphan Removal)

В конце презентации упомянут фрагмент:

`OneToOne(orphan = true)`

_Дополнение и исправление:_ Синтаксически правильно в JPA писать `orphanRemoval = true` (например, `@OneToMany(orphanRemoval = true)`).

**Что это значит?** Это более агрессивная форма удаления, чем `CascadeType.REMOVE`. Если вы удалите дочерний элемент из коллекции родителя (например, `employee.getTasks().remove(task)`), Hibernate автоматически сгенерирует SQL-запрос `DELETE`, чтобы удалить эту задачу из базы данных навсегда, так как она стала "сиротой" (больше не привязана к родителю).

---

Хотели бы вы, чтобы я объяснил подробнее какую-либо из этих концепций, например, разницу между двунаправленными (bidirectional) и однонаправленными (unidirectional) связями?