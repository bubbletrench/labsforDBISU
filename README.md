# Лабораторные работы
Лабораторные работы по Базам Данных Медведев Владислав 02261-ДБ
Вариант 29. Покупка билетов на поезда
## Вариант 29. Покупка билетов на поезда. Постановка задачи
Условие задачи:

Имеются поезда (номер, год выпуска, количество вагонов, тип) и расписание движения (дата и время отправления/прибытия, станции). Накапливаются сведения о проданных билетах с указанием пассажиров, даты продажи, стоимости и информации о местах.

Сущности:

    Поезда (train_id, номер_поезда, год_выпуска, количество_вагонов, тип_поезда)

    Расписание (schedule_id, номер_поезда, дата_отправления, время_отправления, место_отправления, дата_прибытия, время_прибытия, место_прибытия)

    Билеты (ticket_id, номер_рейса, дата_продажи, время_продажи, ФИО_пассажира, паспортные_данные, номер_поезда, количество_билетов, наличие_льгот, вид_вагона, номер_вагона, номер_места, стоимость)

    Пассажиры (passenger_id, ФИО_пассажира, паспортные_данные, тип_льготы)

Процессы: (Отношения)

При покупке билета обеспечивается доступ к расписанию для выбора подходящего поезда. Накапливается информация о проданных билетах с детализацией по пассажирам, местам и стоимости.

Выходные документы:

    Расписание поездов с определенными местами отправления и прибытия и с полной информацией о поездах

    Список проданных билетов за определенный интервал времени с указанием их стоимости, упорядоченный по датам, а внутри одной даты – по номерам вагонов

## Проверка на 3НФ.
Анализ на соответствие 3НФ (Третьей нормальной форме)

3НФ требует:

    Соответствие 2НФ (все неключевые атрибуты зависят от всего первичного ключа)

    Отсутствие транзитивных зависимостей (неключевые атрибуты не должны зависеть от других неключевых атрибутов)

Проверка вашей модели:

✅ Поезд

    номер_поезда → год_выпуска, тип_поезда, кол-во_вагонов

    Соответствует 3НФ - все атрибуты зависят от первичного ключа

✅ Расписание

    айди_рейса → номер_поезда, дата_отправления, время_отправления, etc.

    Соответствует 3НФ - все атрибуты зависят от первичного ключа

❌ Билет - НЕ СООТВЕТСТВУЕТ 3НФ

    Проблемы:

        наличие_льгот зависит от пассажира, а не от билета

        вид_вагона может зависеть от номер_вагона и поезда

        стоимость может зависеть от вида_вагона и наличие_льгот

✅ Пассажир

    айди_пассажира → фамилия_пассажира, имя_пассажира, отчество_пассажира, паспортные_данные

    Соответствует 3НФ

