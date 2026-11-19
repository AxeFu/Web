# Портфолио разработчика на Arch Linux

Практикум для IntelliJ IDEA Ultimate · Java 21 · Spring Boot 4.1.2 · Maven · PostgreSQL 17 · Docker Compose · HTML, CSS и JavaScript · Unity Web

**Как проходить.** [Карта сайта](#map) · [Arch Linux и IDEA](#setup) ·
[PostgreSQL и repository-service](#database) · [Резюме и web-service](#resume) ·
[Демо](#demos) · [Docker Compose](#containers) · [Unity Web](#unity-web) ·
[Проверка результата](#acceptance)

<a id="map"></a>

## 0. Что получится

Сайт открывается с резюме, а каждое умение можно проверить в живом демо. На главной будут имя, специализация, короткий рассказ о себе, опыт или учебные проекты, навыки с доказательствами и рабочие контакты. В примерах личные данные оставлены как поля для заполнения.

| Страница | Что демонстрирует |
| --- | --- |
| `/` | Резюме, проекты, навыки и ссылки на демо |
| `/catalog.html` · `/catalog.html#search-form` | Номера страниц, фильтр по категории и поиск с подсказками |
| `/feed.html` | Бесконечная прокрутка с кнопкой «Загрузить ещё» |
| `/categories.html` | Точное количество товаров по категориям на 50 000 тестовых строк |
| `/board.html` | Kanban-доска. Карточки можно переносить и сортировать, порядок сохраняется |
| `/upload.html` | Перетаскивание файлов из системы и выбор через стандартный диалог |
| `/gallery.html` | Горизонтальная карусель с кнопками и свайпом |
| `/ui.html#modal` · `/ui.html#dropdown` · `/ui.html#animations` | Модальное окно, кастомный селектор, CSS-переходы и анимация |
| `/unity/` | Ваша сборка Unity Web с Brotli. Этот маршрут обслуживает Nginx |

Демо — часть портфолио, а не резюме вместо резюме. Например, подпишите ссылку на каталог так: «Пагинация и SQL-поиск по 50 000 тестовых товаров». Не выдавайте сгенерированные товары за данные реального магазина.

**Архитектура.** В режиме Docker браузер открывает Nginx. Nginx отдаёт Unity из отдельной папки и проксирует остальные запросы в `web-service`. Это Spring Boot-приложение отдаёт HTML/CSS/JS, принимает файлы и предоставляет публичное API. Оно вызывает по HTTP отдельный `repository-service`. Только `repository-service` знает адрес и пароль PostgreSQL, применяет миграции и выполняет SQL. Сервисы собираются в отдельные JAR и запускаются независимо. `repository-service` — отдельный процесс и контейнер, а не интерфейс Java с аннотацией `@Repository` внутри веб-приложения.

| Откуда → куда | Локально при отладке | В Docker Compose |
| --- | --- | --- |
| Браузер → сайт | `localhost:8080` | `localhost:8081` → Nginx |
| `web-service` → `repository-service` | `http://127.0.0.1:8082` | `http://repository:8082` |
| `repository-service` → PostgreSQL | `127.0.0.1:5433` | `db:5432` |
| PostgreSQL → данные | Docker-том `pgdata` | Тот же том |
| `web-service` → принятые файлы | `./uploads` | Docker-том `uploads` |

Это учебное разделение двух сервисов. Здесь нет общей базы для двух приложений, очередей и Kubernetes. Отдельный сервис данных даёт повод освоить сетевые границы, таймауты и диагностику ошибок. В более крупной системе границы обычно выбирают по бизнес-функциям и владельцам данных, а не выносят весь SQL в один универсальный сервис.

**Что вы изучаете.** Maven собирает код в JAR и разрешает зависимости. Spring Framework связывает Java-компоненты и обрабатывает HTTP и транзакции. Spring Boot настраивает сервер и типовые компоненты, чтобы вы могли сосредоточиться на коде. PostgreSQL хранит и считает данные. Docker Compose запускает процессы вместе. HTML описывает элементы страницы, CSS задаёт их вид, JavaScript связывает интерфейс с API.

На каждом этапе сначала запустите и проверьте минимальную версию. Код ниже раскладывается по файлам, указанные пути отсчитываются от корня проекта. Папка `src/main/resources/static` принадлежит только веб-модулю. Маршруты страниц со знаком `.html` оставлены намеренно: Spring отдаёт файлы без контроллеров и роутера JavaScript.

<a id="setup"></a>

## 1. Подготовьте Arch Linux

Обновите систему и поставьте JDK, Docker, Compose, Git и curl. Maven из пакетов не нужен, поскольку будем пользоваться Maven Wrapper из проекта.

~~~bash
sudo pacman -Syu --needed jdk21-openjdk docker docker-compose git curl
sudo systemctl enable --now docker.service

archlinux-java status
java -version
javac -version
docker compose version
sudo docker run --rm hello-world
~~~

Если текущая Java не 21 и вы хотите сделать JDK 21 системным, выполните `sudo archlinux-java set java-21-openjdk`. Это необязательно, если JDK 21 явно выбран в IDEA и настроен в вашем терминале. Системный JDK для IDE обычно находится в `/usr/lib/jvm/java-21-openjdk`.

Чтобы IDEA и ваш обычный пользователь могли обращаться к Docker через системный сокет, на личной учебной машине можно выполнить `sudo usermod -aG docker "$USER"`, полностью выйти из графической сессии, снова войти и проверить `docker info`. Членство в `docker` даёт привилегии уровня root. Если это неприемлемо, используйте rootless Docker либо работайте с Docker через `sudo` в терминале и запускайте Java в IDEA независимо. Не исправляйте права командой `chmod 666 /var/run/docker.sock`.

IDEA установите через JetBrains Toolbox или официальный архив. В актуальной IDEA функции Ultimate доступны по подписке. Проверьте плагины Spring, Docker и Database Tools and SQL. Нужные действия проще найти через **Find Action** (`Ctrl+Shift+A`), если меню отличается.

Подробнее в документации — [Docker в ArchWiki](https://wiki.archlinux.org/title/Docker), [JDK 21 в Arch](https://archlinux.org/packages/extra/x86_64/jdk21-openjdk/), [установка IDEA](https://www.jetbrains.com/help/idea/installation-guide.html), [права Docker](https://docs.docker.com/engine/install/linux-postinstall/), [rootless Docker](https://docs.docker.com/engine/security/rootless/).

## 2. Соберите Maven-проект в IntelliJ IDEA Ultimate

1. В IDEA откройте **File → New → Project → Spring Boot**. Укажите `web-workshop`, Language **Java**, Type **Maven**, Packaging **Jar**, JDK и Java **21**, Group `ru.example`, Package `ru.example.workshop`. Выберите стабильный Spring Boot **4.1.2** и зависимость **Spring Web**. Если в IDE нет мастера, сгенерируйте проект с такими же настройками на [Spring Initializr](https://start.spring.io/), распакуйте и откройте его корневой `pom.xml` как проект.
2. Генератор нужен для получения `mvnw`, `mvnw.cmd` и каталога `.mvn`. Удалите только созданный мастером корневой `src` в этом новом пустом проекте. Вместо него создайте каталоги `repository-service` и `web-service`. В корне и каждом из них создайте POM из примеров ниже.
3. Откройте окно **Maven**, нажмите **Reload All Maven Projects**. В **File → Project Structure → Project SDK** выберите Java 21. В **Settings → Build, Execution, Deployment → Build Tools → Maven** выберите **Use Maven wrapper**. Убедитесь, что IDE видит оба модуля. Если IDEA предлагает загрузить Maven-проект после изменения POM, согласитесь.
4. Для классов используйте **New → Package** внутри соответствующего `src/main/java` и **New → Java Class**. Если каталог не отмечен как исходный, сначала перезагрузите Maven после создания структуры. Пути и `package` должны совпадать.
5. В корне запустите `chmod +x mvnw` и `./mvnw -v`. Проверьте строку Java version. Wrapper скачивает Maven при первом запуске. POM описывает зависимости и сборку, а корневой POM с `packaging=pom` управляет сборкой двух модулей, но сам сервером не является.

<!-- file:pom.xml -->
~~~xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>4.1.2</version>
    <relativePath/>
  </parent>
  <groupId>ru.example</groupId>
  <artifactId>web-workshop</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>pom</packaging>
  <properties><java.version>21</java.version></properties>
  <modules>
    <module>repository-service</module>
    <module>web-service</module>
  </modules>
</project>
~~~

<!-- file:repository-service/pom.xml -->
~~~xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>ru.example</groupId>
    <artifactId>web-workshop</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  </parent>
  <artifactId>repository-service</artifactId>
  <dependencies>
    <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
    <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-jdbc</artifactId></dependency>
    <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-flyway</artifactId></dependency>
    <dependency><groupId>org.flywaydb</groupId><artifactId>flyway-database-postgresql</artifactId></dependency>
    <dependency><groupId>org.postgresql</groupId><artifactId>postgresql</artifactId><scope>runtime</scope></dependency>
  </dependencies>
  <build>
    <finalName>app</finalName>
    <plugins>
      <plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin>
    </plugins>
  </build>
</project>
~~~

<!-- file:web-service/pom.xml -->
~~~xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>ru.example</groupId>
    <artifactId>web-workshop</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  </parent>
  <artifactId>web-service</artifactId>
  <dependencies>
    <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
  </dependencies>
  <build>
    <finalName>app</finalName>
    <plugins>
      <plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin>
    </plugins>
  </build>
</project>
~~~

Подробнее в документации — [первый Spring-проект в IDEA](https://www.jetbrains.com/help/idea/your-first-spring-application.html), [настройки Maven в IDEA](https://www.jetbrains.com/help/idea/maven-support.html), [Maven Wrapper](https://maven.apache.org/wrapper/), [многомодульная сборка](https://maven.apache.org/guides/mini/guide-multiple-modules.html), [системные требования Spring Boot](https://docs.spring.io/spring-boot/4.1/system-requirements.html).

<a id="database"></a>

## 3. Запустите PostgreSQL в Docker

В корне проекта создайте `compose.yaml`. Пока в нём только база. Позже вы добавите сервисы в тот же файл и проверите итоговую конфигурацию.

<!-- file:compose.yaml -->
~~~yaml
services:
  db:
    image: postgres:17
    environment:
      POSTGRES_DB: portfolio
      POSTGRES_USER: portfolio
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-portfolio_dev}
    ports:
      - "127.0.0.1:5433:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U portfolio -d portfolio"]
      interval: 3s
      timeout: 3s
      retries: 20

volumes:
  pgdata:
~~~

~~~bash
docker compose up -d --wait db
docker compose ps
docker compose exec db psql -U portfolio -d portfolio -c 'select current_database();'
~~~

В IDEA откройте **View → Tool Windows → Database → + → Data Source → PostgreSQL**. Host `127.0.0.1`, Port `5433`, Database и User `portfolio`, Password `portfolio_dev`. Если IDEA предложит скачать JDBC-драйвер, нажмите **Download missing driver files**, затем **Test Connection → OK**. В SQL Console выполните `select current_database(), version();`.

Здесь Docker-образ — шаблон PostgreSQL, контейнер — запущенный экземпляр, том — долговременные данные. `5433:5432` означает «порт компьютера → порт контейнера». Из соседнего контейнера обращаются к `db:5432`, а не к `localhost:5433`. Пароль в примере предназначен только для локального стенда. Изменение переменной `POSTGRES_PASSWORD` не меняет пароль уже созданной роли в существующем томе.

Подробнее в документации — [образ PostgreSQL](https://hub.docker.com/_/postgres), [зависимости и готовность в Compose](https://docs.docker.com/compose/how-tos/startup-order/), [PostgreSQL в IDEA](https://www.jetbrains.com/help/idea/postgresql.html), [Docker Compose в IDEA](https://www.jetbrains.com/help/idea/docker-compose.html).

## 4. Сделайте отдельный repository-service

Создайте Spring Boot-точку входа, конфигурацию и миграции. `@SpringBootApplication` запускает приложение и поиск компонентов в своём пакете. `JdbcClient` выполняет параметризованный SQL. Вы увидите запросы и планы PostgreSQL напрямую, без ORM и скрытой генерации запросов.

<!-- file:repository-service/src/main/java/ru/example/workshop/repository/RepositoryApplication.java -->
~~~java
package ru.example.workshop.repository;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class RepositoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(RepositoryApplication.class, args);
    }
}
~~~

<!-- file:repository-service/src/main/resources/application.yaml -->
~~~yaml
server:
  address: ${SERVER_ADDRESS:127.0.0.1}
  port: 8082
spring:
  datasource:
    url: ${DB_URL:jdbc:postgresql://127.0.0.1:5433/portfolio}
    username: ${DB_USER:portfolio}
    password: ${DB_PASSWORD:portfolio_dev}
~~~

В `V1` создайте таблицы и индексы, в `V2` добавьте воспроизводимые демонстрационные данные. Flyway применит новые миграции при запуске. Уже применённые файлы не редактируйте после проверки, новые изменения записывайте в следующую миграцию. Так база любого участника будет приводиться к одной схеме.

<!-- file:repository-service/src/main/resources/db/migration/V1__schema.sql -->
~~~sql
create extension if not exists pg_trgm;

create table category (
    id bigint generated always as identity primary key,
    name text not null unique
);

create table product (
    id bigint generated always as identity primary key,
    name text not null,
    category_id bigint not null references category(id)
);

create index product_category_id_idx on product (category_id, id);
create index product_name_trgm_idx on product using gin (name gin_trgm_ops);

create table task (
    id bigint generated always as identity primary key,
    title text not null,
    lane text not null check (lane in ('todo', 'done')),
    position integer not null check (position >= 0)
);
~~~

<!-- file:repository-service/src/main/resources/db/migration/V2__demo_data.sql -->
~~~sql
insert into category(name) values ('Мониторы'), ('Клавиатуры'), ('Мыши'), ('Аудио'), ('Пока пусто');

insert into product(name, category_id)
select (array['Монитор', 'Клавиатура', 'Мышь', 'Наушники'])[1 + (n - 1) % 4]
       || ' Demo ' || lpad(n::text, 5, '0'),
       1 + (n - 1) % 4
from generate_series(1, 50000) as s(n);

insert into task(title, lane, position) values
    ('Собрать главную страницу', 'todo', 0),
    ('Сделать демо каталога', 'todo', 1),
    ('Настроить Docker', 'done', 0);

analyze product;
~~~

Ниже один контроллер для учебного стенда. Он принимает HTTP-запросы и выполняет небольшой объём SQL. После первого запуска полезно вынести запросы в `@Repository`, а сохранение доски — в `@Service` с `@Transactional`. Оба класса останутся **внутри отдельного repository-service**.

<!-- file:repository-service/src/main/java/ru/example/workshop/repository/DataApi.java -->
~~~java
package ru.example.workshop.repository;

import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import org.springframework.http.HttpStatus;
import org.springframework.jdbc.core.simple.JdbcClient;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.server.ResponseStatusException;

@RestController
@RequestMapping("/internal")
public class DataApi {
    private final JdbcClient db;
    public DataApi(JdbcClient db) { this.db = db; }

    public record Product(long id, String name, long categoryId) {}
    public record Page(List<Product> items, int page, int size, long total) {}
    public record Feed(List<Product> items, Long nextAfter) {}
    public record Category(long id, String name, long count) {}
    public record Task(long id, String title, String lane, int position) {}
    public record Order(List<Long> todo, List<Long> done) {}

    private static final String PRODUCT =
        "select id, name, category_id as \"categoryId\" from product";

    @GetMapping("/products")
    Page products(@RequestParam(name="page", defaultValue="0") int page,
                  @RequestParam(name="size", defaultValue="20") int size,
                  @RequestParam(name="categoryId", required=false) Long categoryId,
                  @RequestParam(name="q", defaultValue="") String q) {
        q = q.strip();
        if (q.length() > 80 || (!q.isEmpty() && q.length() < 3)) {
            throw new ResponseStatusException(HttpStatus.BAD_REQUEST);
        }
        if (page < 0 || size < 1 || size > 100 || (categoryId != null && categoryId < 1)) {
            throw new ResponseStatusException(HttpStatus.BAD_REQUEST);
        }
        var clauses = new ArrayList<String>();
        if (categoryId != null) clauses.add("category_id = :category");
        if (!q.isEmpty()) clauses.add("name ilike :pattern escape '!'");
        String where = clauses.isEmpty() ? "" : " where " + String.join(" and ", clauses);
        var rows = db.sql(PRODUCT + where + " order by id limit :size offset :offset")
            .param("size", size).param("offset", (long) page * size);
        var count = db.sql("select count(*) from product" + where);
        if (categoryId != null) {
            rows.param("category", categoryId);
            count.param("category", categoryId);
        }
        if (!q.isEmpty()) {
            rows.param("pattern", pattern(q));
            count.param("pattern", pattern(q));
        }
        return new Page(rows.query(Product.class).list(), page, size,
            count.query(Long.class).single());
    }

    @GetMapping("/products/{id}")
    Product product(@PathVariable("id") long id) {
        return db.sql(PRODUCT + " where id = :id").param("id", id).query(Product.class)
            .optional().orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
    }

    @GetMapping("/feed")
    Feed feed(@RequestParam(name="after", defaultValue="0") long after) {
        if (after < 0) throw new ResponseStatusException(HttpStatus.BAD_REQUEST);
        int limit = 20;
        var rows = db.sql(PRODUCT + " where id > :after order by id limit :limit")
            .param("after", after).param("limit", limit + 1).query(Product.class).list();
        var items = rows.subList(0, Math.min(limit, rows.size()));
        return new Feed(items, rows.size() > limit ? items.getLast().id() : null);
    }

    @GetMapping("/suggestions")
    List<Product> suggest(@RequestParam("q") String q) {
        q = q.strip();
        if (q.length() < 3) return List.of();
        if (q.length() > 80) throw new ResponseStatusException(HttpStatus.BAD_REQUEST);
        return db.sql(PRODUCT + " where name ilike :pattern escape '!' order by id limit 8")
            .param("pattern", pattern(q)).query(Product.class).list();
    }

    private static String pattern(String value) {
        return "%" + value.replace("!", "!!").replace("%", "!%").replace("_", "!_") + "%";
    }

    @GetMapping("/categories")
    List<Category> categories() {
        return db.sql("""
            select c.id, c.name, count(p.id) as count
            from category c left join product p on p.category_id = c.id
            group by c.id, c.name order by c.id
            """).query(Category.class).list();
    }

    @GetMapping("/tasks")
    List<Task> tasks() {
        return db.sql("select id, title, lane, position from task order by lane, position, id")
            .query(Task.class).list();
    }

    @PutMapping("/tasks/order")
    @Transactional
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void order(@RequestBody Order order) {
        if (order.todo() == null || order.done() == null) {
            throw new ResponseStatusException(HttpStatus.BAD_REQUEST);
        }
        var ids = new HashSet<Long>();
        ids.addAll(order.todo());
        ids.addAll(order.done());
        if (ids.contains(null) || ids.size() != order.todo().size() + order.done().size()) {
            throw new ResponseStatusException(HttpStatus.BAD_REQUEST);
        }
        var stored = db.sql("select id from task order by id for update").query(Long.class).list();
        if (!ids.equals(new HashSet<>(stored))) {
            throw new ResponseStatusException(HttpStatus.CONFLICT);
        }
        saveLane("todo", order.todo());
        saveLane("done", order.done());
    }

    private void saveLane(String lane, List<Long> ids) {
        for (int i = 0; i < ids.size(); i++) {
            db.sql("update task set lane = :lane, position = :pos where id = :id")
                .param("lane", lane).param("pos", i).param("id", ids.get(i)).update();
        }
    }
}
~~~

### Проверьте в IDEA

1. При запущенном `db` откройте `RepositoryApplication.java`. Нажмите зелёную стрелку у `main` → **Run 'RepositoryApplication'**. В **Run → Edit Configurations** переименуйте конфигурацию в `Repository · local`, выставьте JDK 21 и Working directory `$ProjectFileDir$`. Убедитесь, что выбрано classpath модуля `repository-service`. Если пароль/порт отличаются от примера, добавьте `DB_URL` и `DB_PASSWORD` в Environment variables этой конфигурации.
2. В логах должны быть успешные миграции и запуск сервера на `8082`. В окне Database обновите схему `public` и найдите `product` и `flyway_schema_history`.
3. Выполните запросы ниже. После этого поставьте breakpoint внутри `products`, перезапустите через **Debug**, повторите запрос и посмотрите параметры в **Debugger**.

~~~bash
curl -fsS 'http://localhost:8082/internal/products?page=0&size=3'
curl -fsS 'http://localhost:8082/internal/feed?after=20'
curl -fsS 'http://localhost:8082/internal/categories'
curl -fsS -G --data-urlencode 'q=монитор' http://localhost:8082/internal/suggestions
~~~

**Что проверить в SQL Console.** `select count(*) from product;` должен вернуть **50 000**, а запрос категорий — четыре строки по **12 500** и одну с **0**. Именно `count(p.id)`, а не `count(*)` оставляет ноль для категории без товаров при `LEFT JOIN`. Выполните оба плана и сравните фактическое число строк, чтения и время. Не обещайте конкретный план или скорость — PostgreSQL выбирает их по данным и статистике.

~~~sql
explain (analyze, buffers)
select id, name from product order by id limit 20 offset 40000;

explain (analyze, buffers)
select id, name from product where id > 40000 order by id limit 20;

explain (analyze, buffers)
select id from product where name ilike '%монитор%' limit 8;
~~~

`ORDER BY id` задаёт уникальный порядок обоим способам пагинации. Номера страниц удобны для перехода напрямую, но большой `OFFSET` требует пропустить строки. Для ленты используем курсор `after`. Бэкенд запрашивает 21 строку, возвращает 20 и вычисляет наличие следующей порции. Точный `COUNT` выполняется по запросу каталога и не нужен ленте. Если каталог станет большим, измерьте задержку подсчёта до выбора кэша или предварительной агрегации. `pg_trgm` может ускорять `ILIKE` с вхождением в середине строки, но индекс не гарантирует конкретный план.

Подробнее в документации — [Flyway в Spring Boot](https://docs.spring.io/spring-boot/how-to/data-initialization.html), [JdbcClient в Spring Framework](https://docs.spring.io/spring-framework/reference/data-access/jdbc/core.html), [LIMIT и OFFSET](https://www.postgresql.org/docs/17/queries-limit.html), [pg_trgm](https://www.postgresql.org/docs/17/pgtrgm.html), [EXPLAIN](https://www.postgresql.org/docs/17/using-explain.html).


<a id="resume"></a>

## 5. Сделайте web-service и главную страницу резюме

Веб-сервис не добавляет зависимость от модуля `repository-service`, JDBC или драйвера PostgreSQL. Его контракт с соседним сервисом — только HTTP. `RestClient` используем с ограничением времени подключения и чтения. Если сервис данных недоступен, браузер получает `502`, а главная страница резюме продолжает открываться.

<!-- file:web-service/src/main/java/ru/example/workshop/web/WebApplication.java -->
~~~java
package ru.example.workshop.web;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class WebApplication {
    public static void main(String[] args) {
        SpringApplication.run(WebApplication.class, args);
    }
}
~~~

<!-- file:web-service/src/main/resources/application.yaml -->
~~~yaml
server:
  address: ${SERVER_ADDRESS:127.0.0.1}
  port: 8080
spring:
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 11MB
workshop:
  repository-base-url: ${REPOSITORY_BASE_URL:http://127.0.0.1:8082}
  upload-dir: ${UPLOAD_DIR:./uploads}
~~~

<!-- file:web-service/src/main/java/ru/example/workshop/web/PublicApi.java -->
~~~java
package ru.example.workshop.web;

import java.net.URI;
import java.net.http.HttpClient;
import java.time.Duration;
import java.util.function.Function;
import java.util.function.Supplier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.http.client.JdkClientHttpRequestFactory;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.*;
import org.springframework.web.server.ResponseStatusException;
import org.springframework.web.util.UriBuilder;

@RestController
@RequestMapping("/api")
public class PublicApi {
    private final RestClient repository;

    public PublicApi(@Value("${workshop.repository-base-url}") String baseUrl) {
        var http = HttpClient.newBuilder().connectTimeout(Duration.ofSeconds(2)).build();
        var factory = new JdkClientHttpRequestFactory(http);
        factory.setReadTimeout(Duration.ofSeconds(3));
        repository = RestClient.builder().baseUrl(baseUrl).requestFactory(factory).build();
    }

    @GetMapping("/products")
    ResponseEntity<byte[]> products(@RequestParam(name="page", defaultValue="0") int page,
                                    @RequestParam(name="size", defaultValue="20") int size,
                                    @RequestParam(name="categoryId", required=false) Long categoryId,
                                    @RequestParam(name="q", defaultValue="") String q) {
        if (q.length() > 80) throw new ResponseStatusException(HttpStatus.BAD_REQUEST);
        return get(uri -> {
            uri.path("/internal/products").queryParam("page", page).queryParam("size", size);
            if (categoryId != null) uri.queryParam("categoryId", categoryId);
            if (!q.isBlank()) return uri.queryParam("q", "{q}").build(q);
            return uri.build();
        });
    }

    @GetMapping("/products/{id}")
    ResponseEntity<byte[]> product(@PathVariable("id") long id) {
        return get(u -> u.path("/internal/products/{id}").build(id));
    }

    @GetMapping("/feed")
    ResponseEntity<byte[]> feed(@RequestParam(name="after", defaultValue="0") long after) {
        return get(u -> u.path("/internal/feed").queryParam("after", after).build());
    }

    @GetMapping("/suggestions")
    ResponseEntity<byte[]> suggestions(@RequestParam("q") String q) {
        if (q.length() > 80) throw new ResponseStatusException(HttpStatus.BAD_REQUEST);
        return get(u -> u.path("/internal/suggestions").queryParam("q", "{q}").build(q));
    }

    @GetMapping("/categories")
    ResponseEntity<byte[]> categories() { return get(u -> u.path("/internal/categories").build()); }

    @GetMapping("/tasks")
    ResponseEntity<byte[]> tasks() { return get(u -> u.path("/internal/tasks").build()); }

    @PutMapping(path="/tasks/order", consumes=MediaType.APPLICATION_JSON_VALUE)
    ResponseEntity<byte[]> order(@RequestBody byte[] body) {
        return forward(() -> repository.put().uri("/internal/tasks/order")
            .contentType(MediaType.APPLICATION_JSON).body(body).retrieve().toEntity(byte[].class));
    }

    private ResponseEntity<byte[]> get(Function<UriBuilder, URI> uri) {
        return forward(() -> repository.get().uri(uri).retrieve().toEntity(byte[].class));
    }

    private ResponseEntity<byte[]> forward(Supplier<ResponseEntity<byte[]>> call) {
        try {
            var answer = call.get();
            if (answer.getStatusCode().value() == 204) return ResponseEntity.noContent().build();
            return ResponseEntity.status(answer.getStatusCode())
                .contentType(MediaType.APPLICATION_JSON).body(answer.getBody());
        } catch (RestClientResponseException ex) {
            if (ex.getStatusCode().is4xxClientError()) {
                throw new ResponseStatusException(ex.getStatusCode());
            }
            throw new ResponseStatusException(HttpStatus.BAD_GATEWAY, "Ошибка сервиса данных", ex);
        } catch (RestClientException ex) {
            throw new ResponseStatusException(HttpStatus.BAD_GATEWAY, "Сервис данных недоступен", ex);
        }
    }
}
~~~

В `web-service/src/main/resources/static/` создайте `index.html` и `site.css`. Здесь `{Имя Фамилия}` и другие поля — примеры. Замените их своими данными перед публикацией. Раздел «Опыт» можно назвать «Проекты», если коммерческого опыта пока нет. Для каждого проекта укажите задачу, свою роль, использованное решение и то, что посетитель может открыть или проверить. Не добавляйте уровни владения навыками в процентах.

<!-- file:web-service/src/main/resources/static/index.html -->
~~~html
<!doctype html>
<html lang="ru">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Имя Фамилия — портфолио разработчика</title>
  <meta name="description" content="Резюме и демонстрации Java, Spring Boot, SQL и веб-интерфейсов">
  <link rel="stylesheet" href="/site.css">
</head>
<body><main>
  <nav aria-label="Разделы резюме">
    <a href="#about">Обо мне</a>
    <a href="#skills">Навыки и демо</a>
    <a href="#projects">Проекты</a>
    <a href="#contact">Контакты</a>
  </nav>
  <header class="hero">
    <p class="eyebrow">Портфолио разработчика</p>
    <h1>{Имя Фамилия}</h1>
    <p>{Желаемая роль} · {Город или формат работы}</p>
    <p>{В двух предложениях: что умею и какие задачи хочу решать.}</p>
    <a class="cta" href="#skills">Посмотреть демо</a>
  </header>
  <section id="about">
    <h2>Обо мне</h2>
    <p>{Образование, профильный опыт и задачи, которые мне интересны.}</p>
  </section>
  <section id="skills">
    <h2>Навыки и демо</h2>
    <div class="grid">
      <article><h3>Java, Spring, PostgreSQL</h3><p>SQL-поиск, пагинация и точные счётчики.</p>
        <a href="/catalog.html">Каталог</a> · <a href="/catalog.html#search-form">Поиск</a> ·
        <a href="/feed.html">Лента</a> ·
        <a href="/categories.html">Категории</a></article>
      <article><h3>Интерактивный JavaScript</h3><p>Сохранение порядка и отправка файлов.</p>
        <a href="/board.html">Доска</a> · <a href="/upload.html">Загрузка</a></article>
      <article><h3>HTML и CSS</h3><p>Карусель, диалог, селектор, движение.</p>
        <a href="/gallery.html">Галерея</a> · <a href="/ui.html#modal">Модальное окно</a> ·
        <a href="/ui.html#dropdown">Селектор</a> · <a href="/ui.html#animations">Анимации</a></article>
      <article><h3>Unity Web и Docker</h3><p>Сборка Brotli через Nginx.</p>
        <a href="/unity/">Запустить Unity</a></article>
    </div>
  </section>
  <section id="projects">
    <h2>Избранные проекты</h2>
    <article>
      <h3>Этот сайт-портфолио</h3>
      <p>Собрал два Spring Boot-сервиса с Maven, PostgreSQL и Docker Compose.
        Сравнил пагинацию по номеру и курсору на 50 000 сгенерированных товаров.</p>
      <p>{Ссылка на исходники и короткое описание ещё одного проекта.}</p>
    </article>
  </section>
  <section id="contact">
    <h2>Контакты</h2>
    <p>{Рабочая почта} · {GitHub} · {При желании — резюме в PDF}</p>
  </section>
</main></body>
</html>
~~~

<!-- file:web-service/src/main/resources/static/site.css -->
~~~css
:root {
  color-scheme: light;
  font-family: system-ui, sans-serif;
  line-height: 1.55;
  color: #17182a;
  background: #f6f6fb;
  --accent: #4d32ad;
}
* { box-sizing: border-box; }
[hidden] { display: none !important; }
body { margin: 0; }
main { max-width: 64rem; margin: auto; padding: 2rem min(5vw, 3rem) 4rem; }
nav { display: flex; flex-wrap: wrap; gap: .9rem; align-items: center; }
section { margin-block: 2.5rem; }
.hero {
  margin-top: 1.5rem; padding: min(6vw, 3rem);
  background: #eae2ff; border-radius: 1.4rem;
}
h1 { font-size: clamp(2rem, 6vw, 4rem); line-height: 1.05; margin: .3em 0; }
h2, h3 { line-height: 1.25; }
.eyebrow { letter-spacing: .08em; text-transform: uppercase; font-size: .83rem; }
.grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(min(100%, 15rem), 1fr)); gap: 1rem; }
article, .panel { padding: 1rem; background: #fff; border: 1px solid #d8d4e7; border-radius: .9rem; }
article h3 { margin-top: 0; }
a { color: var(--accent); text-underline-offset: .15em; }
button, input { font: inherit; }
button, .cta {
  background: var(--accent); color: #fff; border: 1px solid transparent;
  border-radius: .55rem; padding: .4rem .7rem; cursor: pointer;
  transition: background-color .2s, transform .2s;
}
.cta { display: inline-block; }
button:disabled { opacity: .55; cursor: default; }
button:not(:disabled):hover, .cta:hover { background: #321b82; transform: translateY(-1px); }
:focus-visible { outline: 3px solid #007a71; outline-offset: 3px; }
input[type="search"] { max-width: 100%; padding: .4rem .55rem; }
.row { display: flex; flex-wrap: wrap; align-items: center; gap: .65rem; }
.dropzone { border: 2px dashed #716a86; border-radius: .9rem; padding: 1rem; }
.dropzone.active { background: #eae2ff; border-color: var(--accent); }
.lane { min-height: 5rem; padding: .8rem 1rem .8rem 2.2rem; margin: .5rem 0; border: 1px dashed #716a86; }
.task { margin-bottom: .65rem; padding: .5rem; background: #f4f0ff; }
.task .row { margin-top: .4rem; }
.track {
  position: relative; display: flex; overflow-x: auto; scroll-snap-type: x mandatory;
  border-radius: .9rem; background: #fff;
}
.track figure { flex: 0 0 100%; margin: 0; scroll-snap-align: start; }
.track img { width: 100%; aspect-ratio: 16 / 9; object-fit: cover; display: block; }
.track figcaption { padding: .7rem; }
dialog { max-width: min(30rem, 90vw); border: 0; border-radius: .9rem; padding: 1.4rem; }
dialog::backdrop { background: rgb(17 15 30 / 55%); }
.demo-menu { position: relative; width: fit-content; }
.demo-menu summary { cursor: pointer; }
.demo-menu button[aria-pressed="true"] { background: #23655d; }
.pulse { animation: pop .45s ease-out; }
@keyframes pop { 50% { transform: scale(1.12); background: #23655d; } }
@media (prefers-reduced-motion: reduce) {
  button, .cta { transition: none; }
  button:not(:disabled):hover, .cta:hover { transform: none; }
  .pulse { animation: none; }
}
~~~

В IDEA запустите `WebApplication` из gutter, как раньше `RepositoryApplication`. В **Run → Edit Configurations** назовите конфигурацию `Web · local` и выставьте Working directory `$ProjectFileDir$`, чтобы загруженные файлы попадали в корень проекта. Откройте `http://localhost:8080/`. Без IDE эти два приложения можно запустить из двух терминалов в корне проекта командами `./mvnw -pl repository-service spring-boot:run` и `./mvnw -pl web-service spring-boot:run`. В **Services** можно следить за Docker, в **Run** — за отдельными логами Java-сервисов. На Linux комбинации клавиш могут перехватываться оконным менеджером, поэтому главное — названия действий.

~~~bash
curl -fsS 'http://localhost:8080/api/products?page=0&size=3'
curl -fsS -G --data-urlencode 'q=монитор' http://localhost:8080/api/suggestions
~~~

Остановите только `Repository · local` и повторите запрос `curl -i http://localhost:8080/api/categories`. Ожидается `502`, при этом `/` остаётся доступным. Затем снова запустите сервис данных. Это хороший момент поставить два breakpoint: в `PublicApi.products` и `DataApi.products`. Один запрос из браузера остановится в двух разных JVM. Возобновите выполнение первого процесса, чтобы запрос дошёл до второго. Из-за таймаута в 3 секунды медленный пошаговый debug может вернуть `502`. Для длительного эксперимента временно увеличьте таймаут и верните его после отладки.

Подробнее в документации — [клиенты REST в Spring Framework](https://docs.spring.io/spring-framework/reference/integration/rest-clients.html), [Spring MVC](https://docs.spring.io/spring-framework/reference/web/webmvc.html), [статические ресурсы Spring Boot](https://docs.spring.io/spring-boot/reference/web/servlet.html), [отладка в IntelliJ IDEA](https://www.jetbrains.com/help/idea/debugging-code.html).

<a id="demos"></a>

## 6. Сделайте каталог, поиск, прокрутку и счётчики

Внутри `static` создайте четыре HTML-страницы и два JS-файла. Общий модуль `common.js` проверяет HTTP-статус. При отображении текста из API используйте `textContent`, а не присваивание `innerHTML`.

<!-- file:web-service/src/main/resources/static/common.js -->
~~~js
export async function json(url, options) {
  const response = await fetch(url, options);
  if (!response.ok) throw new Error("HTTP " + response.status);
  return response.status === 204 ? null : response.json();
}

export function addText(list, text) {
  const li = document.createElement("li");
  li.textContent = text;
  list.append(li);
}
~~~

<!-- file:web-service/src/main/resources/static/catalog.html -->
~~~html
<!doctype html>
<html lang="ru"><head>
<meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
<title>Каталог — портфолио</title><link rel="stylesheet" href="/site.css">
</head><body><main>
<nav><a href="/">← Резюме</a><a href="/categories.html">Категории</a></nav>
<h1>Каталог</h1>
<form id="search-form" class="row">
  <label for="search">Название товара</label>
  <input id="search" type="search" maxlength="80" autocomplete="off"
         placeholder="Например, монитор" aria-describedby="search-status">
  <button>Найти</button>
</form>
<p id="search-status" role="status">Введите не менее трёх символов.</p>
<ul id="suggestions"></ul>
<div class="row">
  <button id="prev" disabled>Назад</button>
  <button id="next" disabled>Вперёд</button>
  <form id="jump">
    <label for="page">Страница</label>
    <input id="page" type="number" min="1" required value="1">
    <button>Перейти</button>
  </form>
</div>
<p id="status" role="status"></p><ol id="items"></ol>
<script type="module" src="/browse.js"></script>
</main></body></html>
~~~

<!-- file:web-service/src/main/resources/static/feed.html -->
~~~html
<!doctype html>
<html lang="ru"><head>
<meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
<title>Лента — портфолио</title><link rel="stylesheet" href="/site.css">
</head><body><main>
<nav><a href="/">← Резюме</a><a href="/catalog.html">Каталог</a></nav>
<h1>Бесконечная лента</h1>
<p>Прокрутите вниз или нажмите кнопку. Сервер отдаёт по 20 записей.</p>
<ol id="feed-items"></ol>
<p id="feed-status" role="status"></p><button id="more">Загрузить ещё</button>
<div id="sentinel" aria-hidden="true" style="height: 1px"></div>
<script type="module" src="/browse.js"></script>
</main></body></html>
~~~

<!-- file:web-service/src/main/resources/static/categories.html -->
~~~html
<!doctype html>
<html lang="ru"><head>
<meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
<title>Категории — портфолио</title><link rel="stylesheet" href="/site.css">
</head><body><main>
<nav><a href="/">← Резюме</a><a href="/catalog.html">Каталог</a></nav>
<h1>Количество товаров</h1>
<p>PostgreSQL считает товары по категориям. В каталоге 50 000 учебных записей.</p>
<ul id="categories"></ul><p id="category-status" role="status"></p>
<script type="module" src="/browse.js"></script>
</main></body></html>
~~~

<!-- file:web-service/src/main/resources/static/product.html -->
~~~html
<!doctype html>
<html lang="ru"><head>
<meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
<title>Товар — портфолио</title><link rel="stylesheet" href="/site.css">
</head><body><main>
<nav><a href="/">← Резюме</a><a href="/catalog.html">Каталог</a></nav>
<h1 id="product-name">Учебный товар</h1><p id="product-status" role="status">Загрузка…</p>
<script type="module" src="/browse.js"></script>
</main></body></html>
~~~

<!-- file:web-service/src/main/resources/static/browse.js -->
~~~js
import { json, addText } from "./common.js";
const $ = (id) => document.getElementById(id);
const pretty = new Intl.NumberFormat("ru");

if ($("items")) {
  const params = new URLSearchParams(location.search);
  let page = 0;
  let pages = 0;
  let loading = false;
  const start = Math.max(0, Number(params.get("page") || 1) - 1);
  const prev = $("prev"), next = $("next"), input = $("page"), status = $("status");

  async function loadPage(requested) {
    if (loading || !Number.isSafeInteger(requested) || requested < 0 || requested > 2147483647) return;
    loading = prev.disabled = next.disabled = true;
    status.textContent = "Загрузка…";
    try {
      const q = new URLSearchParams({ page: requested, size: 20 });
      if (params.has("categoryId")) q.set("categoryId", params.get("categoryId"));
      if (params.has("q")) q.set("q", params.get("q"));
      const data = await json("/api/products?" + q);
      page = data.page;
      pages = Math.ceil(data.total / data.size);
      if (page >= Math.max(1, pages)) {
        loading = false;
        return await loadPage(Math.max(0, pages - 1));
      }
      const list = $("items");
      list.replaceChildren();
      list.start = page * data.size + 1;
      for (const item of data.items) addText(list, item.name + " · #" + item.id);
      input.value = page + 1;
      input.max = Math.max(1, pages);
      params.set("page", page + 1);
      history.replaceState(null, "", "?" + params);
      status.textContent = pages ? "Страница " + (page + 1) + " из " + pages +
        " · товаров " + pretty.format(data.total) : "По этим условиям товаров нет.";
    } catch (error) {
      status.textContent = "Не удалось загрузить страницу. Повторите запрос. " + error.message;
    } finally {
      loading = false;
      prev.disabled = page <= 0;
      next.disabled = page + 1 >= pages;
    }
  }

  prev.onclick = () => void loadPage(page - 1);
  next.onclick = () => void loadPage(page + 1);
  $("jump").onsubmit = (e) => { e.preventDefault(); void loadPage(Number(input.value) - 1); };
  void loadPage(Number.isSafeInteger(start) && start <= 2147483647 ? start : 0);

  const search = $("search"), suggestions = $("suggestions"), searchStatus = $("search-status");
  let timer, pending;
  search.value = params.get("q") ?? "";
  $("search-form").onsubmit = (e) => {
    e.preventDefault();
    if (loading) return;
    const value = search.value.trim();
    if (value.length > 0 && value.length < 3) {
      searchStatus.textContent = "Введите не менее трёх символов.";
      return;
    }
    clearTimeout(timer);
    pending?.abort();
    suggestions.replaceChildren();
    searchStatus.textContent = value ? "Поиск: " + value : "Фильтр по названию сброшен.";
    if (value) params.set("q", value);
    else params.delete("q");
    void loadPage(0);
  };
  search.addEventListener("input", () => {
    clearTimeout(timer);
    pending?.abort();
    suggestions.replaceChildren();
    const value = search.value.trim();
    if (value.length < 3) {
      searchStatus.textContent = "Введите не менее трёх символов.";
      return;
    }
    searchStatus.textContent = "Ищем…";
    timer = setTimeout(async () => {
      const request = new AbortController();
      pending = request;
      try {
        const rows = await json("/api/suggestions?" + new URLSearchParams({ q: value }),
          { signal: request.signal });
        if (request.signal.aborted) return;
        for (const item of rows) {
          const li = document.createElement("li"), link = document.createElement("a");
          link.textContent = item.name + " · #" + item.id;
          link.href = "/product.html?id=" + item.id;
          li.append(link);
          suggestions.append(li);
        }
        searchStatus.textContent = rows.length ? "Подсказок: " + rows.length : "Совпадений нет.";
      } catch (error) {
        if (error.name !== "AbortError") searchStatus.textContent = "Ошибка поиска. " + error.message;
      }
    }, 250);
  });
}

if ($("product-name")) {
  const id = new URLSearchParams(location.search).get("id");
  const status = $("product-status");
  if (!id || !/^\d+$/.test(id)) {
    status.textContent = "Некорректный ID товара.";
  } else {
    json("/api/products/" + id).then((item) => {
      $("product-name").textContent = item.name;
      status.textContent = "Номер в учебном каталоге: " + item.id;
    }).catch((error) => { status.textContent = "Не удалось открыть товар. " + error.message; });
  }
}

if ($("feed-items")) {
  const target = $("sentinel"), button = $("more"), status = $("feed-status");
  let after = 0, busy = false, ended = false;

  async function loadMore() {
    if (busy || ended) return;
    busy = button.disabled = true;
    observer.unobserve(target);
    let succeeded = false;
    status.textContent = "Загрузка…";
    try {
      const data = await json("/api/feed?after=" + after);
      for (const item of data.items) addText($("feed-items"), item.name + " · #" + item.id);
      ended = data.nextAfter === null;
      after = data.nextAfter;
      succeeded = true;
      status.textContent = ended ? "Все товары загружены." : "Добавлено: " + data.items.length;
    } catch (error) {
      status.textContent = "Не загрузилось. Нажмите «Загрузить ещё». " + error.message;
    } finally {
      busy = false;
      button.disabled = ended;
      if (succeeded && !ended) observer.observe(target);
    }
  }

  const observer = new IntersectionObserver((entries) => {
    if (entries.some((entry) => entry.isIntersecting)) void loadMore();
  }, { rootMargin: "0px 0px 120px 0px" });
  button.onclick = () => void loadMore();
  void loadMore();
}

if ($("categories")) {
  json("/api/categories").then((rows) => {
    for (const row of rows) {
      const li = document.createElement("li"), link = document.createElement("a");
      link.href = "/catalog.html?categoryId=" + row.id;
      link.textContent = row.name + " — " + pretty.format(row.count);
      li.append(link);
      $("categories").append(li);
    }
  }).catch((error) => { $("category-status").textContent = "Не удалось получить счётчики. " + error.message; });
}
~~~

**Как это работает.** Каталог использует номера страниц с единицы в интерфейсе и с нуля в API. Прямой переход принимает номер, а URL сохраняет выбранную страницу. Подсказки появляются после 250 мс без ввода. `AbortController` отменяет устаревший запрос со стороны браузера, чтобы его ответ не затёр новые результаты. Подсказки — обычные ссылки, доступны по Tab и ведут на страницу конкретного товара по ID. Enter или кнопка «Найти» применяют поиск ко всему каталогу, включая точное число результатов. Пустой поиск снимает фильтр.

В ленте `IntersectionObserver` наблюдает за элементом внизу списка. Один запрос за раз исключает повторы, а после сетевой ошибки остаётся ручная кнопка. При тысячах элементов, одновременно оставленных в DOM, страница может замедлиться. Для следующей версии измерьте это в Performance и при необходимости добавьте виртуализацию.

**Проверка.** Откройте `/catalog.html?page=2`. Перейдите на последнюю страницу, затем попробуйте пустую категорию. В DevTools → Network включите медленную сеть, быстро введите «монитор», затем «клавиатура» и проверьте, что отображается только последний ответ. В ленте сравните последние ID порций: они должны расти. В `/categories.html` проверьте пять категорий и сумму 50 000.

Подробнее в документации — [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch), [AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController), [Intersection Observer](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API).


## 7. Реализуйте перетаскивание карточек и сохранение порядка

В `/board.html` будут колонки «Сделать» и «Готово». Для мыши используем браузерные `dragstart`, `dragover` и `drop`. Для клавиатуры и телефона оставляем видимые кнопки «Выше», «Ниже» и «В другую колонку». Нативный HTML Drag and Drop не стоит считать надёжным способом взаимодействия на всех сенсорных устройствах.

Схема сохранения проста: браузер отправляет две упорядоченные последовательности ID, сервис данных проверяет, что каждый ID встречается ровно один раз и состав карточек совпадает с базой, затем меняет позиции в транзакции. При ошибке браузер возвращает прежний порядок. Одновременные правки двух вкладок одной доски по-прежнему могут перезаписать друг друга. Если захотите полноценно поддержать совместную работу, добавьте версию доски и отклоняйте изменения устаревшей версии с `409`.

<!-- file:web-service/src/main/resources/static/board.html -->
~~~html
<!doctype html>
<html lang="ru"><head>
<meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
<title>Доска — портфолио</title><link rel="stylesheet" href="/site.css">
</head><body><main>
<a href="/">← Резюме</a><h1>Доска задач</h1>
<p>Перетащите карточку на другую карточку, чтобы вставить перед ней,
  или на свободное место колонки, чтобы добавить в конец.</p>
<p id="board-status" role="status"></p>
<div id="board" class="grid">
  <article><h2>Сделать</h2><ol class="lane" data-lane="todo"></ol></article>
  <article><h2>Готово</h2><ol class="lane" data-lane="done"></ol></article>
</div>
<script type="module" src="/board.js"></script>
</main></body></html>
~~~

<!-- file:web-service/src/main/resources/static/board.js -->
~~~js
import { json } from "./common.js";
const board = document.querySelector("#board"), status = document.querySelector("#board-status");
const lanes = ["todo", "done"];
let tasks = { todo: [], done: [] };
let saving = false;

function paint(focusId, focusAction) {
  for (const lane of lanes) {
    const list = board.querySelector('[data-lane="' + lane + '"]');
    list.replaceChildren();
    for (const [i, task] of tasks[lane].entries()) {
      const card = document.createElement("li"), controls = document.createElement("div");
      card.className = "task";
      card.dataset.id = task.id;
      card.draggable = !saving;
      card.tabIndex = -1;
      card.append(document.createTextNode(task.title));
      controls.className = "row";
      const actions = [
        ["up", "Выше", i === 0, () => move(task.id, lane, tasks[lane][i - 1].id, "up")],
        ["down", "Ниже", i === tasks[lane].length - 1, () => move(task.id, lane, tasks[lane][i + 2]?.id, "down")],
        ["other", "В другую колонку", false, () => move(task.id, lane === "todo" ? "done" : "todo", undefined, "other")]
      ];
      let focusButton;
      for (const [key, title, disabled, action] of actions) {
        const button = document.createElement("button");
        button.textContent = title;
        button.setAttribute("aria-label", task.title + ": " + title);
        button.disabled = saving || disabled;
        button.onclick = action;
        if (key === focusAction && !button.disabled) focusButton = button;
        controls.append(button);
      }
      card.append(controls);
      list.append(card);
      if (task.id === focusId) (focusButton ?? card).focus({ preventScroll: true });
    }
  }
}

async function move(id, lane, before, action) {
  if (saving || id === before || !lanes.includes(lane)) return;
  const from = lanes.find((key) => tasks[key].some((card) => card.id === id));
  if (!from) return;
  const original = structuredClone(tasks);
  const index = tasks[from].findIndex((card) => card.id === id);
  const [card] = tasks[from].splice(index, 1);
  const found = tasks[lane].findIndex((item) => item.id === before);
  tasks[lane].splice(found < 0 ? tasks[lane].length : found, 0, card);
  saving = true;
  paint(id);
  try {
    await json("/api/tasks/order", {
      method: "PUT", headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ todo: tasks.todo.map(t => t.id), done: tasks.done.map(t => t.id) })
    });
    status.textContent = "Порядок сохранён.";
  } catch (error) {
    tasks = original;
    status.textContent = "Не удалось сохранить. Порядок на экране восстановлен. " + error.message;
  } finally {
    saving = false;
    paint(id, action);
  }
}

board.addEventListener("dragstart", (e) => {
  const card = e.target.closest("[data-id]");
  if (!card || saving) return;
  e.dataTransfer.setData("text/plain", card.dataset.id);
  e.dataTransfer.effectAllowed = "move";
});
board.addEventListener("dragover", (e) => {
  if (!saving && e.target.closest("[data-lane]")) e.preventDefault();
});
board.addEventListener("drop", (e) => {
  const list = e.target.closest("[data-lane]");
  if (!list) return;
  e.preventDefault();
  const id = Number(e.dataTransfer.getData("text/plain"));
  const target = e.target.closest("[data-id]");
  void move(id, list.dataset.lane, target ? Number(target.dataset.id) : undefined);
});

json("/api/tasks").then((rows) => {
  for (const row of rows) tasks[row.lane].push(row);
  paint();
}).catch((error) => { status.textContent = "Доска не загрузилась. " + error.message; });
~~~

**Проверка.** Перенесите карточку перед другой, в другую колонку и в пустую колонку. Обновите страницу. Убедитесь, что порядок сохранён. Повторите действие с клавиатуры. В Database Console выполните `select * from task order by lane, position;`. Остановите сервис данных и попытайтесь переставить карточку: интерфейс должен показать ошибку и вернуть предыдущий порядок.

Подробнее в документации — [HTML Drag and Drop API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API), [декларативные транзакции Spring](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative.html).

## 8. Добавьте загрузку файлов

В отличие от доски, здесь перетаскиваются файлы из операционной системы. JS берёт их из `DataTransfer.files` при `drop` или из `input.files` при выборе. Каждый файл отправляется отдельно в `multipart/form-data` с полем `file`. Браузер сам задаёт `Content-Type` с границей частей, вручную этот заголовок не ставьте.

Spring ограничивает размер 10 МиБ на файл. Сервер не использует исходное имя для записи, сохраняет файл под UUID вне `static` и не раздаёт его по URL. Готовый файл перемещается из временного после копирования. Это демо приёма файлов, а не публичное файловое хранилище.

<!-- file:web-service/src/main/java/ru/example/workshop/web/UploadApi.java -->
~~~java
package ru.example.workshop.web;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.StandardCopyOption;
import java.util.UUID;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.server.ResponseStatusException;

@RestController
public class UploadApi {
    private final Path root;
    public UploadApi(@Value("${workshop.upload-dir}") String directory) {
        root = Path.of(directory).toAbsolutePath().normalize();
    }

    public record Uploaded(String id, long size) {}

    @PostMapping(path="/api/files", consumes=MediaType.MULTIPART_FORM_DATA_VALUE)
    @ResponseStatus(HttpStatus.CREATED)
    Uploaded upload(@RequestParam("file") MultipartFile file) throws IOException {
        if (file.isEmpty()) throw new ResponseStatusException(HttpStatus.BAD_REQUEST);
        Files.createDirectories(root);
        String id = UUID.randomUUID().toString();
        Path temp = Files.createTempFile(root, ".upload-", ".part");
        try {
            try (var input = file.getInputStream()) {
                Files.copy(input, temp, StandardCopyOption.REPLACE_EXISTING);
            }
            Files.move(temp, root.resolve(id), StandardCopyOption.ATOMIC_MOVE);
        } finally {
            Files.deleteIfExists(temp);
        }
        return new Uploaded(id, file.getSize());
    }
}
~~~

<!-- file:web-service/src/main/resources/static/upload.html -->
~~~html
<!doctype html>
<html lang="ru"><head>
<meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
<title>Загрузка — портфолио</title><link rel="stylesheet" href="/site.css">
</head><body><main>
<a href="/">← Резюме</a><h1>Загрузка файлов</h1>
<div id="drop" class="dropzone">
  <p>Перетащите файлы сюда. Максимум 10 МиБ на файл.</p>
  <label for="files">Или выберите файлы:</label> <input id="files" type="file" multiple>
</div>
<p id="upload-status" role="status"></p><ul id="results"></ul>
<script type="module" src="/upload.js"></script>
</main></body></html>
~~~

<!-- file:web-service/src/main/resources/static/upload.js -->
~~~js
import { json, addText } from "./common.js";
const zone = document.querySelector("#drop"), input = document.querySelector("#files");
const status = document.querySelector("#upload-status"), results = document.querySelector("#results");
let busy = false;

async function upload(files) {
  if (busy || !files.length) return;
  busy = input.disabled = true;
  try {
    for (const file of files) {
      status.textContent = "Обрабатываем: " + file.name;
      if (file.size === 0 || file.size > 10 * 1024 * 1024) {
        addText(results, file.name + " — недопустимый размер");
        continue;
      }
      const body = new FormData();
      body.append("file", file);
      try {
        const result = await json("/api/files", { method: "POST", body });
        addText(results, file.name + " — принят, ID " + result.id);
      } catch (error) {
        addText(results, file.name + " — ошибка: " + error.message);
      }
    }
    status.textContent = "Готово. Результаты ниже.";
  } finally {
    busy = input.disabled = false;
    input.value = "";
  }
}

zone.addEventListener("dragover", (e) => {
  if (!e.dataTransfer.types.includes("Files")) return;
  e.preventDefault();
  e.dataTransfer.dropEffect = busy ? "none" : "copy";
  if (!busy) zone.classList.add("active");
});
zone.addEventListener("dragleave", (e) => {
  if (!zone.contains(e.relatedTarget)) zone.classList.remove("active");
});
zone.addEventListener("drop", (e) => {
  e.preventDefault();
  zone.classList.remove("active");
  void upload([...e.dataTransfer.files]);
});
input.addEventListener("change", () => void upload([...input.files]));
~~~

**Проверка.** Перетащите небольшой тестовый файл, затем отправьте его через выбор файлов. Посмотрите, что в `./uploads` появились две записи под разными UUID. Этот же запрос можно воспроизвести из терминала:

~~~bash
printf 'test upload\n' > /tmp/portfolio-test.txt
curl -i -F 'file=@/tmp/portfolio-test.txt' http://localhost:8080/api/files
~~~

Отдельно проверьте файл больше лимита через `curl`, обходя проверку браузера. Для теста удобно взять файл 10,5 МиБ: он проходит транспортный лимит Nginx, но должен получить `413` от Spring. Исключите `/uploads/` из Git. Если позже захотите разрешить скачивание или предпросмотр, сначала определите допустимые форматы и проверку содержимого. Не выкладывайте присланные посетителями файлы прямо в `static`.

Подробнее в документации — [File drag and drop](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API/File_drag_and_drop), [FormData](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest_API/Using_FormData_Objects), [гайд Spring по загрузке файлов](https://spring.io/guides/gs/uploading-files).

## 9. Сделайте галерею и страницу компонентов

Для галереи поместите три собственных изображения `01.jpg`, `02.jpg` и `03.jpg` в `web-service/src/main/resources/static/photos/`. Измените подписи и `alt` так, чтобы они описывали именно ваши фотографии или скриншоты. CSS Scroll Snap ставит слайд на границу после прокрутки. Кнопки управляют тем же скроллом, поэтому работают и с клавиатуры. Не запускайте автоматическую смену слайдов.

<!-- file:web-service/src/main/resources/static/gallery.html -->
~~~html
<!doctype html>
<html lang="ru"><head>
<meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
<title>Галерея — портфолио</title><link rel="stylesheet" href="/site.css">
</head><body><main>
<a href="/">← Резюме</a><h1>Галерея проектов</h1>
<div class="row">
  <button id="previous">Предыдущий</button><button id="following">Следующий</button>
</div>
<div id="track" class="track" role="region" tabindex="0" aria-label="Фотографии проектов">
  <figure><img src="/photos/01.jpg" alt="Замените на описание первого кадра"><figcaption>Кадр 1</figcaption></figure>
  <figure><img src="/photos/02.jpg" alt="Замените на описание второго кадра"><figcaption>Кадр 2</figcaption></figure>
  <figure><img src="/photos/03.jpg" alt="Замените на описание третьего кадра"><figcaption>Кадр 3</figcaption></figure>
</div>
<script src="/gallery.js" defer></script>
</main></body></html>
~~~

<!-- file:web-service/src/main/resources/static/gallery.js -->
~~~js
const track = document.querySelector("#track");
const slides = [...track.querySelectorAll("figure")];
const back = document.querySelector("#previous"), next = document.querySelector("#following");
const reduce = matchMedia("(prefers-reduced-motion: reduce)");
let current = 0;

function setControls() {
  back.disabled = current === 0;
  next.disabled = current === slides.length - 1;
}
function show(delta) {
  current = Math.max(0, Math.min(slides.length - 1, current + delta));
  track.scrollTo({ left: slides[current].offsetLeft, behavior: reduce.matches ? "instant" : "smooth" });
  setControls();
}
back.onclick = () => show(-1);
next.onclick = () => show(1);
track.addEventListener("scroll", () => {
  if (track.clientWidth) current = Math.min(slides.length - 1, Math.round(track.scrollLeft / track.clientWidth));
  setControls();
}, { passive: true });
setControls();
~~~

Для модального окна используйте `<dialog>` и `showModal()`. Браузер сам блокирует взаимодействие с фоном, закрывает окно по Escape и управляет начальным фокусом. Кастомный селектор здесь — раскрывающийся блок `<details>` со списком кнопок, выбранным состоянием и возвратом фокуса. Это не имитация WAI-ARIA combobox. Если нужен селектор внутри формы, начните с `<select>` или реализуйте и протестируйте полный паттерн APG, включая навигацию стрелками и управление фокусом.

<!-- file:web-service/src/main/resources/static/ui.html -->
~~~html
<!doctype html>
<html lang="ru"><head>
<meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
<title>Компоненты — портфолио</title><link rel="stylesheet" href="/site.css">
</head><body><main>
<a href="/">← Резюме</a><h1>HTML и CSS</h1>
<section id="modal" class="panel">
  <h2>Модальное окно</h2>
  <button id="open-dialog">Открыть</button>
  <dialog id="dialog" aria-labelledby="dialog-title">
    <h2 id="dialog-title">Пример проекта</h2><p>Здесь можно показать детали работы.</p>
    <form method="dialog"><button autofocus>Закрыть</button></form>
  </dialog>
</section>
<section id="dropdown" class="panel">
  <h2>Кастомный селектор</h2>
  <details id="selector" class="demo-menu">
    <summary id="selection">Тема: Светлая</summary>
    <div class="row">
      <button type="button" aria-pressed="true">Светлая</button>
      <button type="button" aria-pressed="false">Сиреневая</button>
      <button type="button" aria-pressed="false">Мятная</button>
    </div>
  </details>
</section>
<section id="animations" class="panel">
  <h2>CSS-анимации</h2><p>Наведите мышь для плавной смены цвета, нажмите для анимации.</p>
  <button id="motion">Запустить анимацию</button>
</section>
<script src="/ui.js" defer></script>
</main></body></html>
~~~

<!-- file:web-service/src/main/resources/static/ui.js -->
~~~js
const dialog = document.querySelector("#dialog"), opener = document.querySelector("#open-dialog");
opener.onclick = () => dialog.showModal();
dialog.addEventListener("close", () => opener.focus());

const details = document.querySelector("#selector"), summary = document.querySelector("#selection");
const colors = ["#4d32ad", "#653281", "#23655d"];
const options = [...details.querySelectorAll("button")];
options.forEach((button, index) => {
  button.onclick = () => {
    for (const item of options) item.setAttribute("aria-pressed", String(item === button));
    summary.textContent = "Тема: " + button.textContent;
    document.documentElement.style.setProperty("--accent", colors[index]);
    details.open = false;
    summary.focus();
  };
});
details.addEventListener("keydown", (e) => {
  if (e.key === "Escape" && details.open) {
    e.preventDefault();
    details.open = false;
    summary.focus();
  }
});

const motion = document.querySelector("#motion");
motion.onclick = () => {
  motion.classList.remove("pulse");
  void motion.offsetWidth; // Применить сброс перед повторным запуском анимации.
  motion.classList.add("pulse");
};
~~~

**Проверка.** Пройдите страницу клавишами Tab, Enter, Space и Escape. Убедитесь, что Tab не попадает на фон открытого диалога, селектор меняет акцент, а фокус возвращается после закрытия. Включите системное уменьшение движения или эмуляцию `prefers-reduced-motion` в DevTools. Кнопки и галерея должны остаться рабочими без анимации.

Подробнее в документации — [элемент dialog](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog), [CSS Scroll Snap](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scroll_snap), [CSS transitions](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_transitions), [prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion), [ARIA APG для select-only combobox](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/examples/combobox-select-only/).


<a id="containers"></a>

## 10. Упакуйте два сервиса и Nginx в Docker Compose

Сначала убедитесь, что IDEA запускает оба сервиса и `./mvnw clean verify` собирает два исполняемых JAR. Рабочие файлы — `repository-service/target/app.jar` и `web-service/target/app.jar`. Они независимы: у каждого свой класс `main` и свой набор зависимостей. Команда `verify` в этом руководстве проверяет сборку, а не заменяет проверку API: автотесты в стартовый проект не включены.

В корне создайте общий многоэтапный `Dockerfile`. Аргумент `SERVICE` определяет, какой модуль собрать. В итоговый образ попадёт только соответствующий JAR и JRE, без Maven и исходников.

<!-- file:Dockerfile -->
~~~dockerfile
# syntax=docker/dockerfile:1
FROM eclipse-temurin:21-jdk AS build
WORKDIR /src
COPY .mvn/ .mvn/
COPY mvnw pom.xml ./
COPY repository-service/ repository-service/
COPY web-service/ web-service/
ARG SERVICE
RUN chmod +x mvnw
RUN --mount=type=cache,target=/root/.m2,sharing=locked \
    ./mvnw -B -pl "$SERVICE" -am -DskipTests package && \
    cp "$SERVICE/target/app.jar" /out.jar

FROM eclipse-temurin:21-jre
WORKDIR /opt/app
RUN mkdir -p /data/uploads && touch /data/uploads/.keep \
    && chown -R 10001:10001 /data
COPY --from=build /out.jar app.jar
USER 10001:10001
ENTRYPOINT ["java", "-jar", "app.jar"]
~~~

<!-- file:.dockerignore -->
~~~
.git
.idea
**/target
uploads
unity
.env
~~~

Теперь **замените** `compose.yaml` полной версией. `db` остался тем же сервисом с тем же томом. Профиль `stack` подключает три дополнительных контейнера. `docker compose up -d --wait db` по-прежнему запускает только базу для IDEA.

<!-- file:compose.yaml -->
~~~yaml
services:
  db:
    image: postgres:17
    environment:
      POSTGRES_DB: portfolio
      POSTGRES_USER: portfolio
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-portfolio_dev}
    ports:
      - "127.0.0.1:5433:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U portfolio -d portfolio"]
      interval: 3s
      timeout: 3s
      retries: 20

  repository:
    profiles: ["stack"]
    build:
      context: .
      args:
        SERVICE: repository-service
    environment:
      SERVER_ADDRESS: "0.0.0.0"
      DB_URL: jdbc:postgresql://db:5432/portfolio
      DB_USER: portfolio
      DB_PASSWORD: ${POSTGRES_PASSWORD:-portfolio_dev}
    depends_on:
      db:
        condition: service_healthy

  web:
    profiles: ["stack"]
    build:
      context: .
      args:
        SERVICE: web-service
    environment:
      SERVER_ADDRESS: "0.0.0.0"
      REPOSITORY_BASE_URL: http://repository:8082
      UPLOAD_DIR: /data/uploads
    volumes:
      - uploads:/data/uploads
    depends_on:
      - repository

  nginx:
    profiles: ["stack"]
    image: nginx:stable-alpine
    ports:
      - "127.0.0.1:8081:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
      - ./unity:/usr/share/nginx/html/unity:ro
    depends_on:
      - web

volumes:
  pgdata:
  uploads:
~~~

Создайте `nginx/default.conf`. Регулярные locations ниже отвечают за уже сжатые файлы. Модуль компрессии Brotli в Nginx не нужен: сервер отдаёт байты, которые заранее сжал Unity. `location /unity/` не должен иметь модификатор `^~`, иначе сработает префикс и Nginx пропустит регулярные правила с заголовками.

<!-- file:nginx/default.conf -->
~~~nginx
server {
    listen 80;
    server_name _;
    absolute_redirect off;
    root /usr/share/nginx/html;
    client_max_body_size 12m;

    location = /unity {
        return 308 /unity/;
    }

    location ~ ^/unity/.+\.wasm\.br$ {
        gzip off;
        types {}
        default_type application/wasm;
        add_header Content-Encoding br;
        try_files $uri =404;
    }

    location ~ ^/unity/.+\.js\.br$ {
        gzip off;
        types {}
        default_type application/javascript;
        add_header Content-Encoding br;
        try_files $uri =404;
    }

    location ~ ^/unity/.+\.br$ {
        gzip off;
        types {}
        default_type application/octet-stream;
        add_header Content-Encoding br;
        try_files $uri =404;
    }

    location /unity/ {
        index index.html;
        try_files $uri $uri/ =404;
    }

    location / {
        proxy_pass http://web:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
~~~

Создайте каталог `unity/` в корне, даже если самого экспорта ещё нет, затем проверьте стенд. `nginx/default.conf` нужно сохранить **до** запуска Compose.

~~~bash
mkdir -p unity
docker compose --profile stack config --quiet
docker compose --profile stack up -d --build
docker compose --profile stack ps
docker compose --profile stack logs --tail=60 repository web nginx
docker compose --profile stack exec nginx nginx -t
curl -fsS http://localhost:8081/api/categories
~~~

Проверьте главную страницу на `http://localhost:8081/`. Если сразу после запуска Nginx возвращает `502`, дождитесь сообщения о старте в логах `web` и `repository`. `depends_on` с проверкой готовности применён к PostgreSQL. Для остальных сервисов он задаёт порядок старта, но не гарантирует, что Java уже принимает запросы. Если после пересоздания только контейнера web ошибка остаётся, перезапустите Nginx командой `docker compose --profile stack restart nginx`, чтобы он заново разрешил имя сервиса.

В IDEA можно подключить Docker: **Settings → Build, Execution, Deployment → Docker → +**, выберите соединение с Docker Engine через сокет, проверьте **Connection successful**. В **Services** появятся контейнеры и их логи. Если IDE-конфигурация Compose не передаёт профиль `stack`, используйте приведённую команду в терминале IDEA.

Для обычной остановки используйте `docker compose --profile stack down`. Два именованных тома переживут остановку. Команда `down -v` удаляет содержимое обоих томов, поэтому для обычного перезапуска она не нужна. Пересборка образов не очищает данные.

Подробнее в документации — [многоэтапная сборка Docker](https://docs.docker.com/build/building/multi-stage/), [профили Compose](https://docs.docker.com/compose/how-tos/profiles/), [сеть Compose](https://docs.docker.com/compose/how-tos/networking/), [Nginx proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass).

<a id="unity-web"></a>

## 11. Экспортируйте Unity Web и проверьте Brotli

IntelliJ IDEA здесь запускает Java и управляет файлами сайта, но сборку Unity создаёт **Unity Editor**. Если у вас уже есть готовый Web-экспорт, используйте его и начните с копирования. Если редактор на Arch работает нестабильно, проверьте список поддерживаемых ОС для своей версии Unity. Arch не входит в список официально поддерживаемых Linux-дистрибутивов для Unity 6.0. Можно собрать проект на поддерживаемой ОС и перенести в Arch только готовую папку экспорта.

1. Для подходящей версии Unity Editor установите в Unity Hub модуль **Web Build Support**. Название модуля в некоторых версиях может содержать **WebGL**.
2. В редакторе откройте **File → Build Profiles → Web**, добавьте или активируйте профиль Web и включите нужную сцену в список сцен. Отключите **Development Build** для релизного размера.
3. В **Edit → Project Settings → Player → Web → Publishing Settings** установите **Compression Format → Brotli** и снимите **Decompression Fallback**. При выключенном fallback браузер получает сжатые файлы с HTTP-заголовками и распаковывает их сам. Пример Nginx рассчитан на обычные файлы `.br`, а не на `.unityweb`.
4. Для первого проекта оставьте многопоточность Web выключенной. При её включении нужна кросс-оригинная изоляция и отдельная проверка COOP/COEP. Эти настройки не требуются в конфигурации для обычной однопоточной сборки.
5. Нажмите **Build**, выберите чистую папку экспорта с простым именем, например `WebBuild`, и скопируйте её **содержимое целиком** в пустой `unity/` рядом с `compose.yaml`. В результате нужны `unity/index.html`, `unity/Build/…` и остальные папки, которые действительно создал экспорт, например `TemplateData` или `StreamingAssets`. Не переименовывайте сгенерированные ресурсы вручную: загрузчик уже ссылается на их имена.
6. Откройте `http://localhost:8081/unity/` через Nginx. Не используйте `file://` и не проверяйте этот путь на прямом порту Spring `8080`. Для размещения сайта вне localhost настройте HTTPS, а потом повторите проверку ответов.

Проверьте фактический файл `.wasm.br`. Следующие команды выполняются в корне проекта. При отсутствии файла проверьте выбранный формат сжатия, fallback и правильность пути экспорта.

~~~bash
test -f unity/index.html
find unity/Build -maxdepth 1 -name '*.wasm.br' -print
wasm_file=$(find unity/Build -maxdepth 1 -name '*.wasm.br' -print -quit)
if [ -n "$wasm_file" ]; then
  curl -fsSI -H 'Accept-Encoding: br' \
    "http://localhost:8081/${wasm_file// /%20}"
fi

find unity -type f -printf '%s\n' | awk '
  { bytes += $1; files++ }
  END {
    printf "%d файлов, %d байт\n", files, bytes
    exit (files == 0 || bytes > 100000000)
  }'
~~~

Для Wasm ожидайте `200`, `Content-Type: application/wasm` и `Content-Encoding: br`. Для `.js.br` ожидайте JavaScript MIME и `br`, для `.data.br` — `application/octet-stream` и `br`. В браузере откройте DevTools → **Network**, включите **Disable cache**, перезагрузите страницу и проверьте, что сцена запустилась. Не ориентируйтесь на `Content-Type` главного HTML: нужны заголовки самих ресурсов `.br`.

**Что значит «до 100 МБ».** Здесь бюджет — не более **100 000 000 байт** для всех файлов, которые вы выкладываете в `unity/`. Это строже, чем проверять только Wasm. Дополнительно запишите холодный трафик первого запуска по столбцу **Transferred** в Network. Вес скачивания не равен потреблению памяти после распаковки. Если размер больше, сначала посмотрите крупнейшие файлы, уменьшите разрешение текстур, выберите подходящее сжатие аудио и исключите ненужные сцены и ресурсы. Затем сделайте новый экспорт в чистую папку и измерьте снова.

Если видите `404`, проверьте имя файла и относительные пути Unity. Если не распаковывается Brotli, проверьте `Content-Encoding`. Если Wasm не компилируется потоково, проверьте MIME `application/wasm`. Для финальной проверки используйте и браузер, и `curl`: один заголовок не доказывает, что сцена запустилась.

Подробнее в документации — [экспорт и сжатие Unity Web](https://docs.unity.com/en-us/engine/6000.0/manual/platform-specific/webgl/building-distribution/deploying), [настройка Nginx для Unity](https://docs.unity.com/en-us/engine/6000.3/manual/platform-specific/webgl/building-distribution/server-configuration-code-samples/web-server-config-nginx), [поддерживаемые ОС Unity 6.0](https://docs.unity.com/en-us/engine/6000.0/manual/get-started/install-and-upgrade/getting-started-installing-unity/system-requirements).

<a id="acceptance"></a>

## 12. Приёмка и оформление портфолио

| Проверка | Что считается результатом |
| --- | --- |
| Резюме | На главной ваше имя и роль. Указаны проекты, личный вклад и контакты. Все опубликованные ссылки открываются. |
| Сервисы | Есть два исполняемых JAR. В Compose у web и repository нет опубликованных портов. Сайт читает данные через отдельный сервис. При отключении repository статическая главная по-прежнему работает. |
| Каталог и поиск | Номер страницы меняется, прямой переход работает, размер ограничен. Подсказки обновляются при вводе. Enter фильтрует выдачу. Клик открывает точный товар. |
| Лента | Порции дописываются без повторов. В конце запросы прекращаются. При ошибке доступна кнопка повторной загрузки. |
| Счётчики | Четыре категории по 12 500, одна пустая. Сумма 50 000. |
| Доска и файлы | Порядок переживает обновление страницы. При сбое происходит откат. Можно выбрать файл и перетащить его. Файл больше лимита отклоняется. |
| Интерфейс | Галерея листается, модальное окно работает с клавиатуры, выбор меняет тему. При reduced motion компоненты остаются рабочими. |
| Unity | Сцена запускается. Wasm отдаётся с правильными MIME и Brotli-заголовком. Суммарный размер экспорта не больше 100 000 000 байт. |

Идея для README и собеседования: покажите два `EXPLAIN (ANALYZE, BUFFERS)`, один скриншот переноса карточки и ответ с заголовками Unity. Напишите своими словами, почему лента использует курсор, почему счётчик пустой категории равен нулю и почему веб-сервис не обращается к PostgreSQL напрямую. Это лучше демонстрирует понимание, чем список из семи технологий.

Добавьте в `.gitignore` как минимум `/uploads/`, `/unity/` и `.env`. Unity-экспорт можно хранить отдельно от исходников сайта. На случай публикации замените локальные пароли и уберите опубликованный порт БД из публичной конфигурации. На публичном портфолио не оставляйте произвольную загрузку с записью на диск анонимным посетителям. Оставьте локальный предпросмотр, либо подключите авторизацию, квоту, срок хранения и очистку. Для публичной доски используйте демоданные на посетителя или запретите общую анонимную запись. Это не нужно для прохождения практикума на localhost.

**Если что-то не работает**

| Симптом | Что смотреть первым |
| --- | --- |
| IDEA не видит Spring или классы подсвечены красным | Project SDK, Maven Wrapper, Reload All Maven Projects, выбранный модуль |
| `permission denied` на сокете Docker | Активная сессия и доступ к Docker. После изменения группы войдите заново |
| `Connection refused` на JDBC | `docker compose ps`, порт `5433` на хосте и `db:5432` внутри сети |
| Ошибка Flyway checksum | Сравните уже применённую миграцию с исходной. Изменение оформите новой версией |
| Главная работает, API отдаёт `502` | Логи `repository`, адрес `REPOSITORY_BASE_URL`, доступность БД, таймаут клиента |
| `403` или `404` на `/unity/` | Открыт порт Nginx `8081`, существует `unity/index.html`, каталог смонтирован |
| `413` при загрузке | Сверьте лимит Spring на файл/запрос и `client_max_body_size` в Nginx |
| После перезапуска нет файлов | Убедитесь, что `UPLOAD_DIR` и путь подключения тома совпадают |

Начните с короткого контрольного круга: база в Docker → один запрос напрямую в `repository-service` → тот же запрос через `web-service` → личная главная страница. Затем добавляйте демо по одному. Так каждый новый участок можно проверить отдельно.

