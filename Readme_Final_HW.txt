1_1
CREATE SCHEMA pandemic;
USE pandemic;

1_2
USE pandemic;
SHOW TABLES;
SELECT * FROM infectious_cases LIMIT 10;

2_1
Створюємо таблицю locations

CREATE TABLE pandemic.locations (
  location_id INT AUTO_INCREMENT PRIMARY KEY,
  entity VARCHAR(100),
  code VARCHAR(10)
);

2_2
Додаємо унікальні країни

INSERT INTO pandemic.locations (entity, code)
SELECT DISTINCT Entity, Code
FROM pandemic.infectious_cases;

2_3
Створюємо нову нормалізовану таблицю

CREATE TABLE pandemic.infectious_cases_normalized (
  id INT AUTO_INCREMENT PRIMARY KEY,
  location_id INT,
  year INT,
  number_yaws INT,
  polio_cases INT,
  cases_guinea_worm INT,
  number_rabies INT,
  number_malaria INT,
  FOREIGN KEY (location_id) REFERENCES pandemic.locations(location_id)
);

2_4
Заповнюємо нормалізовану таблицю

INSERT INTO pandemic.infectious_cases_normalized (
  location_id,
  year,
  number_yaws,
  polio_cases,
  cases_guinea_worm,
  number_rabies,
  number_malaria
)
SELECT 
  l.location_id,
  i.Year,
  NULLIF(CAST(i.Number_yaws AS CHAR), ''),
  NULLIF(CAST(i.polio_cases AS CHAR), ''),
  NULLIF(CAST(i.cases_guinea_worm AS CHAR), ''),
  NULLIF(CAST(i.Number_rabies AS CHAR), ''),
  NULLIF(CAST(i.Number_malaria AS CHAR), '')
FROM pandemic.infectious_cases i
JOIN pandemic.locations l
  ON i.Entity = l.entity AND i.Code = l.code;

2_5
Перевірка кількості записів

SELECT COUNT(*) FROM pandemic.infectious_cases;
SELECT COUNT(*) FROM pandemic.infectious_cases_normalized;

3_1

SELECT 
  l.entity,
  l.code,
  COUNT(icn.number_rabies) AS rabies_count,
  AVG(icn.number_rabies) AS avg_rabies,
  MIN(icn.number_rabies) AS min_rabies,
  MAX(icn.number_rabies) AS max_rabies,
  SUM(icn.number_rabies) AS total_rabies
FROM pandemic.infectious_cases_normalized icn
JOIN pandemic.locations l ON icn.location_id = l.location_id
WHERE icn.number_rabies IS NOT NULL
GROUP BY l.entity, l.code
ORDER BY avg_rabies DESC
LIMIT 10;

4_1

SELECT
  year,
  STR_TO_DATE(CONCAT(year, '-01-01'), '%Y-%m-%d') AS first_january_date,
  CURDATE() AS today,
  TIMESTAMPDIFF(YEAR, STR_TO_DATE(CONCAT(year, '-01-01'), '%Y-%m-%d'), CURDATE()) AS years_difference
FROM pandemic.infectious_cases_normalized;

5-1

DELIMITER $$

DROP FUNCTION IF EXISTS year_diff $$

CREATE FUNCTION year_diff(given_year INT) RETURNS INT
DETERMINISTIC
BEGIN
  DECLARE years_difference INT;
  SET years_difference = TIMESTAMPDIFF(
    YEAR,
    STR_TO_DATE(CONCAT(given_year, '-01-01'), '%Y-%m-%d'),
    CURDATE()
  );
  RETURN years_difference;
END $$

DELIMITER ;

SELECT 
  year,
  year_diff(year) AS years_since
FROM pandemic.infectious_cases_normalized;