🎯 Улучшенная модель, соответствующая 3НФ
```sql
-- 1. Поезда (без изменений)
CREATE TABLE trains (
    train_id VARCHAR(10) PRIMARY KEY,
    manufacture_year INTEGER NOT NULL,
    train_type VARCHAR(20) NOT NULL,
    carriage_count INTEGER NOT NULL
);

-- 2. Расписание (без изменений)
CREATE TABLE schedules (
    schedule_id SERIAL PRIMARY KEY,
    train_id VARCHAR(10) NOT NULL REFERENCES trains(train_id),
    departure_date DATE NOT NULL,
    departure_time TIME NOT NULL,
    departure_station VARCHAR(100) NOT NULL,
    arrival_date DATE NOT NULL,
    arrival_time TIME NOT NULL,
    arrival_station VARCHAR(100) NOT NULL
);

-- 3. Пассажиры (без изменений)
CREATE TABLE passengers (
    passenger_id SERIAL PRIMARY KEY,
    last_name VARCHAR(100) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    middle_name VARCHAR(100),
    passport_data VARCHAR(50) UNIQUE NOT NULL,
    benefit_type VARCHAR(30) -- Вынесли льготы сюда!
);

-- 4. NEW: Типы вагонов и цены
CREATE TABLE carriage_pricing (
    carriage_type_id SERIAL PRIMARY KEY,
    carriage_type VARCHAR(20) NOT NULL UNIQUE, -- 'купейный', 'плацкартный', etc.
    base_price DECIMAL(10,2) NOT NULL
);

-- 5. NEW: Вагоны в поездах
CREATE TABLE train_carriages (
    train_id VARCHAR(10) REFERENCES trains(train_id),
    carriage_number INTEGER NOT NULL,
    carriage_type_id INTEGER REFERENCES carriage_pricing(carriage_type_id),
    PRIMARY KEY (train_id, carriage_number)
);

-- 6. Улучшенная таблица билетов
CREATE TABLE tickets (
    ticket_id SERIAL PRIMARY KEY,
    schedule_id INTEGER NOT NULL REFERENCES schedules(schedule_id),
    passenger_id INTEGER NOT NULL REFERENCES passengers(passenger_id),
    train_id VARCHAR(10) NOT NULL,
    carriage_number INTEGER NOT NULL,
    seat_number INTEGER NOT NULL,
    sale_datetime TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    final_price DECIMAL(10,2) NOT NULL,
    
    -- Внешний ключ на состав поезда
    FOREIGN KEY (train_id, carriage_number) 
    REFERENCES train_carriages(train_id, carriage_number),
    
    -- Уникальность места
    UNIQUE (schedule_id, carriage_number, seat_number)
);
```

# Лабораторная работа 1.
## ER-диаграмма
<img width="796" height="669" alt="diagramm" src="https://github.com/user-attachments/assets/0795a525-8b07-4a45-a465-ce1eedd9b4db" />

# Лабораторная работа 2.
## Логическая модель по диаграмме

1. Сущность: Поезд
  номер_поезда (PK) — уникальный идентификатор поезда
  год_выпуска — год выпуска поезда
  тип_поезда — тип поезда (пассажирский, скорый, экспресс и т. д.)
  кол-во_вагонов — количество вагонов в составе
2. Сущность: Расписание
  айди_рейса (PK) — уникальный идентификатор рейса
  номер_поезда (FK) — внешний ключ, ссылается на Поезд(номер_поезда)
  дата_отправления — дата отправления поезда
  время_отправления — время отправления
  место_отправления — станция отправления
  дата_прибытия — дата прибытия поезда
  время_прибытия — время прибытия
  место_прибытия — станция прибытия
3. Сущность: Билет
  номер_билета (PK) — уникальный номер билета
  айди_рейса (FK) — внешний ключ, ссылается на Расписание(айди_рейса)
  айди_пассажира (FK) — внешний ключ, ссылается на Пассажир(айди_пассажира)
  дата_продажи — дата покупки билета
  время_продажи — время покупки билета
  кол-во_билетов — количество купленных билетов
  наличие_льгот — наличие льгот (true/false)
  вид_вагона — тип вагона (купе, плацкарт, СВ и т. д.)
  номер_вагона — номер вагона
  номер_места — номер места
  стоимость — стоимость билета
4. Сущность: Пассажир
  айди_пассажира (PK) — уникальный идентификатор пассажира
  фамилия_пассажира — фамилия пассажира
  имя_пассажира — имя пассажира
  отчество_пассажира — отчество пассажира
  паспортные_данные — серия и номер паспорта

