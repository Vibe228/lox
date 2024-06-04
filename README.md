1 файл первая часть задания которую можно сделать в одном задании
drop procedure if exists createUsersAndDatabases; 
 
delimiter $$ 
create procedure createUsersAndDatabases() 
begin 
declare id int default 1; 
create database if not exists BaseAll; 
create table if not exists BaseAll.Users( 
ID int auto_increment primary key, 
userName tinytext not null, 
userPass blob not null
); 
 
while id <= 9 do 
set @user = concat("d", id); 
set @password = substring(md5(rand()), 1, 8); 
set @database = concat("Base", id); 
 
set @sql_user = concat("create user if not exists ", @user, " identified by '", @password, "';"); 
prepare stmt from @sql_user; 
execute stmt; 
set @sql_database = concat("create database if not exists ", @database, ";"); 
prepare stmt from @sql_database; 
execute stmt; 
set @sql_grant = concat("grant all privileges on ", @database, ".* to ", @user, ";"); 
prepare stmt from @sql_grant; 
execute stmt; 
 
set @crypt_pass = aes_encrypt(@password, '05'); 
insert into BaseAll.Users(userName, userPass) values (@user, @crypt_pass); 
 
set id = id + 1; 
end while; 
end$$ 
delimiter ; 
 
call createUsersAndDatabases();

2файл декриптизация пароля

drop procedure if exists usersDecryptedPasses; 

delimiter $$
create procedure usersDecryptedPasses()
begin
  select ID, userName, cast(aes_decrypt(userPass, '05') as char) as userPass
  from BaseAll.Users;
end$$
delimiter ;

call usersDecryptedPasses();

3 файл создание базы данных по описанию

create database if not exists aboba; 

CREATE TABLE `clients` (
  `id` int PRIMARY KEY NOT NULL AUTO_INCREMENT,
  `last_name` varchar(255) NOT NULL,
  `first_name` varchar(255) NOT NULL,
  `middle_name` varchar(255),
  `birthdate` date NOT NULL,
  `fk_documents` int NOT NULL,
  `phone` varchar(20) UNIQUE NOT NULL,
  `email` varchar(255)
);

CREATE TABLE `documents` (
  `id` int PRIMARY KEY NOT NULL AUTO_INCREMENT,
  `series` int NOT NULL,
  `nomber` int UNIQUE NOT NULL,
  `issue_date` date NOT NULL,
  `issued` varchar(255) NOT NULL
);

CREATE TABLE `rooms` (
  `id` int PRIMARY KEY NOT NULL AUTO_INCREMENT,
  `nomber` varchar(255) NOT NULL,
  `class` varchar(255) NOT NULL,
  `price` float NOT NULL
);

CREATE TABLE `staff` (
  `GUID` int PRIMARY KEY NOT NULL AUTO_INCREMENT,
  `last_name` varchar(255) NOT NULL,
  `first_name` varchar(255) NOT NULL,
  `middle_name` varchar(255),
  `birthdate` date NOT NULL,
  `phone` varchar(20) UNIQUE NOT NULL,
  `address` varchar(255) NOT NULL
);

CREATE TABLE `dop_services` (
  `id` int PRIMARY KEY NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL,
  `price` float
);

CREATE TABLE `booking` (
  `id` int PRIMARY KEY NOT NULL AUTO_INCREMENT,
  `in_date` date NOT NULL,
  `out_date` date NOT NULL,
  `fk_client` int NOT NULL,
  `fk_room` int NOT NULL,
  `payment` varchar(255) NOT NULL,
  `fk_dop` int,
  `fk_staff` int
);

CREATE TABLE `HistoryCost` (
  `id` int PRIMARY KEY NOT NULL AUTO_INCREMENT,
  `change_date` date NOT NULL,
  `fk_dop` int NOT NULL,
  `old_price` float NOT NULL,
  `new_price` float NOT NULL
);

ALTER TABLE `clients` ADD FOREIGN KEY (`fk_documents`) REFERENCES `documents` (`id`);

ALTER TABLE `HistoryCost` ADD FOREIGN KEY (`fk_dop`) REFERENCES `dop_services` (`id`);

ALTER TABLE `booking` ADD FOREIGN KEY (`fk_dop`) REFERENCES `dop_services` (`id`);

ALTER TABLE `booking` ADD FOREIGN KEY (`fk_staff`) REFERENCES `staff` (`GUID`);

ALTER TABLE `booking` ADD FOREIGN KEY (`fk_room`) REFERENCES `rooms` (`id`);

ALTER TABLE `booking` ADD FOREIGN KEY (`fk_client`) REFERENCES `clients` (`id`);

4 файл заполнение таблиц рандомными данными

