## 1. Зачем вообще нужны Generics

### Проблема ДО generics

Без generics коллекции работали с `Object`:

```java
ArrayList list = new ArrayList();
list.add("abc");
list.add(1);

for (Object o : list) {
    String s = (String) o; // ClassCastException
}
```

❌ Ошибка возникает **во время выполнения**  
❌ Нет контроля типов

---

### Решение — Generics

```java
ArrayList<String> list = new ArrayList<>();
list.add("abc");
list.add(1); // ошибка компиляции
```

✔️ Контроль типов на этапе компиляции  
✔️ Нет кастинга  
✔️ Код безопаснее и чище

---

## 2. Что такое Generics

**Generics** — это механизм Java, который позволяет:

- параметризовать классы, методы и интерфейсы **типами**
    
- работать с разными типами **без потери типобезопасности**


Проще:

> «Я заранее говорю, с какими типами будет работать код»

---

## 3. Generic-классы

### Объявление generic-класса

```java
class Box<T> {
    T value;

    T get() {
        return value;
    }

    void set(T value) {
        this.value = value;
    }
}
```

Использование:

```java
Box<String> box = new Box<>();
box.set("hello");
```

📌 `T` — это **параметр типа**, а не конкретный класс.

---

## 4. Почему нельзя создать `new T()`

```java
class Zoo<T> {
    T animal = new T(); // ошибка компиляции
}
```

### Почему это запрещено

- во время выполнения **тип T неизвестен**
    
- generics работают через **type erasure** (стирание типов)


➡️ JVM не знает, какой именно класс создавать.

---

## 5. Ограничения generic-типов (bounded types)

### Ограничение сверху — `extends`

```java
class Zoo<T extends Cat> {
    T cat;
}
```

Теперь `T`:

- либо `Cat`
    
- либо его наследник


📌 Можно вызывать методы `Cat` без кастинга.

---

### Можно ли `extends Object`?

```java
<T extends Object>
```

✔️ Можно  
❌ Но бессмысленно — все классы и так наследуются от `Object`

---

## 6. Wildcards (`?`)

### Что такое wildcard

`?` — это **неизвестный тип**

```java
List<?> list;
```

📌 Мы знаем, что это список, но **не знаем, чего именно**.

---

## 7. `? extends T` (Producer)

```java
List<? extends Animal>
```

Означает:

- список `Animal` **или его наследников**


### Что МОЖНО

```java
Animal a = list.get(0);
```

### Что НЕЛЬЗЯ

```java
list.add(new Animal()); // ошибка
```

📌 Причина: компилятор не знает конкретный подтип.

---

## 8. `? super T` (Consumer)

```java
List<? super Cat>
```

Означает:

- список `Cat` **или его родителей**


### Что МОЖНО

```java
list.add(new Cat());
```

### Что НЕЛЬЗЯ

```java
Cat c = list.get(0); // нельзя, только Object
```

---

## 9. Правило PECS (очень важно)

**PECS** = _Producer Extends, Consumer Super_

- `extends` — когда **читаем**
    
- `super` — когда **записываем**


📌 Это **один из самых частых вопросов**.

---

## 10. Generic-методы

### Что это

Generic-метод — метод со своим параметром типа.

```java
class Utils {
    public static <T> T getFirst(T a) {
        return a;
    }
}
```

📌 `<T>` объявляется **перед типом возвращаемого значения**.

---

## 11. Generics и наследование (важный подвох)

```java
List<Cat> cats = new ArrayList<>();
List<Animal> animals = cats; // ошибка
```

❌ Generics **не поддерживают ковариантность**

Используй wildcard:

```java
List<? extends Animal> animals = cats;
```

---

## 12. Generics и полиморфизм

- `Cat extends Animal`
    
- НО `List<Cat>` **не является** `List<Animal>`


📌 Это ломает интуицию, но **критически важно помнить**.

---

## 13. Type Erasure (стирание типов)

### Что это значит

- generics существуют **только на этапе компиляции**
    
- в JVM все `T` превращаются в `Object` (или ограничение)


Из-за этого:

- нельзя `new T()`
    
- нельзя `instanceof T`
    
- нельзя иметь `static T`


---

## 14. Обход ограничения `new T()`

### Использование `Class<T>`

```java
class Zoo<T> {
    Class<T> clazz;

    Zoo(Class<T> clazz) {
        this.clazz = clazz;
    }

    T create() throws Exception {
        return clazz.newInstance();
    }
}
```

Использование:

```java
Zoo<Tiger> zoo = new Zoo<>(Tiger.class);
Tiger tiger = zoo.create();
```

📌 Тип передаётся **явно**.

---

## 15. Частые ошибки и ловушки

- ❌ `List<Object>` ≠ `List<String>`
    
- ❌ `? extends` — нельзя добавлять
    
- ❌ `? super` — нельзя безопасно читать
    
- ❌ generics ≠ runtime-проверка
    
- ✔️ generics = compile-time безопасность