## Физическая модель
```sql
-- Типы поездов
CREATE TABLE train_types (
    type_id INTEGER PRIMARY KEY,
    type_name VARCHAR(20) NOT NULL UNIQUE
);

-- Типы вагонов
CREATE TABLE carriage_types (
    carriage_type_id INTEGER PRIMARY KEY,
    type_name VARCHAR(20) NOT NULL UNIQUE,
    base_price DECIMAL(10,2) NOT NULL CHECK (base_price >= 0)
);

-- Типы льгот
CREATE TABLE benefit_types (
    benefit_id INTEGER PRIMARY KEY,
    benefit_name VARCHAR(20) NOT NULL UNIQUE,
    discount_percent INTEGER NOT NULL CHECK (discount_percent >= 0 AND discount_percent <= 100)
);

-- Поезда
CREATE TABLE trains (
    train_id VARCHAR(10) PRIMARY KEY,
    manufacture_year INTEGER NOT NULL CHECK (manufacture_year > 1900 AND manufacture_year <= EXTRACT(YEAR FROM CURRENT_DATE)),
    carriage_count INTEGER NOT NULL CHECK (carriage_count > 0 AND carriage_count <= 20),
    type_id INTEGER NOT NULL,
    
    FOREIGN KEY (type_id) REFERENCES train_types(type_id) ON DELETE RESTRICT
);

-- Пассажиры
CREATE TABLE passengers (
    passenger_id SERIAL PRIMARY KEY,
    last_name VARCHAR(50) NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    middle_name VARCHAR(50),
    passport_series VARCHAR(4) NOT NULL,
    passport_number VARCHAR(6) NOT NULL,
    benefit_id INTEGER,
    
    FOREIGN KEY (benefit_id) REFERENCES benefit_types(benefit_id) ON DELETE SET NULL,
    UNIQUE (passport_series, passport_number)
);

-- Расписание
CREATE TABLE schedules (
    schedule_id SERIAL PRIMARY KEY,
    train_id VARCHAR(10) NOT NULL,
    departure_date DATE NOT NULL,
    departure_time TIME NOT NULL,
    departure_station VARCHAR(100) NOT NULL,
    arrival_date DATE NOT NULL,
    arrival_time TIME NOT NULL,
    arrival_station VARCHAR(100) NOT NULL,
    
    FOREIGN KEY (train_id) REFERENCES trains(train_id) ON DELETE CASCADE,
    CHECK (arrival_date >= departure_date),
    UNIQUE (train_id, departure_date, departure_time)
);

-- Билеты
CREATE TABLE tickets (
    ticket_id SERIAL PRIMARY KEY,
    schedule_id INTEGER NOT NULL,
    passenger_id INTEGER NOT NULL,
    sale_datetime TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    ticket_count INTEGER NOT NULL DEFAULT 1 CHECK (ticket_count > 0 AND ticket_count <= 4),
    carriage_type_id INTEGER NOT NULL,
    carriage_number INTEGER NOT NULL CHECK (carriage_number > 0),
    seat_number INTEGER NOT NULL CHECK (seat_number > 0),
    base_price DECIMAL(10,2) NOT NULL CHECK (base_price >= 0),
    final_price DECIMAL(10,2) NOT NULL CHECK (final_price >= 0),
    
    FOREIGN KEY (schedule_id) REFERENCES schedules(schedule_id) ON DELETE CASCADE,
    FOREIGN KEY (passenger_id) REFERENCES passengers(passenger_id) ON DELETE CASCADE,
    FOREIGN KEY (carriage_type_id) REFERENCES carriage_types(carriage_type_id) ON DELETE RESTRICT,
    UNIQUE (schedule_id, carriage_number, seat_number)
);

-- Заполнение справочников базовыми данными
INSERT INTO train_types (type_id, type_name) VALUES
(1, 'общий'),
(2, 'скоростной'),
(3, 'высокоскоростной');

INSERT INTO carriage_types (carriage_type_id, type_name, base_price) VALUES
(1, 'общий', 1000.00),
(2, 'плацкартный', 2000.00),
(3, 'купейный', 3500.00);

INSERT INTO benefit_types (benefit_id, benefit_name, discount_percent) VALUES
(1, 'пенсионер', 30),
(2, 'ребенок-сирота', 50),
(3, 'инвалид', 40);

-- Создание индексов для оптимизации запросов
CREATE INDEX idx_schedules_departure ON schedules(departure_station, departure_date);
CREATE INDEX idx_schedules_arrival ON schedules(arrival_station, arrival_date);
CREATE INDEX idx_tickets_sale_date ON tickets(sale_datetime);
CREATE INDEX idx_tickets_passenger ON tickets(passenger_id);
CREATE INDEX idx_passengers_name ON passengers(last_name, first_name);
CREATE INDEX idx_passengers_passport ON passengers(passport_series, passport_number);
CREATE INDEX idx_trains_type ON trains(type_id);
```
## DDL-запросы