USE aboba;
-- Заполнение таблицы documents случайными данными
INSERT INTO documents (series, nomber, issue_date, issued)
SELECT 
    FLOOR(RAND() * 9000) + 1000,
    FLOOR(RAND() * 900000) + 100000,
    DATE_ADD('2000-01-01', INTERVAL FLOOR(RAND() * 7300) DAY),
    CONCAT('UFMS ', CHAR(FLOOR(RAND() * 26) + 65), CHAR(FLOOR(RAND() * 26) + 65))
FROM 
    (SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5) t;

-- Заполнение таблицы clients случайными данными
INSERT INTO clients (last_name, first_name, middle_name, birthdate, fk_documents, phone, email)
SELECT 
    CONCAT(CHAR(FLOOR(RAND() * 26) + 65), CHAR(FLOOR(RAND() * 26) + 65), CHAR(FLOOR(RAND() * 26) + 65)),
    CONCAT(CHAR(FLOOR(RAND() * 26) + 65), CHAR(FLOOR(RAND() * 26) + 65), CHAR(FLOOR(RAND() * 26) + 65)),
    CONCAT(CHAR(FLOOR(RAND() * 26) + 65), CHAR(FLOOR(RAND() * 26) + 65), CHAR(FLOOR(RAND() * 26) + 65)),
    DATE_ADD('1980-01-01', INTERVAL FLOOR(RAND() * 14600) DAY),
    FLOOR(RAND() * 5) + 1,
    CONCAT('7', FLOOR(RAND() * 9000000000) + 1000000000),
    CONCAT(CHAR(FLOOR(RAND() * 26) + 97), CHAR(FLOOR(RAND() * 26) + 97), CHAR(FLOOR(RAND() * 26) + 97), '@example.com')
FROM 
    (SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5) t;

-- Заполнение таблицы rooms случайными данными
INSERT INTO rooms (nomber, class, price)
SELECT 
    CONCAT('Room ', FLOOR(RAND() * 100) + 1),
    CASE WHEN FLOOR(RAND() * 5) = 0 THEN 'Economy' 
         WHEN FLOOR(RAND() * 5) = 1 THEN 'Standard' 
         WHEN FLOOR(RAND() * 5) = 2 THEN 'Junior Suite' 
         WHEN FLOOR(RAND() * 5) = 3 THEN 'Suite' 
         ELSE 'Presidential' END,
    FLOOR(RAND() * 10000) / 100
FROM 
    (SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5) t;

-- Заполнение таблицы staff случайными данными
INSERT INTO staff (last_name, first_name, middle_name, birthdate, phone, address)
SELECT 
    CONCAT(CHAR(FLOOR(RAND() * 26) + 65), CHAR(FLOOR(RAND() * 26) + 65), CHAR(FLOOR(RAND() * 26) + 65)),
    CONCAT(CHAR(FLOOR(RAND() * 26) + 65), CHAR(FLOOR(RAND() * 26) + 65), CHAR(FLOOR(RAND() * 26) + 65)),
    CONCAT(CHAR(FLOOR(RAND() * 26) + 65), CHAR(FLOOR(RAND() * 26) + 65), CHAR(FLOOR(RAND() * 26) + 65)),
    DATE_ADD('1980-01-01', INTERVAL FLOOR(RAND() * 14600) DAY),
    CONCAT('7', FLOOR(RAND() * 9000000000) + 1000000000),
    CONCAT('Street ', FLOOR(RAND() * 100) + 1, ', City ', CHAR(FLOOR(RAND() * 26) + 65))
FROM 
    (SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5) t;

-- Заполнение таблицы dop_services случайными данными
INSERT INTO dop_services (name, price)
SELECT 
    CONCAT('Service ', CHAR(FLOOR(RAND() * 26) + 65), CHAR(FLOOR(RAND() * 26) + 65)),
    FLOOR(RAND() * 10000) / 100
FROM 
    (SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5) t;

-- Заполнение таблицы booking случайными данными
INSERT INTO booking (in_date, out_date, fk_client, fk_room, payment, fk_dop, fk_staff)
SELECT 
	CURDATE() - INTERVAL FLOOR(RAND() * 30) DAY AS in_date,
    CURDATE() + INTERVAL FLOOR(RAND() * 30 + 1) DAY AS out_date,
    FLOOR(RAND() * 5) + 1,
    FLOOR(RAND() * 5) + 1,
    CASE WHEN FLOOR(RAND() * 3) = 0 THEN 'Credit Card' 
         WHEN FLOOR(RAND() * 3) = 1 THEN 'Cash' 
         ELSE 'Online Payment' END,
    FLOOR(RAND() * 5) + 1,
    FLOOR(RAND() * 5) + 1
