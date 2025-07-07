## Антипаттерны внедрения зависимостей

- Control freak
- Bastard injection
- Constrained construction
- Service locator
- нарушение принципа единственной ответственности

### Анти-паттерн №1. «Control Freak». Руководитель-наркоман.

Все зависимости контролируются напрямую. Противоположность принципу инверсии
управления.
Антипаттерн возникает всегда, когда мы внутри класса явно создаем изменяемую
зависимость, используя ключевое слово new. Класс, который не отпускает контроль
над своими зависимостями, является Control Freak.

Пример такого класса показан ниже:

```java
private readonly ProductRepository
repository;

public ProductService() {
    string connectionString = ConfigurationManager.ConnectionStrings["Context"].ConnectionString;

    this.repository = new SqlProductRepository(connectionString);
}
```

Вариант избежать данного анти-паттерна: использовать Dependency Injection (
желательно через конструктор). Корректный пример после рефакторинга, не
являющийся Control Freak, выглядит следующим образом:

```java

private readonly ProductRepository
repository;

public ProductService(ProductRepository repository) {
    if (repository == null)
        throw new ArgumentNullException("repository");

    this.repository = repository;
}
```

### Анти-паттерн №2. «Bastard Injection». Внебрачная зависимость.

Данный пункт подразумевает множественные перегрузки конструкторов вместе с
«конструкторами по умолчанию», которые повсеместно встречаются в .Net, включая
BCL. Основная проблема в том, что внешние зависимости обычно определены в других
модулях и подобные конструкторы увеличивают связность системы буквально на
пустом месте. Следует избегать внебрачных внешних зависимостей везде где
возможно, а разрешение зависимостей отдать DI-контейнеру. Пример анти-паттерна
приведен ниже:

```java
private readonly ProductRepository
repository;

public ProductService() :this(ProductService.

CreateDefaultRepository())
        {
        }

public ProductService(ProductRepository repository) {
    if (repository == null)
        throw new ArgumentNullException("repository");

    this.repository = repository;
}

private static ProductRepository CreateDefaultRepository() {
    string connectionString = ConfigurationManager.ConnectionStrings["Context"].ConnectionString;
    return new SqlProductRepository(connectionString);
}

```

Как вариант избежать данного анти-паттерна, предлагается использовать Dependency
Injection. Желательно опять же через конструктор, т.к. внедрение зависимостей
через конструктор является наиболее корректным способом DI.

### Анти-паттерн №3. «Constrained Construction». Ограниченное построение.

Анти-паттерн возникает, если существует требование ко всем зависимостям иметь
«особенный» конструктор. Данное требование зачастую проистекает из желания
«однотипно создавать через Reflection». Что-то типа:

```java
string connectionString = ConfigurationManager.ConnectionStrings["Context"].ConnectionString;
string productRepositoryTypeName = ConfigurationManager.AppSettings["ProductRepositoryType"];
var productRepositoryType = Type.GetType(productRepositoryTypeName, true);
var repository = (ProductRepository) Activator.CreateInstance(productRepositoryType, connectionString);

```

Следует избегать данного анти-паттерна, а для однотипного создания классов
использовать фабрики.

### Анти-паттерн №4. «Service Locator». Сервис-локатор.

Анти-паттерн возникает при гранулированном получении отдельных сервисов в
различных частях кода.
Автор перечисленных DI-анти-паттернов признает, что вопрос сервис-локатора
дискуссионный, однако продолжает считать Service Locator анти-паттерном,
аргументируя в основном тем, что бизнес-логика не должна знать об
инфраструктурных вещах, одной из которых является сервис-локатор и все
зависимости должны пробрасываться явно.

Список анти-паттернов DI приведен из книги “Dependency Injection in .NET”, за
авторством Mark Seemann.

https://habr.com/ru/articles/166287/

## Многоуровневая архитектура

Существуют различные сп особы разделения приложение на слои:

- Model-View-Controller (Модель-Представление-Контроллер) - сокращенно МVС:

    - Model (модель) - данные и методы работы с ними. Модель не знает ничего
      о других слоях, она только изменяет данные по запросам из слоя
      контроллеров и сохраняет их в базу данных. Зачастую именно здесь и
      содержится бизнес-логика;
    - View (представление) - отображение данных пользователю и обработка
      событий пользовательского интерфейса. Представление не обрабатывает данные
      пользователя - этим занимаются другие слои;
    - Controller - связывает два слоя воедино. Преобразует ввод пользователя в
      методы модели или в события представления .

МVС часто используется в веб-программировании, где представление это код НТМL,
CSS и JavaScript, который загружается и используется браузером для формирования
интерфейса пользователя , а контроллер и модель это код на сервере;

- Model-View-Presenter (МVР) - это другой популярный вариант трехуровневой
  архитектуры, производный от Model-View-Controller. Обычно он используется
  при проектировании интерфейсов пользователя. Presenter в нем содержит
  обработчики событий пользовательского интерфейса.

Слои веб приложения:

- слой постоянства;
- слой бизнес-логики;
- слой контроллеров ;
- слой презентации;
- слой безопасности;
- слой пакетных заданий.