Таблица train_types
<img width="807" height="492" alt="Screenshot 2025-12-01 210337" src="https://github.com/user-attachments/assets/5963f6f6-f70f-4326-9994-08de4010d00c" />
Таблица carriage_types
<img width="807" height="492" alt="Screenshot 2025-12-01 210337" src="https://github.com/user-attachments/assets/edfd8ea9-5f0e-4a33-a8a8-84bd86e7a5c6" />
Таблица benefit_types
<img width="807" height="492" alt="Screenshot 2025-12-01 210337" src="https://github.com/user-attachments/assets/52885e6a-cd60-4dc6-a70c-88777855482d" />
Таблица trains
<img width="894" height="547" alt="image" src="https://github.com/user-attachments/assets/8d805674-3bc3-4d82-99d3-f2e6b1800699" />
Таблица passengers
<img width="905" height="550" alt="image" src="https://github.com/user-attachments/assets/ccd1ad71-8b51-4a79-b59b-e8149725922b" />
Таблица schedules
<img width="902" height="546" alt="image" src="https://github.com/user-attachments/assets/ef01609d-6305-4ea1-ada1-d27cd546e644" />
Таблица tickets
<img width="902" height="546" alt="image" src="https://github.com/user-attachments/assets/b05ff342-7750-4851-b497-e05081ce6dc2" />

### Таблица заполненная данными

Таблица train_types
<img width="680" height="309" alt="image" src="https://github.com/user-attachments/assets/742a84f9-c6ce-4fb8-af06-2ec256b9f46f" />
<img width="531" height="447" alt="image" src="https://github.com/user-attachments/assets/3489270e-5ca8-466e-8ea2-0529b504f97a" />
Таблица carriage_types
<img width="806" height="322" alt="image" src="https://github.com/user-attachments/assets/71b6a8f1-bd99-462b-8870-4ab2d41a19dd" />
<img width="452" height="459" alt="image" src="https://github.com/user-attachments/assets/92da4ff8-c4a1-4d5f-9c91-c5bf0e796061" />
Таблица benefit_types
<img width="825" height="337" alt="image" src="https://github.com/user-attachments/assets/967b01ed-5250-4c32-882b-6f8afbed5e95" />
<img width="478" height="448" alt="image" src="https://github.com/user-attachments/assets/dba75204-a4e2-4aae-bac8-38b7e0228ed3" />
Таблица trains
<img width="841" height="345" alt="image" src="https://github.com/user-attachments/assets/fdb8806b-f1bc-4622-875e-303f5e8e6e6b" />
<img width="549" height="453" alt="image" src="https://github.com/user-attachments/assets/76756bd3-f9aa-4a9e-a126-62874221652a" />
Таблица passengers
<img width="1231" height="309" alt="image" src="https://github.com/user-attachments/assets/d295692b-1dd7-42ae-b953-6afb1f0ee107" />
<img width="818" height="466" alt="image" src="https://github.com/user-attachments/assets/2aa25b06-985b-41b0-8b6a-20f28f3d1bb9" />
Таблица schedules
<img width="1442" height="269" alt="image" src="https://github.com/user-attachments/assets/deb069da-3c0c-4fd2-b209-1a1584fc9aee" />
<img width="1101" height="471" alt="image" src="https://github.com/user-attachments/assets/f7d23c0d-c5e8-4199-9b68-2daf428e72b9" />
Таблица tickets
<img width="1048" height="383" alt="image" src="https://github.com/user-attachments/assets/53fb8471-0286-4de0-997a-bb11a2e2d7f6" />
<img width="1225" height="558" alt="image" src="https://github.com/user-attachments/assets/4329dfcc-14ef-4f1d-ac01-8c0027d7d89e" />

