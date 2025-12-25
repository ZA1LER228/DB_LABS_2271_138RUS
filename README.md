# DB_LABS_2271_138RUS

Лабораторная работа 1

![5257967580620721007 (2)](https://github.com/user-attachments/assets/ada1747c-6a3a-4618-a898-ace551f7eb4d)

Лабораторная работа 2

      INSERT INTO nepokrytov_2271.room (room_id, hotel_id, floor, type_id, price)
      VALUES 
          (1, 1, 1, 1, 1500.00),  -- Одноместный номер в Отель-Солнце
          (2, 1, 1, 2, 2500.00),  -- Двухместный номер в Отель-Солнце
          (3, 1, 2, 3, 4000.00),  -- Люкс в Отель-Солнце
          (4, 2, 1, 1, 1600.00),  -- Одноместный номер в Звездный отдых
          (5, 2, 1, 2, 2700.00),  -- Двухместный номер в Звездный отдых
          (6, 2, 2, 3, 4500.00),  -- Люкс в Звездный отдых
          (7, 3, 1, 1, 1800.00),  -- Одноместный номер в Комфортная жизнь
          (8, 3, 2, 2, 2800.00),  -- Двухместный номер в Комфортная жизнь
          (9, 3, 3, 3, 5000.00);  -- Люкс в Комфортная жизнь


      INSERT INTO nepokrytov_2271.room_type (type_id, class, raiting)
      VALUES 
            (1, 'Одноместный', 4.1),  -- Описание для одноместного номера
            (2, 'Двухместный', 4.3),  -- Описание для двухместного номера
            (3, 'Люкс', 5.0),          -- Описание для люксового номера
            (4, 'Семейный', 4.5),      -- Описание для семейного номера
            (5, 'Апартаменты', 4.8);   -- Описание для апартаментов

      INSERT INTO nepokrytov_2271.booking (booking_id, client_id, room_id, check_in, check_out)
      VALUES 
          (1, 1, 101, '2023-10-01', '2023-10-05'),
          (2, 2, 102, '2023-10-10', '2023-10-15'),
          (3, 1, 103, '2023-10-20', '2023-10-25'),
          (4, 1, 104, '2023-11-20', '2023-11-25');
      

      INSERT INTO nepokrytov_2271.hotel (hotel_id, name, rating, address)
      VALUES 
          (1, 'Гранд Отель Европа', 4.7, 'ул. Невский проспект, д. 1, Санкт-Петербург'),
          (2, 'Москва Реджинси', 4.5, 'ул. Тверская, д. 15, Москва'),
          (3, 'Сочи Плаза', 4.3, 'ул. Орджоникидзе, д. 11, Сочи'),
          (4, 'Казань Палас', 4.6, 'ул. Пушкина, д. 4, Казань'),
          (5, 'Екатеринбург Парк Инн', 4.4, 'ул. 8 Марта, д. 194, Екатеринбург');

      INSERT INTO nepokrytov_2271.client (client_id, name, number, birthday)
      VALUES 
          (1, 'Иванов Александр Сергеевич', '+79111234567', '1985-03-15'),
          (2, 'Петрова Екатерина Владимировна', '+79219876543', '1990-07-22'),
          (3, 'Сидоров Михаил Петрович', '+79031112233', '1978-11-30'),
          (4, 'Козлова Анна Дмитриевна', '+79504445566', '1995-05-18'),
          (5, 'Николаев Денис Олегович', '+79607778899', '1982-09-08'),
          (6, 'Федорова Ольга Игоревна', '+79703334455', '1992-12-25'),
          (7, 'Морозов Павел Андреевич', '+79806667788', '1975-06-14'),
          (8, 'Волкова Марина Сергеевна', '+79909990011', '1988-04-03'),
          (9, 'Алексеев Артем Викторович', '+79152223344', '1998-01-19'),
          (10, 'Семенова Юлия Александровна', '+79265556677', '1993-08-11');

Лабораторная работа 3

      -- Представление: Детальный отчет по бронированиям
      CREATE OR REPLACE VIEW nepokrytov_2271.booking_detailed_report AS
      SELECT 
          b.booking_id,
          c.name AS client_name,
          c.number AS client_phone,
          h.name AS hotel_name,
          r.room_id,
          rt.class AS room_type,
          r.floor,
          r.price,
          b.check_in,
          b.check_out,
          (b.check_out - b.check_in) AS nights_count,
          r.price * (b.check_out - b.check_in) AS total_cost
      FROM nepokrytov_2271.booking b
      JOIN nepokrytov_2271.client c ON b.client_id = c.client_id
      JOIN nepokrytov_2271.room r ON b.room_id = r.room_id
      JOIN nepokrytov_2271.hotel h ON r.hotel_id = h.hotel_id
      JOIN nepokrytov_2271.room_type rt ON r.type_id = rt.type_id
      ORDER BY b.check_in DESC, total_cost DESC;


      -- Функция: Поиск доступных номеров по критериям
      CREATE OR REPLACE FUNCTION nepokrytov_2271.find_available_rooms_by_criteria(
          p_check_in DATE,
          p_check_out DATE,
          p_hotel_id INTEGER DEFAULT NULL,
          p_max_price DECIMAL DEFAULT NULL
      )
      RETURNS TABLE (
          room_id INTEGER,
          hotel_name VARCHAR,
          room_type VARCHAR,
          floor INTEGER,
          price DECIMAL(10,2),
          total_stay_cost DECIMAL(10,2)
      ) 
      LANGUAGE 'plpgsql'
      AS $BODY$
      BEGIN
          RETURN QUERY
          SELECT 
              r.room_id,
              h.name::VARCHAR AS hotel_name,
              rt.class::VARCHAR AS room_type,
              r.floor,
              r.price::DECIMAL(10,2),
              (r.price * (p_check_out - p_check_in))::DECIMAL(10,2) AS total_stay_cost
          FROM nepokrytov_2271.room r
          JOIN nepokrytov_2271.hotel h ON r.hotel_id = h.hotel_id
          JOIN nepokrytov_2271.room_type rt ON r.type_id = rt.type_id
          WHERE nepokrytov_2271.is_room_available(r.room_id, p_check_in, p_check_out) = true
          AND (p_hotel_id IS NULL OR r.hotel_id = p_hotel_id)
          AND (p_max_price IS NULL OR r.price <= p_max_price)
          ORDER BY h.name, r.price ASC;
      END;
      $BODY$;


      -- Использование представления
      SELECT * FROM nepokrytov_2271.booking_detailed_report;

      -- Использование функции с разными параметрами
      SELECT * FROM nepokrytov_2271.find_available_rooms_by_criteria('2024-01-15', '2024-01-20');
      SELECT * FROM nepokrytov_2271.find_available_rooms_by_criteria('2024-01-15', '2024-01-20', 1);
      SELECT * FROM nepokrytov_2271.find_available_rooms_by_criteria('2024-01-15', '2024-01-20', 1, 3000);


Лабораторная работа 4

      -- 1. Генерируем данные
      CALL nepokrytov_2271.generate_test_data();

      -- 2. УДАЛЯЕМ ВСЕ существующие индексы полностью
      DROP INDEX IF EXISTS nepokrytov_2271.idx_booking_checkin_nights;
      DROP INDEX IF EXISTS nepokrytov_2271.idx_room_price_filtered;
      DROP INDEX IF EXISTS nepokrytov_2271.idx_booking_dates;
      DROP INDEX IF EXISTS nepokrytov_2271.idx_booking_room_id;
      DROP INDEX IF EXISTS nepokrytov_2271.idx_room_price_hotel;
      DROP INDEX IF EXISTS nepokrytov_2271.idx_booking_client_id;

      -- 3. Анализ ДО индексов
      SELECT '=== ДО оптимизации (БЕЗ индексов) ===';
      EXPLAIN ANALYZE 
      SELECT 
          b.booking_id,
          c.name as client_name,
          h.name as hotel_name,
          r.price,
          b.check_in,
          b.check_out,
          (b.check_out - b.check_in) as nights,
          (r.price * (b.check_out - b.check_in)) as total_cost
      FROM nepokrytov_2271.booking b
      JOIN nepokrytov_2271.client c ON b.client_id = c.client_id
      JOIN nepokrytov_2271.room r ON b.room_id = r.room_id
      JOIN nepokrytov_2271.hotel h ON r.hotel_id = h.hotel_id
      WHERE b.check_in >= '2024-01-01'
      AND r.price BETWEEN 2000 AND 8000
      AND (b.check_out - b.check_in) >= 3
      ORDER BY total_cost DESC
      LIMIT 50;

      SELECT '=== СОЗДАЁМ ОПТИМАЛЬНЫЕ ИНДЕКСЫ ===';

      -- ИНДЕКС 1: Для быстрого JOIN между booking и room
      CREATE INDEX idx_booking_room_id ON nepokrytov_2271.booking (room_id, check_in);
      -- Это ускорит Hash Join между booking и room

      -- ИНДЕКС 2: Для фильтрации booking по дате и количеству ночей
      CREATE INDEX idx_booking_dates ON nepokrytov_2271.booking (check_in, check_out) 
      WHERE check_in >= '2024-01-01';
      -- Частичный индекс только для нужных дат

      -- ИНДЕКС 3: Для фильтрации room по цене И быстрого JOIN с hotel
      CREATE INDEX idx_room_price_hotel ON nepokrytov_2271.room (price, hotel_id) 
      WHERE price BETWEEN 2000 AND 8000;
      -- Частичный индекс + включает hotel_id для JOIN

      -- ИНДЕКС 4: Для быстрого JOIN с client
      CREATE INDEX idx_booking_client_id ON nepokrytov_2271.booking (client_id, room_id);
      -- Ускоряет Hash Join с client

      -- Обновляем статистику
      ANALYZE nepokrytov_2271.booking;
      ANALYZE nepokrytov_2271.room;
      ANALYZE nepokrytov_2271.client;
      ANALYZE nepokrytov_2271.hotel;

      -- 5. Анализ ПОСЛЕ индексов
      SELECT '=== ПОСЛЕ оптимизации (С индексами) ===';
      EXPLAIN ANALYZE 
      SELECT 
          b.booking_id,
          c.name as client_name,
          h.name as hotel_name,
          r.price,
          b.check_in,
          b.check_out,
          (b.check_out - b.check_in) as nights,
          (r.price * (b.check_out - b.check_in)) as total_cost
      FROM nepokrytov_2271.booking b
      JOIN nepokrytov_2271.client c ON b.client_id = c.client_id
      JOIN nepokrytov_2271.room r ON b.room_id = r.room_id
      JOIN nepokrytov_2271.hotel h ON r.hotel_id = h.hotel_id
      WHERE b.check_in >= '2024-01-01'
      AND r.price BETWEEN 2000 AND 8000
      AND (b.check_out - b.check_in) >= 3
      ORDER BY total_cost DESC
      LIMIT 50;

      -- 6. Оптимизация для Hash Join (если всё ещё медленно)
      SET work_mem = '16MB';  -- Увеличиваем память для хэш-таблиц
      SET enable_nestloop = off;  -- Запрещаем Nested Loop, только Hash Join
      SET enable_mergejoin = off;  -- Запрещаем Merge Join

      -- Запускаем ещё раз
      SELECT '=== С ОПТИМИЗАЦИЕЙ HASH JOIN ===';
      EXPLAIN ANALYZE 
      SELECT 
          b.booking_id,
          c.name as client_name,
          h.name as hotel_name,
          r.price,
          b.check_in,
          b.check_out,
          (b.check_out - b.check_in) as nights,
          (r.price * (b.check_out - b.check_in)) as total_cost
      FROM nepokrytov_2271.booking b
      JOIN nepokrytov_2271.client c ON b.client_id = c.client_id
      JOIN nepokrytov_2271.room r ON b.room_id = r.room_id
      JOIN nepokrytov_2271.hotel h ON r.hotel_id = h.hotel_id
      WHERE b.check_in >= '2024-01-01'
      AND r.price BETWEEN 2000 AND 8000
      AND (b.check_out - b.check_in) >= 3
      ORDER BY total_cost DESC
      LIMIT 50;

      -- 7. Вернём настройки обратно
      RESET work_mem;
      RESET enable_nestloop;
      RESET enable_mergejoin;

Лабораторная работа 5

      -- ТЕСТ КАСКАДНОГО УДАЛЕНИЯ С UPDATE

      -- 1. Очистим журнал
      DELETE FROM nepokrytov_2271.audit_log;

      -- 2. Создадим тестового клиента
      INSERT INTO nepokrytov_2271.client (name, number, birthday) 
      VALUES ('ТестКлиентКаскад', '111-222-333', '1995-05-15')
      RETURNING client_id as "ID клиента";

      -- 3. Создадим бронирование
      INSERT INTO nepokrytov_2271.booking (client_id, room_id, check_in, check_out)
      SELECT 
          (SELECT client_id FROM nepokrytov_2271.client WHERE name = 'ТестКлиентКаскад'),
          room_id,
          CURRENT_DATE,
          CURRENT_DATE + 2
      FROM nepokrytov_2271.room 
      LIMIT 1
      RETURNING booking_id as "ID бронирования";

      -- 4. Обновим клиента (UPDATE)
      UPDATE nepokrytov_2271.client 
      SET number = '999-888-777'
      WHERE name = 'ТестКлиентКаскад'
      RETURNING client_id, name, number;

      -- 5. Удалим клиента (каскадное удаление бронирований)
      DELETE FROM nepokrytov_2271.client 
      WHERE name = 'ТестКлиентКаскад';

      -- 6. Посмотрим журнал аудита
      SELECT '=== ЖУРНАЛ АУДИТА ===' as test;
      SELECT 
          id,
          table_name,
          operation,
          record_id,
          TO_CHAR(change_time, 'HH24:MI:SS') as время
      FROM nepokrytov_2271.audit_log 
      ORDER BY id;

      -- 7. Проверим цепочку операций с временем
      SELECT '=== ЦЕПОЧКА ОПЕРАЦИЙ С ВРЕМЕНЕМ ===' as test;
      SELECT 
          id,
          CASE 
              WHEN operation = 'INSERT' AND table_name = 'client' THEN '1. Создан клиент'
              WHEN operation = 'UPDATE' AND table_name = 'client' THEN '2. Обновлён клиент'
              WHEN operation = 'CASCADE' AND table_name = 'client' THEN '3. Каскадное удаление (бронирования)'
              WHEN operation = 'DELETE' AND table_name = 'client' THEN '4. Удалён клиент'
              ELSE table_name || ' - ' || operation
          END as "Операция",
          'ID: ' || record_id as "ID записи",
          TO_CHAR(change_time, 'DD.MM.YYYY HH24:MI:SS') as "Дата и время",
          TO_CHAR(change_time, 'HH24:MI:SS:MS') as "Точное время (до миллисекунд)"
      FROM nepokrytov_2271.audit_log 
      ORDER BY id;
