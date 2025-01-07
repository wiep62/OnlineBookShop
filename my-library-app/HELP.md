# Read Me First
The following was discovered as part of building this project:

* The original package name 'com.example.my-library-app' is invalid and this project uses 'com.example.my_library_app' instead.

# Getting Started

### Reference Documentation
For further reference, please consider the following sections:

* [Official Apache Maven documentation](https://maven.apache.org/guides/index.html)
* [Spring Boot Maven Plugin Reference Guide](https://docs.spring.io/spring-boot/3.4.1/maven-plugin)
* [Create an OCI image](https://docs.spring.io/spring-boot/3.4.1/maven-plugin/build-image.html)

### Maven Parent overrides

Due to Maven's design, elements are inherited from the parent POM to the project POM.
While most of the inheritance is fine, it also inherits unwanted elements like `<license>` and `<developers>` from the parent.
To prevent this, the project POM contains empty overrides for these elements.
If you manually switch to a different parent and actually want the inheritance, you need to remove those overrides.

**********************************************************

Открываем наш проект в среде разработки и вначале немного модифицируем наш pom.xml файл.

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>
<parent>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-parent</artifactId>
<version>2.7.11</version>
<relativePath/> <!-- lookup parent from repository -->
</parent>
<groupId>com.example</groupId>
<artifactId>my-library-app</artifactId>
<version>0.0.1-SNAPSHOT</version>
<name>my-library-app</name>
<description>Demo project for Spring Boot</description>
<properties>
<java.version>17</java.version>
<org.mapstruct.version>1.4.2.Final</org.mapstruct.version>
<lombok-mapstruct-binding.version>0.2.0</lombok-mapstruct-binding.version>
<maven.compiler.plugin.version>3.5.1</maven.compiler.plugin.version>
</properties>
<dependencies>
<!--JPA-->
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<!--WEB-->
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!--LIQUIBASE-->
<dependency>
<groupId>org.liquibase</groupId>
<artifactId>liquibase-core</artifactId>
</dependency>
<!--DEV-TOOLS-->
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-devtools</artifactId>
<scope>runtime</scope>
<optional>true</optional>
</dependency>
<!--DATABASE-->
<dependency>
<groupId>org.postgresql</groupId>
<artifactId>postgresql</artifactId>
<scope>runtime</scope>
</dependency>
<!--LOMBOK-->
<dependency>
<groupId>org.projectlombok</groupId>
<artifactId>lombok</artifactId>
<optional>true</optional>
</dependency>
<!--MAPPING-->
<dependency>
<groupId>org.mapstruct</groupId>
<artifactId>mapstruct</artifactId>
<version>${org.mapstruct.version}</version>
</dependency>
<dependency>
<groupId>org.projectlombok</groupId>
<artifactId>lombok-mapstruct-binding</artifactId>
<version>${lombok-mapstruct-binding.version}</version>
</dependency>
<!--TESTING-->
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-test</artifactId>
<scope>test</scope>
</dependency>
</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<excludes>
						<exclude>
							<groupId>org.projectlombok</groupId>
							<artifactId>lombok</artifactId>
						</exclude>
					</excludes>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>${maven.compiler.plugin.version}</version>
				<configuration>
					<source>17</source>
					<target>17</target>
					<annotationProcessorPaths>
						<path>
							<groupId>org.mapstruct</groupId>
							<artifactId>mapstruct-processor</artifactId>
							<version>${org.mapstruct.version}</version>
						</path>
						<path>
							<groupId>org.projectlombok</groupId>
							<artifactId>lombok</artifactId>
							<version>${lombok.version}</version>
						</path>
						<path>
							<groupId>org.projectlombok</groupId>
							<artifactId>lombok-mapstruct-binding</artifactId>
							<version>${lombok-mapstruct-binding.version}</version>
						</path>
					</annotationProcessorPaths>
				</configuration>
			</plugin>
		</plugins>
	</build>

</project>
Из основных зависимостей:

postgresql - доступ к базе данных;

liquibase - будет использоваться для "накатывания" таблиц в базе данных и заполнения первоначальными данными.

mapstruct - для маппинга наших сущностей.

lombok - для сокращения шаблонного кода через использование аннотаций.

Начнем с кода нашей сущности, как я упоминал раньше - это будет книга (Book). Поэтому создадим класс Book.

@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "book")
public class Book {

    @Id
    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "title")
    private String title;

    @Column(name = "author")
    private String author;

    @Column(name = "year")
    private int year;

}
Далее интерфейс BookRepository и унаследуем его от JpaRepository - чтобы получить набор стандартных методов JPA для работы с БД.

public interface BookRepository extends JpaRepository<Book, Long> {
}
Создадим класс BookDto, так как уровень сервиса не должен напрямую работать с нашей сущностью Book (может в данном конкретном случае это и не нужно, так как это немного усложняет код).

@Data
@NoArgsConstructor
@AllArgsConstructor
public class BookDto {

    private Long id;
    private String title;
    private String author;
    private int year;
}
Создадим интерфейс BookMapper для маппинга наших сущностей из Book и BookDto и обратно, здесь мы и используем mapstruct.

@Mapper(componentModel = "spring")
public interface BookMapper {
Book dtoToModel(BookDto bookDto);

    BookDto modelToDto(Book book);

    List<BookDto> toListDto(List<Book> books);
}
Создадим еще один интерфейс BookService, в котором и напишем основные методы работы с нашими книгами.

public interface BookService {
List<BookDto> findAll ();
BookDto findById( Long id);
BookDto save (BookDto book);
void deleteById (Long id);
}
и реализуем эти методы в классе BookServiceImpl.

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class BookServiceImpl implements BookService {

    private final BookRepository bookRepository;
    private final BookMapper bookMapper;

    @Override
    public List<BookDto> findAll() {
        return bookMapper.toListDto(bookRepository.findAll());
    }

    @Override
    public BookDto findById(Long id) {
        return Optional.of(getById(id)).map(bookMapper::modelToDto).get();
    }

    @Override
    @Transactional
    public BookDto save(BookDto book) {
        return bookMapper.modelToDto(bookRepository.save(
                bookMapper.dtoToModel(book)));
    }

    @Override
    @Transactional
    public void deleteById(Long id) {
        var book = getById(id);
        bookRepository.delete(book);
    }

    private Book getById(Long id) {
        return bookRepository.findById(id)
                .orElseThrow(() -> new RuntimeException(
                        "Book with id: " + id + " not found"));
    }
}
Последний класс, который мы создадим здесь - это будет BooksController - который и будет выдавать наши эндпойнты для просмотра, добавления, редактирования и удаления наших книг.

@RestController
@RequestMapping("api/v1")
@RequiredArgsConstructor
public class BooksController {
private final BookService bookService;

    @GetMapping("/books")
    public List<BookDto> allBooks() {
        return bookService.findAll();
    }

    @GetMapping("/book/{id}")
    @ResponseStatus(HttpStatus.OK)
    public ResponseEntity<BookDto> getBook(@PathVariable Long id) {
        return ResponseEntity.ok().body(bookService.findById(id));
    }

    @PostMapping("/book")
    public ResponseEntity<BookDto> createBook( @RequestBody BookDto book) throws URISyntaxException {
        BookDto result = bookService.save(book);
        return ResponseEntity.created(new URI("/api/v1/books/" + result.getId()))
                .body(result);
    }

    @PutMapping("/book/{id}")
    @ResponseStatus(HttpStatus.OK)
    public ResponseEntity<BookDto> updateBook( @PathVariable Long id, @RequestBody BookDto book) {
        return ResponseEntity.ok().body(bookService.save(book));
    }

    @DeleteMapping("/book/{id}")
    public ResponseEntity<?> deleteBook(@PathVariable Long id) {
        bookService.deleteById(id);
        return ResponseEntity.ok().build();
    }

}
Сейчас займемся нашим application.properties файлом и пропишем там доступ к нашей базе данных, включим настройки liquibase, настроим порт 8181 - чтобы наш бэкенд поднимался именно на этом порту, а также включим другие настройки.

server.port=8181

spring.datasource.url=jdbc:postgresql://localhost:15432/books_db
spring.datasource.username=username
spring.datasource.password=password

spring.jpa.hibernate.ddl-auto=none
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.show-sql=true

spring.mvc.pathmatch.matching-strategy = ANT_PATH_MATCHER

spring.liquibase.enabled=true
spring.liquibase.drop-first=false
spring.liquibase.change-log=classpath:db/changelog/db.changelog-master.xml
spring.liquibase.default-schema=public
Осталось написать liquibase скрипты для накатывания таблицы базы данных и вставки первоначальной информации.

Вначале создадим структуру папок как на рисунке


То есть в папке resources создаем папку db и в ней папку changelog, в которой находится файл db.changelog-master.xml, путь к нему прописан в нашем файле application.properties. Это будет основной файл, который будет запускать скрипты.

<?xml version="1.0" encoding="UTF-8" ?>
<databaseChangeLog
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

    <include file="v.1.0.0/cumulative.xml" relativeToChangelogFile="true" />

</databaseChangeLog>
Далее в папке changelog создаем папку v.1.0.0, где создаем три файла.

2023-05-12-1-create-table-book.xml - накатываем таблицу book

<?xml version="1.0" encoding="UTF-8" ?>
<databaseChangeLog
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

    <changeSet logicalFilePath="2023-05-12-1-create-table-book"
               id="2023-05-12-1-create-table-book" author="s.m">
        <createTable tableName="book">
            <column name="id" type="serial">
                <constraints nullable="false" primaryKey="true"/>
            </column>
            <column name="title" type="varchar(100)">
                <constraints nullable="false"/>
            </column>
            <column name="author" type="varchar(100)">
                <constraints nullable="false"/>
            </column>
            <column name="year" type="int">
                <constraints nullable="false"/>
            </column>
        </createTable>
    </changeSet>

</databaseChangeLog>
файл 2023-05-12-2-insert-books.xml - вставляем 6 книг в нашу таблицу

<?xml version="1.0" encoding="UTF-8" ?>
<databaseChangeLog
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

    <changeSet logicalFilePath="2023-05-12-2-insert-books"
               id="2023-05-12-2-insert-books" author="s.m">
        <insert tableName="book">
            <column  name="title"  value="Effective Java"/>
            <column  name="author"  value="Joshua Bloch"/>
            <column  name="year"  value="2018"/>
        </insert>
        <insert tableName="book">
            <column  name="title"  value="Java Projects"/>
            <column  name="author"  value="Peter Verhas"/>
            <column  name="year"  value="2018"/>
        </insert>
        <insert tableName="book">
            <column  name="title"  value="Spring in Action, 5th Edition"/>
            <column  name="author"  value="Craig Walls"/>
            <column  name="year"  value="2018"/>
        </insert>
        <insert tableName="book">
            <column  name="title"  value="Java Persistence with Hibernate, Second Edition"/>
            <column  name="author"  value="Christian Bauer, Gavin King, and Gary Gregory"/>
            <column  name="year"  value="2015"/>
        </insert>
        <insert tableName="book">
            <column  name="title"  value="Java: A Beginner's Guide, Eighth Edition"/>
            <column  name="author"  value="Herbert Schildt"/>
            <column  name="year"  value="2019"/>
        </insert>
        <insert tableName="book">
            <column  name="title"  value="Spring Boot 2 Recipes"/>
            <column  name="author"  value="Marten Deinum"/>
            <column  name="year"  value="2018"/>
        </insert>

    </changeSet>

</databaseChangeLog>
и файл cumulative.xml, где мы аккумулируем все наши файлы со скриптами, которые хотим запустить.

<?xml version="1.0" encoding="UTF-8" ?>
<databaseChangeLog
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

    <include file="2023-05-12-1-create-table-book.xml" relativeToChangelogFile="true" />
    <include file="2023-05-12-2-insert-books.xml" relativeToChangelogFile="true" />

</databaseChangeLog>
Еще напишем небольшой скрипт, который нам понадобится позже. К нему будет обращаться база данных postgreSQL и создавать там саму базу данных, к которой мы и будем подключаться.

Для этого создадим папку infrastructure, а в ней папку db. Должно получиться так.


И уже в папке db создадим файл create_db.sql - здесь мы создаем базу данных books_db.

create database books_db;
Сейчас в корне нашего проекта создаем Dockerfile


FROM maven:3.8.4-openjdk-17 as builder
WORKDIR /app
COPY . /app/.
RUN mvn -f /app/pom.xml clean package -Dmaven.test.skip=true

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar /app/*.jar
EXPOSE 8181
ENTRYPOINT ["java", "-jar", "/app/*.jar"]
Dockerfile нужен для того чтобы на основании его мы создали образ, а уже на основании образа запустили контейнер с нашим бэкендом.

Здесь мы используем многоэтапную сборку, на первом этапе мы получаем jar-ник нашего проекта, а на втором мы его уже запускаем.

Пройдемся по Dockerfile

FROM maven:3.8.4-openjdk-17 as builder - указываем на основании какого образа, который докер стянет с docker hub мы будем билдить наш проект, as builder - это название, которое мы присвоили, для того чтобы обратиться с другого слоя образа для получения данных.

WORKDIR /app - создаем директорию app внутри слоя образа.

COPY . /app/. - копируем все наши папки с текущего проекта в папку app в слое образа.

RUN mvn -f /app/pom.xml clean package -Dmaven.test.skip=true - запускаем maven, который билдит наш проект и получаем jar-ник.

FROM eclipse-temurin:17-jre-alpine - снова указываем на основании какого образа, мы будем запускать наш проект, здесь уже мы не используем jdk, а только jre - так как нам не нужны инструменты разработчика.

WORKDIR /app - создаем директорию app в новом слое образа.

COPY --from=builder /app/target/.jar /app/.jar - копируем с предыдущего слоя с папки target наш jar-ник в папку app.

EXPOSE 8181 - указываем на каком порту должен работать наш контейнер.

ENTRYPOINT ["java", "-jar", "/app/*.jar"] - запускаем наше приложение в контейнере.

Пришло время написать сам docker-compose.yml файл.

Снова в корне нашего проекта создаем файл docker-compose.yml.

version: '3.8'
services:
client-backend:
image: client:0.0.1
build:
context: .
dockerfile: Dockerfile
ports:
- "8181:8181"
depends_on:
- service-db
environment:
- SERVER_PORT= 8181
- SPRING_DATASOURCE_URL=jdbc:postgresql://service-db/books_db

service-db:
image: postgres:14.7-alpine
environment:
POSTGRES_USER: username
POSTGRES_PASSWORD: password
ports:
- "15432:5432"
volumes:
- ./infrastructure/db/create_db.sql:/docker-entrypoint-initdb.d/create_db.sql
- db-data:/var/lib/postgresql/data
restart: unless-stopped

pgadmin
container_name: pgadmin4_container
image: dpage/pgadmin4:7
restart: always
environment:
PGADMIN_DEFAULT_EMAIL: admin@admin.com
PGADMIN_DEFAULT_PASSWORD: root
ports:
- "5050:80"
volumes:
- pgadmin-data:/var/lib/pgadmin

volumes:
db-data:
pgadmin-data:
Пройдемся немного по Dockerfile.

version: '3.8' - указываем версию docker compose

services: - указываем какие контейнеры нам нужно будет поднять.

client-backend: - название первого контейнера с нашим бэкендом.

image: client:0.0.1 - указываем название контейнера и его тэг (можно без тэга, тогда будет автоматически присвоен latest).

build: - указываем, что мы хотим его получить не скачивая с докер хаба, а будем "строить" на основании Dockerfile.

context: . - указываем расположение Dockerfile, он у нас располагается в той же папке, что и docker-compose.yml

dockerfile: Dockerfile - указываем наименование Dockerfile

    ports:
      - "8181:8181"
здесь указываем первый порт - это внешний порт, с помощью которого мы можем получить доступ к нашему контейнеру, а второй порт - это внутренний порт контейнера.

    depends_on:
      - service-db
указываем, что наш контейнер должен подняться после того как поднимется контейнер с базой данных, так как, если контейнер с бэкендом поднимется первым и не будет доступа к базе данных, то он упадет с ошибкой.

    environment:
      - SERVER_PORT= 8181
      - SPRING_DATASOURCE_URL=jdbc:postgresql://service-db/books_db
указываем дополнительные настройки, в том числе url для подключения к базе данных. В этом пути мы уже используем внутреннее имя нашего контейнера с базой данных service-db.

service-db:
image: postgres:14.7-alpine
указываем имя нашего контейнера с базой данных, а также имя образа, который будет скачен с докер хаба. Желательно при этом указывать тэг образа, то есть версию, если версия не будет указана, то будет скачена и использована самая последняя версия данного образа, что может привести в будущем к несовместимости по версиям и ошибкам.

    environment:
      POSTGRES_USER: username
      POSTGRES_PASSWORD: password
указываем username и password, они должны совпадать с теми, что мы указали в application.properties.

    ports:
      - "15432:5432"
указываем порты.

    volumes:
      - ./infrastructure/db/create_db.sql:/docker-entrypoint-initdb.d/create_db.sql
      - db-data:/var/lib/postgresql/data
здесь мы указываем, чтобы выполнился наш скрипт, который мы написали ранее и который расположен в папке infrastructure/db по созданию базы данных books_db.

а во втором volume мы делаем чтобы наши данные сохранялись не локально в наш контейнер, а в файл, расположенный вне контейнера. Это нужно для того чтобы при перезапуске образа, если мы его удалим и поднимем снова, чтобы наши изменения в базе данных не потерялись, а сохранились. Это я продемонстрирую чуть позже.

С контейнером pgadmin вроде все понятно, остановлюсь только на

    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: root
это данные для входа в pgadmin по умолчанию.

Написали много кода, теперь его надо протестить, вначале протестим его без фронта, но потом еще добавил и его.

Итак, переходим в терминал, в папку с проектом и выполняем команду docker-compose up


После выполнения данной команды, необходимо будет подождать несколько минут пока докер сбилдит наш образ, а также стянет все образы с докер хаба.

В конце, в консоли вы должны будете увидеть запуск нашего бэкенда.

И если мы зайдем в Docker Desktop, то должны увидеть все три наши работающие контейнеры.


Давайте протестим как работает наш бэкенд. Для этого через Postman выполним команду http://localhost:8181/api/v1/books


и мы получим 6 наших книг, которые мы записали с помощью liquibase.

Зайдем в браузере на http://localhost:5050 и введем username: admin@admin.com и пароль: root


Мы должны зайти на pgAdmin


Здесь мы должны создать подключение к нашей базе данных. Для этого нажимаем Add New Server.


Name задаем любое.

Во вкладке Connection указываем


Host name - наименование, которое мы указали для контейнера с нашей БД (service-db).

Port - внутренний порт контейнера БД (5432).

Maintenance database - наименование нашей базы данных (books_db).

Username - тот юзернайм, который мы указали в docker-compose.yml для подключения к БД (username).

Password- тот пароль, который мы указали в docker-compose.yml для подключения к БД (password).

Далее если мы найдем нашу таблицу book и выполним в ней SELECT * FROM book мы увидим 6 наших книг.



Давайте добавим еще одну книгу. Для этого идем в Postman и отправим POST запрос на адрес http://localhost:8181/api/v1/book с JSON книги, которую мы хотим добавить.


Идем обратно в pgAdmin и смотрим какие книги у нас есть.


Последняя книга также появилась.

Сейчас остановим все наши контейнеры и удалим их, это можно сделать или через Docker Desktop или командами в терминале. Я это сделаю через Docker Desktop.

И снова их поднимем с помощью команды docker-compose up.

И зайдя снова на pgAdmin мы увидим 7 нашим книг, в том числе с последней, которую мы добавили.

Сейчас осталась последняя часть нашей работы - написать фронтэнд на React.

Для этого у вас на компьютере должен быть установлен Node.js и npm.

Чтобы проверить какие версии у вас установлены или вообще есть ли у вас Node.js и npm, можно воспользоваться командами node --version и npm --version.


Переходим в терминал и в корне нашего проекта выполняем команду npx create-react-app@5 frontend


Данной командой мы создадим React проект в папке frontend (папка появиться автоматически) в нашем основном проекте.

Командой cd frontend зайдем в папку frontend.

И выполним команду npm i bootstrap@5 react-cookie@4 react-router-dom@6 reactstrap@9 , данной командой мы установим Bootstrap , поддержку файлов cookie для React, React Router и Reactstrap.


Если открыть папку frontend, то мы увидим следующую структуру папок.

Данный проект c фронтом можно делать и отдельно в другой директории (не в проекте с бэкендом), тогда надо будет поменять путь к Dockerfile к фронту в docker-compose.yml


В файл index.js добавим следующий импорт import 'bootstrap/dist/css/bootstrap.min.css';


Также в файл package.json добавим строку "proxy": "http://host.docker.internal:8181" - для общения нашего фронта с бэкендом на порту 8181 через контейнер.



Добавим файл BookEdit.js для редактирования книг.

import React, { useEffect, useState } from 'react';
import { Link, useNavigate, useParams } from 'react-router-dom';
import { Button, Container, Form, FormGroup, Input, Label } from 'reactstrap';
import AppNavbar from './AppNavbar';

const BookEdit = () => {
const initialFormState = {
title: '',
author: '',
year: ''
};
const [book, setBook] = useState(initialFormState);
const navigate = useNavigate();
const { id } = useParams();

    useEffect(() => {
        if (id !== 'new') {
            fetch(`/api/v1/book/${id}`)
                .then(response => response.json())
                .then(data => setBook(data));
        }
    }, [id, setBook]);

    const handleChange = (event) => {
        const { name, value } = event.target

        setBook({ ...book, [name]: value })
    }

    const handleSubmit = async (event) => {
        event.preventDefault();

        await fetch(`/api/v1/book${book.id ? `/${book.id}` : ''}`, {
            method: (book.id) ? 'PUT' : 'POST',
            headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(book)
        });
        setBook(initialFormState);
        navigate('/books');
    }

    const title = <h2>{book.id ? 'Edit Book' : 'Add Book'}</h2>;

    return (<div>
            <AppNavbar/>
            <Container>
                {title}
                <Form onSubmit={handleSubmit}>
                    <FormGroup>
                        <Label for="title">Title</Label>
                        <Input type="text" name="title" id="title" value={book.title || ''}
                               onChange={handleChange} autoComplete="name"/>
                    </FormGroup>
                    <FormGroup>
                        <Label for="author">Author</Label>
                        <Input type="text" name="author" id="author" value={book.author || ''}
                               onChange={handleChange} autoComplete="address-level1"/>
                    </FormGroup>
                    <FormGroup>
                        <Label for="author">Year</Label>
                        <Input type="text" name="year" id="year" value={book.year || ''}
                               onChange={handleChange} autoComplete="address-level1"/>
                    </FormGroup>

                    <FormGroup>
                        <Button color="primary" type="submit">Save</Button>{' '}
                        <Button color="secondary" tag={Link} to="/books">Cancel</Button>
                    </FormGroup>
                </Form>
            </Container>
        </div>
    )
};

export default BookEdit;
Добавим также файл BookList.js для отображения книг.

import React, { useEffect, useState } from 'react';
import { Button, ButtonGroup, Container, Table } from 'reactstrap';
import AppNavbar from './AppNavbar';
import { Link } from 'react-router-dom';

const BookList = () => {

    const [books, setBooks] = useState([]);
    const [loading, setLoading] = useState(false);

    useEffect(() => {
        setLoading(true);

        fetch('/api/v1/books')
            .then(response => response.json())
            .then(data => {
                setBooks(data);
                setLoading(false);
            })
    }, []);

    const remove = async (id) => {
        await fetch(`/api/v1/book/${id}`, {
            method: 'DELETE',
            headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json'
            }
        }).then(() => {
            let updatedBooks = [...books].filter(i => i.id !== id);
            setBooks(updatedBooks);
        });
    }

    if (loading) {
        return <p>Loading...</p>;
    }

    const bookList = books.map(book => {
        return <tr key={book.id}>
            <td style={{whiteSpace: 'nowrap'}}>{book.title}</td>
            <td> {book.author || ''} </td>
            <td> {book.year || ''} </td>
            <td>
                <ButtonGroup>
                    <Button size="sm" color="primary" tag={Link} to={"/books/" + book.id}>Edit</Button>
                    <Button size="sm" color="danger" onClick={() => remove(book.id)}>Delete</Button>
                </ButtonGroup>
            </td>
        </tr>
    });

    return (
        <div>
            <AppNavbar/>
            <tr></tr>
            <Container fluid>
                <div className="float-end" >
                    <Button color="success" tag={Link} to="/books/new">Add Book</Button>
                </div>
                <h3>My Books</h3>
                <Table className="mt-4">
                    <thead>
                    <tr>
                        <th width="20%">Title</th>
                        <th width="20%">Author</th>
                        <th width="20%">Year</th>
                        <th width="10%">Actions</th>
                    </tr>
                    </thead>
                    <tbody>
                    {bookList}
                    </tbody>
                </Table>
            </Container>
        </div>
    );
};

export default BookList;
Файл App.js исправим следующим содержанием.

import React from 'react';
import './App.css';
import Home from './Home';
import { BrowserRouter as Router, Route, Routes } from 'react-router-dom';
import BookList from "./BookList";
import BookEdit from "./BookEdit";


const App = () => {
return (
<Router>
<Routes>
<Route exact path="/" element={<Home/>}/>
<Route path='/books' exact={true} element={<BookList/>}/>
<Route path='/books/:id' element={<BookEdit/>}/>
</Routes>
</Router>
)
}

export default App;
Добавим файл Home.js - это будет наша стартовая страница.

import React from 'react';
import './App.css';
import AppNavbar from './AppNavbar';
import { Link } from 'react-router-dom';
import { Button, Container } from 'reactstrap';

const Home = () => {
return (
<div>
<AppNavbar/>
<Container fluid>
<Button color="link"><Link to="/books">Manage My Books</Link></Button>
</Container>
</div>
);
}

export default Home;
Добавим файл AppNavbar.js

import React, { useState } from 'react';
import { Collapse, Nav, Navbar, NavbarBrand, NavbarToggler} from 'reactstrap';
import { Link } from 'react-router-dom';

const AppNavbar = () => {

    const [isOpen, setIsOpen] = useState(false);

    return (
        <Navbar color="info" dark expand="md">
            <NavbarBrand tag={Link} to="/">Home</NavbarBrand>
            <NavbarToggler onClick={() => { setIsOpen(!isOpen) }}/>
            <Collapse isOpen={isOpen} navbar>
                <Nav className="justify-content-end" style={{width: "100%"}} navbar>

                </Nav>
            </Collapse>
        </Navbar>
    );
};

export default AppNavbar;
Наш фронт готов. Теперь напишем Dockerfile для того, чтобы сделать на его основе образ.

В корне папки frontend создаем файл Dockerfile следующего содержания.

FROM node:18-alpine

WORKDIR /app

EXPOSE 3000

COPY ["package.json", "package-lock.json*", "./"]

RUN npm install

COPY . .

CMD ["npm", "start"]
FROM node:18-alpine - указываем на основании какого образа мы будем запускать наше приложение с фронтом. У нас будет образ внутри которого будет node и npm.

WORKDIR /app - как всегда внутри образа создаем папку app, в которой мы и будем сохранять наше приложение.

EXPOSE 3000 - на каком порту будет работать наш фронт.

COPY ["package.json", "package-lock.json*", "./"] - копируем файлы package.json и package-lock.json в наш образ, содержащие зависимости.

RUN npm install -  команда, устанавливающая пакеты, то есть она скачает пакет в папку проекта node_modules в соответствии с конфигурацией в файле package.json, обновив версию пакета везде, где это возможно (и, в свою очередь, обновив package-lock.json)

COPY . . - копируем исходный код в образ.

CMD ["npm", "start"] - запускаем проект.

Осталось добавить в наш docker-copmose.yml файл, информацию чтобы он автоматически создавал образ с нашим фронтом.

Теперь docker-compose будет выглядеть так.

version: '3.8'
services:
client-frontend:
image: frontend:0.0.1
build: ./frontend
restart: always
ports:
- '3000:3000'
volumes:
- /app/node_modules
- ./frontend:/app

client-backend:
image: client:0.0.1
build:
context: .
dockerfile: Dockerfile
ports:
- "8181:8181"
depends_on:
- service-db
environment:
- SERVER_PORT= 8181
- SPRING_DATASOURCE_URL=jdbc:postgresql://service-db/books_db

service-db:
image: postgres:14.7-alpine
environment:
POSTGRES_USER: username
POSTGRES_PASSWORD: password
ports:
- "15432:5432"
volumes:
- ./infrastructure/db/create_db.sql:/docker-entrypoint-initdb.d/create_db.sql
- db-data:/var/lib/postgresql/data
restart: unless-stopped

pgadmin:
container_name: pgadmin4_container
image: dpage/pgadmin4:7
restart: always
environment:
PGADMIN_DEFAULT_EMAIL: admin@admin.com
PGADMIN_DEFAULT_PASSWORD: root
ports:
- "5050:80"
volumes:
- pgadmin-data:/var/lib/pgadmin

volumes:
db-data:
pgadmin-data:
Мы добавили новый сервис - то есть новый образ на основании которого запустится контейнер.

client-frontend:
image: frontend:0.0.1
build: ./frontend
здесь мы указали, что хотим чтобы docker сбилдил контейнер на основании образа, полученного на основании Dockerfile расположенного в папке frontend.

    ports:
      - '3000:3000'
запускаем наш фронт на порту 3000.

Файл docker-compose полностью готов. Запускаем.

Снова входим в терминал в папку с нашим проектом и запускаем команду docker-compose up


Все наши контейнеры должны работать.


Переходим в браузер и заходим на адрес http://localhost:3000/


Нажимаем на Manage My Books и мы должны получить все наши книги.


Также протестируем добавление книги.



Удаление книги.


И изменение книги



Все работает. Также можете убедиться, что все изменения переносятся в нашу базу данных.


Вот и мы подошли к концу с написанием нашего небольшого приложения по сохранению книг и использовали при этом docker-compose файла.