## SELECT-запросы с JOIN
### Для документа 1 (Расписание):
Используются JOIN 3 таблиц: schedules, trains, train_types
Фильтрация по станциям отправления/прибытия
Вывод полной информации о поездах
Расчет длительности поездки

<img width="1497" height="685" alt="image" src="https://github.com/user-attachments/assets/9b6ac0e8-6316-4cf3-b7f9-d236b765b4d1" />

### Для документа 2 (Билеты):
Используются JOIN 4-5 таблиц с LEFT JOIN для льгот
Фильтрация по интервалу времени продажи
Сортировка по датам → номерам вагонов → местам
Расчет скидок и агрегация данных

<img width="1029" height="724" alt="image" src="https://github.com/user-attachments/assets/f9eeb346-008d-456c-a7c2-03b2992c4eb9" />

# Лабораторная работа 3.
## Представления для выходных документов
### Первый выходной документ. Расписание поездов.
<img width="510" height="481" alt="Screenshot 2025-12-16 163107" src="https://github.com/user-attachments/assets/727ab392-222d-4c17-b299-d6ba28124a91" />
Проверка работы
<img width="1166" height="566" alt="Screenshot 2025-12-16 163148" src="https://github.com/user-attachments/assets/d1ab60b3-2526-4214-8e31-c5d1754ba53a" />

### Второй выходной документ. Список проданных билетов
<img width="664" height="476" alt="Screenshot 2025-12-16 163225" src="https://github.com/user-attachments/assets/20c9b9cb-a00f-4290-bf8a-d75d391dcd34" />
Проверка работы
<img width="1107" height="566" alt="Screenshot 2025-12-16 163253" src="https://github.com/user-attachments/assets/880fecd8-3fac-4bf3-9ad6-b58fc2c5ea31" />

## Процедуры с параметрами

Ежедневная выручка с билетов
<img width="605" height="690" alt="Screenshot 2025-12-16 164736" src="https://github.com/user-attachments/assets/903b2223-023f-4af0-9396-785734683750" />
<img width="544" height="533" alt="Screenshot 2025-12-16 164818" src="https://github.com/user-attachments/assets/b7746ade-2d58-4b2a-81f4-4614673b68e9" />

Добавление пассажиров
<img width="760" height="729" alt="image" src="https://github.com/user-attachments/assets/ac0e7cea-e38a-42b1-9a05-cee9657b2a37" />
<img width="920" height="445" alt="image" src="https://github.com/user-attachments/assets/97a73b0f-a8bc-4389-bccd-2f5542eb7d2c" />

Обновление пассажира
<img width="659" height="702" alt="image" src="https://github.com/user-attachments/assets/1ed9a36e-24b7-4bf4-9ec0-338db7ba9f29" />
<img width="905" height="408" alt="image" src="https://github.com/user-attachments/assets/97610bc9-a769-4f6b-b6af-935bb8786098" />

Удаление пассажира
<img width="733" height="461" alt="image" src="https://github.com/user-attachments/assets/a86ebb90-64fd-4e43-a89c-04dd49efa820" />
<img width="919" height="260" alt="image" src="https://github.com/user-attachments/assets/08a2b840-d994-4de8-a272-5368e43986dd" />
Примечание: ничего не вывелось, потому что мы удалили пассажира

## Сложные запросы при потощи представления.

Популярность маршрутов
<img width="725" height="576" alt="image" src="https://github.com/user-attachments/assets/cfa9165f-26ee-4e4f-b376-4515a1bb6a16" />
<img width="1488" height="264" alt="image" src="https://github.com/user-attachments/assets/1d20c112-baf3-4623-9622-7a4a2250b082" />

Аналитика по поездам и пассажирам
<img width="682" height="580" alt="image" src="https://github.com/user-attachments/assets/7a0d9b97-9b73-4295-978c-f89ac9a11cef" />
<img width="1508" height="255" alt="image" src="https://github.com/user-attachments/assets/29cd87c0-c692-4df2-abf5-e4d86b0a2c17" />