FROM 
    (SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5) t;

-- Заполнение таблицы historycost случайными данными
INSERT INTO historycost (change_date, fk_dop, old_price, new_price)
SELECT 
    DATE_ADD('2020-01-01', INTERVAL FLOOR(RAND() * 730) DAY),
    FLOOR(RAND() * 5) + 1,
    FLOOR(RAND() * 10000) / 100,
    FLOOR(RAND() * 10000) / 100
FROM 
    (SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5) t;

5 файл проверка эмайлов

DROP PROCEDURE IF EXISTS ValidateEmails;

DELIMITER //

CREATE PROCEDURE ValidateEmails()
BEGIN
    DECLARE done INT DEFAULT 0;
    DECLARE email_address VARCHAR(255);
    DECLARE email_cursor CURSOR FOR SELECT email FROM clients;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

    -- Удаление временной таблицы, если она существует
    DROP TEMPORARY TABLE IF EXISTS EmailValidationResults;

    -- Создание временной таблицы для хранения результатов проверки
    CREATE TEMPORARY TABLE EmailValidationResults (
        email VARCHAR(255),
        is_valid TINYINT(1)
    );

    -- Открытие курсора
    OPEN email_cursor;

    read_loop: LOOP
        FETCH email_cursor INTO email_address;
        IF done THEN
            LEAVE read_loop;
        END IF;

        -- Проверка корректности email
        IF email_address REGEXP '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$' AND
           email_address NOT REGEXP '[\"\'<>]' THEN
            INSERT INTO EmailValidationResults (email, is_valid) VALUES (email_address, 1);
        ELSE
            INSERT INTO EmailValidationResults (email, is_valid) VALUES (email_address, 0);
        END IF;
    END LOOP;

    -- Закрытие курсора
    CLOSE email_cursor;

    -- Вывод результатов проверки
    SELECT * FROM EmailValidationResults;

    -- Очистка временной таблицы
    DROP TEMPORARY TABLE IF EXISTS EmailValidationResults;
END //

DELIMITER ;

6 файл триггер на изменение цен с добавлением в хистори кост

drop trigger if exists before_dop_services_update;
DELIMITER //

CREATE TRIGGER before_dop_services_update
BEFORE UPDATE ON dop_services
FOR EACH ROW
BEGIN
    IF OLD.price != NEW.price THEN
        INSERT INTO HistoryCost (change_date, fk_dop, old_price, new_price)
        VALUES (now(),OLD.id, OLD.price, NEW.price);
    END IF;
END //

DELIMITER ;

7 файл изменение цен для триггера

SELECT * FROM aboba.dop_services;
INSERT INTO dop_services (name, price)
VALUES 
('Pool', 20.00),
('Gym', 15.00),
('Spa', 30.00);
SET SQL_SAFE_UPDATES = 0;

UPDATE dop_services
SET price = 25.00
WHERE name = 'Pool';

UPDATE dop_services
SET price = 17.50
WHERE name = 'Gym';

UPDATE dop_services
SET price = 35.00
WHERE name = 'Spa';

8 файл 2 модуль 1 запрос

SELECT 
    c.last_name,
    c.first_name,
    c.middle_name,
    b.in_date,
    b.out_date,
    DATEDIFF(b.out_date, b.in_date) AS days,
    r.price AS room_price_per_day,
    ds.price AS service_cost,
    (DATEDIFF(b.out_date, b.in_date) * r.price + ds.price) AS total_cost
FROM 
    booking b
JOIN 
    clients c ON b.fk_client = c.id
JOIN 
    rooms r ON b.fk_room = r.id
LEFT JOIN 
    dop_services ds ON b.fk_dop = ds.id;

9 файл 2 модуль 2 запрос (нужно удалить записи из history cost без этого не сработает)

DELETE FROM historycost
WHERE fk_dop NOT IN (
    SELECT DISTINCT fk_dop
    FROM booking
    WHERE fk_dop IS NOT NULL
);

DELETE FROM dop_services
WHERE id NOT IN (
    SELECT DISTINCT fk_dop
    FROM booking
    WHERE fk_dop IS NOT NULL
);

9 файл 2 модуль 3 запрос

SELECT 
    fk_room,
    COUNT(*) AS booking_count
FROM 
    booking
GROUP BY 
    fk_room
ORDER BY 
    booking_count DESC
LIMIT 1;
UPDATE rooms
SET price = price * 1.15
WHERE id = (
    SELECT 
        fk_room
    FROM 
        booking
    GROUP BY 
        fk_room
    ORDER BY 
        COUNT(*) DESC
    LIMIT 1
);
