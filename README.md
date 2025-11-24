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
