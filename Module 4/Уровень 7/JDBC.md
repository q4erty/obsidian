## 1. Введение в JDBC

**JDBC (Java DataBase Connectivity)** — это стандарт, созданный разработчиками Java для унификации работы Java-приложений с различными базами данных.

Поскольку систем управления базами данных (СУБД) существует очень много (MySQL, PostgreSQL, Oracle и др.), каждая из них работает по-своему. JDBC решает эту проблему, предоставляя единый набор интерфейсов (JDBC API). Для взаимодействия с конкретной базой данных используется специальная библиотека — **JDBC Driver**, которая реализует эти интерфейсы под конкретную СУБД.

За выбор правильного драйвера для подключения отвечает специальный класс — `DriverManager`.

## 2. Три главных интерфейса JDBC

Архитектура работы с базой данных строится вокруг трех основных компонентов:

- **`Connection`**: Отвечает за установку и поддержание физического соединения с базой данных.
    
- **`Statement`**: Отвечает за отправку SQL-запросов к базе данных через установленное соединение. _(Дополнение: на практике чаще используют его наследника `PreparedStatement`, так как он защищает от SQL-инъекций и работает быстрее)._
    
- **`ResultSet`**: Отвечает за хранение и обработку результата, полученного от базы данных после выполнения запроса.
    

## 3. Настройка проекта и Подключение

Чтобы Java-приложение могло общаться с MySQL, необходимо добавить соответствующий драйвер в проект. Если вы используете Maven, это делается через зависимость в файле `pom.xml`:

XML

```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.29</version>
</dependency>
```

(Основано на предоставленных источниках )

### Строка подключения (Connection URL)

Для доступа к SQL-серверу используется URL (Universal Resource Locator). Базовый вид строки подключения к локальной базе MySQL выглядит так: `jdbc:mysql://localhost:3306/db_scheme`

- `jdbc:mysql://` — указание протокола и субпротокола.
    
- `localhost` — имя хоста (сервера).
    
- `3306` — стандартный порт MySQL.
    
- `db_scheme` — имя конкретной базы данных.
    

К строке подключения также можно добавлять дополнительные параметры в формате `?имя=значение&имя=значение`.

Пример создания соединения в коде:

Java

```
Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "secret");
```

(Основано на предоставленных источниках )

## 4. Практический пример: Чтение данных из БД

Ниже представлен разбор кода из презентации, демонстрирующий базовый цикл работы с БД.

Java

```
package org.example;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class JdbcApplication {
    public static void main(String[] args) throws Exception {
        // 1. Устанавливаем соединение
        Connection connection = DriverManager.getConnection(
            "jdbc:mysql://localhost:3306/test", "root", "secret"
        );

        // 2. Создаем объект для выполнения запросов
        Statement statement = connection.createStatement();
        
        // 3. Выполняем SQL-запрос и получаем результат
        ResultSet results = statement.executeQuery("SELECT * FROM user");

        // 4. Обрабатываем результат построчно
        while (results.next()) {
            Integer id = results.getInt(1); 
            String name = results.getString(2);
            System.out.println(results.getRow() + ". " + id + "\t" + name);
        }
        
        // 5. Обязательно закрываем соединение
        connection.close(); 
    }
}
```

## 5. Работа с ResultSet и согласование типов данных

Интерфейс `ResultSet` позволяет извлекать данные из колонок. Обращаться к колонкам можно двумя способами:

- По номеру колонки: `getType(номерКолонки)`. _(Важно: нумерация колонок в JDBC начинается с 1, а не с 0)._
    
- По имени колонки: `getType(имяКолонки)`.
    

### Работа с датой и временем (Java Time API)

Современный подход в JDBC позволяет напрямую маппить (преобразовывать) типы данных SQL в классы пакета `java.time`. Для этого используется метод `getObject` с указанием нужного класса:

- `LocalDate localDate = results.getObject(4, LocalDate.class);`
    
- `Instant instant = results.getObject(9, java.time.Instant.class);`
    
- `LocalDateTime local = results.getObject(9, java.time.LocalDateTime.class);`
    
- `OffsetDateTime offset = results.getObject(9, java.time.OffsetDateTime.class);`
    
- `ZonedDateTime zoned = results.getObject(9, java.time.ZonedDateTime.class);`
    

## 6. Дополнение: Работа с ошибками и закрытие ресурсов

В плане вашей лекции был пункт «Работа с ошибками», но в коде использовался просто `throws Exception` и ручное закрытие `connection.close()`.

**Как делать правильно (Best Practice):**

При работе с базой данных могут возникать ошибки сети, неверные пароли или опечатки в SQL-запросах. Все эти ошибки генерируют исключение `SQLException`. Кроме того, ресурсы (`Connection`, `Statement`, `ResultSet`) **обязательно** нужно закрывать, даже если произошла ошибка.

Для этого в современном Java используют блок **try-with-resources**:

Java

```
String url = "jdbc:mysql://localhost:3306/test";
String sql = "SELECT * FROM user";

// Ресурсы, открытые в круглых скобках, закроются автоматически (даже при ошибке)
try (Connection connection = DriverManager.getConnection(url, "root", "secret");
     Statement statement = connection.createStatement();
     ResultSet results = statement.executeQuery(sql)) {

    while (results.next()) {
        System.out.println(results.getString("name"));
    }

} catch (SQLException e) {
    // Обработка ошибки
    System.err.println("Ошибка при работе с базой данных: " + e.getMessage());
}
```

---

Хотите ли вы, чтобы я подробнее разобрал, как использовать `PreparedStatement` вместо обычного `Statement`, чтобы защитить вашу базу данных от взлома?