# Transportation-Project
-- Data overview

select * from test.gc;
select count(*) from test.gc;

select * from test.sheet;
select count(*) from test.sheet;

select * from test.cancelled;
select count(*) from test.cancelled;

select* from test.noshow;
select count(*) from noshow;
-- ---------------------------------------------------------------------------------------
-- Changing the format of date coloumn in gc table
-- As data type of date is text so use srt_to_date Command to 
-- Add week_of_day column to make data useful


SET SQL_SAFE_UPDATES = 0;

UPDATE test.gc
SET `DATE` = STR_TO_DATE(`DATE`, '%d %M %Y')
WHERE `DATE` IS NOT NULL AND `DATE` <> '';

update test.gc
set `DATE`= date_format(`DATE`, '%d-%m-%Y');

SELECT DATE, DAYNAME(DATE) AS day_of_week
FROM test.gc;

ALTER TABLE test.gc
ADD COLUMN week_of_day VARCHAR(55);

update test.gc
set week_of_day = DAYNAME(STR_TO_DATE(`DATE`, '%d-%m-%Y'));

SET SQL_SAFE_UPDATES = 1;

select * from test.gc;
-- -------------------------------------------------------------------------------------------------------------------
-- -------------------------------------------------------------------------------------------------------------------
-- Filling Empty values to Null

-- ------------------- First see Empty values in the columns

select * from test.gc;

SELECT 'TIME' AS column_name, COUNT(*) AS empty_count FROM test.gc WHERE TIME = ''
UNION ALL
SELECT 'OTHER INFO', COUNT(*) FROM test.gc WHERE `OTHER INFO` = ''
UNION ALL
SELECT 'CALLER', COUNT(*) FROM test.gc WHERE CALLER = ''
UNION ALL
SELECT 'PICKUP', COUNT(*) FROM test.gc WHERE PICKUP = ''
UNION ALL
SELECT 'DROP', COUNT(*) FROM test.gc WHERE `DROP` = ''
union all
select 'TYPE', COUNT(*) FROM  test.gc WHERE 'TYPE' = ''
union all
select 'FLIGHT NO', COUNT(*) FROM  test.gc WHERE `FLIGHT NO`  = ''
union all
select 'FARE', COUNT(*) FROM  test.gc WHERE `FARE`  = ''
union all
select 'MFARE', COUNT(*) FROM  test.gc WHERE `MFARE`  = ''
union all
select 'ACCOUNT', COUNT(*) FROM  test.gc WHERE `ACCOUNT`  = ''
union all
select 'DRIVER', COUNT(*) FROM  test.gc WHERE `DRIVER`  = ''
union all
select 'VEHICLE', COUNT(*) FROM  test.gc WHERE `VEHICLE`  = ''
UNION ALL
SELECT 'MOBILE', COUNT(*) FROM test.gc WHERE `MOBILE` = ''


-- ----------------------------Create procedure Function to get Empty table easily


DELIMITER //

CREATE PROCEDURE emptygc()
BEGIN
    SELECT 'TIME' AS column_name, COUNT(*) AS empty_count FROM test.gc WHERE TIME = ''
    UNION ALL
    SELECT 'OTHER INFO', COUNT(*) FROM test.gc WHERE `OTHER INFO` = ''
    UNION ALL
    SELECT 'CALLER', COUNT(*) FROM test.gc WHERE CALLER = ''
    UNION ALL
    SELECT 'PICKUP', COUNT(*) FROM test.gc WHERE PICKUP = ''
    UNION ALL
    SELECT 'DROP', COUNT(*) FROM test.gc WHERE `DROP` = ''
    UNION ALL
    SELECT 'TYPE', COUNT(*) FROM test.gc WHERE `TYPE` = ''
    UNION ALL
    SELECT 'FLIGHT NO', COUNT(*) FROM test.gc WHERE `FLIGHT NO` = ''
    UNION ALL
    SELECT 'FARE', COUNT(*) FROM test.gc WHERE `FARE` = ''
    UNION ALL
    SELECT 'MFARE', COUNT(*) FROM test.gc WHERE `MFARE` = ''
    UNION ALL
    SELECT 'ACCOUNT', COUNT(*) FROM test.gc WHERE `ACCOUNT` = ''
    UNION ALL
    SELECT 'DRIVER', COUNT(*) FROM test.gc WHERE `DRIVER` = ''
    UNION ALL
    SELECT 'VEHICLE', COUNT(*) FROM test.gc WHERE `VEHICLE` = ''
    UNION ALL
    SELECT 'MOBILE', COUNT(*) FROM test.gc WHERE `MOBILE` = '';
END//

DELIMITER ;


DELIMITER //

CREATE PROCEDURE mygc()
BEGIN
    SELECT * FROM test.gc;
END//

DELIMITER ;


-- --------------------------------------------Replacing Empty '' to Null


SET SQL_SAFE_UPDATES = 0;

UPDATE test.gc
SET   
    `OTHER INFO` = NULLIF(`OTHER INFO`, ''),
    CALLER = NULLIF(CALLER, ''),
    PICKUP = NULLIF(PICKUP, ''),
    `DROP` = NULLIF(`DROP`, ''),
    `FLIGHT NO` = NULLIF(`FLIGHT NO`, ''),
    FARE = NULLIF(FARE, ''),
    MFARE = NULLIF(MFARE, ''),
    `ACCOUNT` = NULLIF(`ACCOUNT`, ''),
    DRIVER = NULLIF(DRIVER, ''),
    VEHICLE = NULLIF(VEHICLE, ''),
    MOBILE = NULLIF(MOBILE, '')
WHERE      
    `OTHER INFO` = '' OR     
    CALLER = '' OR     
    PICKUP = '' OR     
    `DROP` = '' OR     
    `FLIGHT NO` = '' OR     
    FARE = '' OR     
    MFARE = '' OR     
    `ACCOUNT` = '' OR     
    DRIVER = '' OR     
    VEHICLE = '' OR      
    MOBILE = '';
call mygc();
-- -------------------------------------------------------------------------------------------------------------------------
-- -------------------------------------------------------------------------------------------------------------------------
--                          Deleting unnecessary Columns and duplicate Jobs(Rows)

ALTER TABLE test.gc
DROP COLUMN VIA;

ALTER TABLE test.gc
DROP COLUMN WAITING;


-- -------------(due to systemetical glicth of software produced duplicate jobs) diff methods to fetch duplicate jobs

-- Its not possible to satisfied all conditions after 'where' clause due to complex nature of unwanted data so we delete one by one 

-- 1- First apply condition where Account is null and driver is null(bcoz job not been dispatched & unmarked acct by operator) '55 duplicte jobs deleted' when time condition kept on


-- ------------------------------ SHOW     
                                   
select* from(
   SELECT *, ROW_NUMBER() OVER (PARTITION BY 
								 `DATE`,
                                 `time`,
                                 `ACCOUNT`,
                                  MOBILE,
                                  CALLER,
                                  PICKUP,
								
                                  `DROP`
                                ORDER BY `OUR REF`) as row_num
FROM test.gc) as x
WHERE row_num > 1 and ACCOUNT= 'Mc';

-- where MOBILE is null and PICKUP is null or `drop` is null
-- and `ACCOUNT` is null  and VEHICLE is null;
-- and mobile is null
-- and DRIVER IS NULL and VEHICLE NOT IN ('16 Seater', 'Mini Van') and `ACCOUNT` not in ('taxicode', 'GetCab', 'supplier.ots');


-- --------------------------------------DELETE

SET SQL_SAFE_UPDATES = 0;  

delete from test.gc
where `OUR REF` in (
			SELECT `OUR REF` 
			FROM (
				SELECT *, 
					   ROW_NUMBER() OVER (PARTITION BY 
											  `DATE`,
											  `time`,
											  `ACCOUNT`,
											  MOBILE,
											  CALLER,
											  PICKUP,
											  `DROP`
											ORDER BY `OUR REF`) AS row_num 
				FROM test.gc
			) AS subquery_alias
			WHERE row_num > 1 AND `ACCOUNT` IS NULL AND VEHICLE IS NULL);
            
            
-- ---2- Delete Rows where and DRIVER IS NULL and VEHICLE NOT IN ('16 Seater', 'Mini Van') and `ACCOUNT` not in ('taxicode', 'GetCab', 'supplier.ots');



SET SQL_SAFE_UPDATES = 1;  

delete from test.gc
where `OUR REF` in (
			SELECT `OUR REF` 
			FROM (
				SELECT *, 
					   ROW_NUMBER() OVER (PARTITION BY 
											  `DATE`,
											  `time`,
											  `ACCOUNT`,
											  MOBILE,
											  CALLER,
											  PICKUP,
											  `DROP`
											ORDER BY `OUR REF`) AS row_num 
				FROM test.gc
			) AS subquery_alias
			WHERE row_num > 1 and DRIVER IS NULL and VEHICLE NOT IN ('16 Seater', 'Mini Van') and `ACCOUNT` not in ('tc', 'Gc', 'oT'));

-- ---------------------DELETE Fake OR testing jobs

delete from test.gc
where `OUR REF` in (
			SELECT `OUR REF` 
			FROM (
				SELECT *, 
					   ROW_NUMBER() OVER (PARTITION BY 
											  `DATE`,
											  `time`,
											  `ACCOUNT`,
											  MOBILE,
											  CALLER,
											  PICKUP,
											  `DROP`
											ORDER BY `OUR REF`) AS row_num 
				FROM test.gc
			) AS subquery_alias
			WHERE MOBILE is null and PICKUP is null or `drop` is null);
 
 
 
delete
from test.gc
where caller = 'demo';
-- ------------------------------------  fake/testing jobs
delete
from test.gc
where caller = 'fake';

delete
from test.gc
where caller = 'test job'

-- --------------------------------------------------------------------------------------------------------------------
-- --------------------------------------------------------------------------------------------------------------------
-- ---------------Delete Cancelled Job from 'gc' table with the help of cancelled job table (28/out of 29 job cancelled)
SELECT*
FROM test.cancelled;


SELECT *
FROM test.cancelled AS c
INNER JOIN test.gc AS g ON c.`Ref.` = g.`OUR REF`;

delete g
FROM test.gc AS g
INNER JOIN test.cancelled AS c ON g.`OUR REF` = c.`Ref.`;



-- --------------------------------------------------------------------------------------------------------------------------
-- --------------------------------------------------------------------------------------------------------------------------
-- -------------------------------------------------Filing Null Values
-- first see the null values in columns


SELECT 'TIME' AS column_name, COUNT(*) AS empty_count FROM test.gc WHERE TIME is null
UNION ALL
SELECT 'OTHER INFO', COUNT(*) FROM test.gc WHERE `OTHER INFO` is null
UNION ALL
SELECT 'CALLER', COUNT(*) FROM test.gc WHERE CALLER is null
UNION ALL
SELECT 'PICKUP', COUNT(*) FROM test.gc WHERE PICKUP is null
UNION ALL
SELECT 'DROP', COUNT(*) FROM test.gc WHERE `DROP`is null
union all
select 'TYPE', COUNT(*) FROM  test.gc WHERE 'TYPE'is null
union all
select 'FLIGHT NO', COUNT(*) FROM  test.gc WHERE `FLIGHT NO` is null
union all
select 'FARE', COUNT(*) FROM  test.gc WHERE `FARE` is null
union all
select 'MFARE', COUNT(*) FROM  test.gc WHERE `MFARE` is null
union all
select 'ACCOUNT', COUNT(*) FROM  test.gc WHERE `ACCOUNT` is null
union all
select 'DRIVER', COUNT(*) FROM  test.gc WHERE `DRIVER` is null
union all
select 'VEHICLE', COUNT(*) FROM  test.gc WHERE `VEHICLE` is null
UNION ALL
SELECT 'MOBILE', COUNT(*) FROM test.gc WHERE `MOBILE` is null;

DELIMITER //
create procedure nullgc()
begin 
	SELECT 'TIME' AS column_name, COUNT(*) AS empty_count FROM test.gc WHERE TIME is null
UNION ALL
SELECT 'OTHER INFO', COUNT(*) FROM test.gc WHERE `OTHER INFO` is null
UNION ALL
SELECT 'CALLER', COUNT(*) FROM test.gc WHERE CALLER is null
UNION ALL
SELECT 'PICKUP', COUNT(*) FROM test.gc WHERE PICKUP is null
UNION ALL
SELECT 'DROP', COUNT(*) FROM test.gc WHERE `DROP`is null
union all
select 'TYPE', COUNT(*) FROM  test.gc WHERE 'TYPE'is null
union all
select 'FLIGHT NO', COUNT(*) FROM  test.gc WHERE `FLIGHT NO` is null
union all
select 'FARE', COUNT(*) FROM  test.gc WHERE `FARE` is null
union all
select 'MFARE', COUNT(*) FROM  test.gc WHERE `MFARE` is null
union all
select 'ACCOUNT', COUNT(*) FROM  test.gc WHERE `ACCOUNT` is null
union all
select 'DRIVER', COUNT(*) FROM  test.gc WHERE `DRIVER` is null
union all
select 'VEHICLE', COUNT(*) FROM  test.gc WHERE `VEHICLE` is null
UNION ALL
SELECT 'MOBILE', COUNT(*) FROM test.gc WHERE `MOBILE` is null;

 end//
DELIMITER ;

call nullgc();
-- ----------------------------------------------------------------------------------------
--                                    Filling caller coloumn only 3 nulls

-- First check weather customer have same drop/mobile  OR  any fetch all jobs of day for any clue

select*
from test.gc
where caller is null;


select*
from test.gc
where `OUR REF` = 661 
union all
select*
from test.gc
where
-- PICKUP like '%st4 8jg%'
`drop` like '%st4 8jg%' Or MOBILE = '';  -- no match found replace with UnKnown

SET SQL_SAFE_UPDATES = 1;  

UPDATE test.gc
SET caller = 'unknown'
WHERE `our ref` = 661;
-- ---------------------------------------------
select*
from test.gc
where `OUR REF` = 2608 
union all
select*
from test.gc
where
PICKUP like '%Fairbourne Ave Alderley Edge SK9 7LU UK%' or `drop` like '%Fairbourne Ave Alderley Edge SK9 7LU UK%' Or MOBILE = ''; -- no match found so replace it with unknown

select*
from test.gc
where DATE = '06-06-2023';  -- check jobs on that day for clue

UPDATE test.gc
SET caller = 'unknown'
WHERE `our ref` = 2608;
-- -------------------------------------------

select*
from test.gc
where `OUR REF` = 4583;

select*
from test.gc
where `OUR REF` = 4583 
union all
select*
from test.gc
where
 `drop` like '%Holmfirth HD9 1RU' or
PICKUP like '%Holmfirth HD9 1RU%' or
MOBILE = '' or
MOBILE like '%%';                 -- no match found replace with UnKnown

select*
from test.gc
where DATE = '06-08-2023';

UPDATE test.gc
SET caller = 'unknown'
WHERE `our ref` = 4583;
-- ----------------------------------------------
-- ------------------------------------------------------------------------------------------------------------------------------
--                                     Filling mobile coloumn, only 11 nulls

-- First check weather customers have same pickup/drop/name  OR  any fetch all jobs of day for any clue


call nullgc();

select*
from test.gc
where MOBILE is null;


select*
from test.gc
where CALLER like '%san%';  -- found saad contact number from different bookings ''

SET SQL_SAFE_UPDATES = 1;  


update test.gc
set MOBILE = ''
where MOBILE is null and 
--  CALLER = 'saad' 
 CALLER = 'lotfi';    -- now 3 rows with saad name updated mob number
 -- ----------------------------------------------------------
 
 select*
from test.gc
where CALLER -- like '%Wealthall%' 
 like '%karen%' -- and PICKUP like '%m28%' 
 and `DROP` like '%m28%';
-- where PICKUP = 'M28 2ya' or `DROP` = 'M28 2ya';    --(non of condition is satisfied so, number is 'missing')


update test.gc
set mobile = 'missing'
where caller = 'Wealthall';
-- -------------------------------------------------------------

select*
from test.gc
 where CALLER like '%Christensen%';    -- under same full name and adress customer number is found ''


update test.gc
set mobile = ''
where caller = 'christensen' and mobile is null;
-- ---------------------------------------------------------- 
select*
from test.gc
 where CALLER like '%Saib%'; -- under full same name and adress customer mob number is found ''
 
 update test.gc
 set mobile = ''
 where caller = 'saib' and mobile is null;
 -- -----------------------------------------------------------
 select*
from test.gc 
where CALLER like '%Liselotte%';  -- one one particular job, no other record found so mob is'missing'

SET SQL_SAFE_UPDATES = 0;

update test.gc
 set mobile = 'missing'
 where caller = 'Paulsson/Liselotte Ms';
 -- ----------------------------------------------------------
 select*
from test.gc
 where CALLER like '%Kumar%';  -- under full same name and adress customer mob number is found ''
 
 update test.gc
 set mobile = ''
 where caller = 'Kumar' and mobile is null;
 -- ----------------------------------------------------------
 
 select*
from test.gc
 where CALLER like '%previous covered job%';
 
 select*
from test.gc
 -- where `date` = '14-01-2024' or
 where `drop` like '%M1 3BE%';
 -- where pickup like'%M1 3BE%'         -- non of conditions is satisfied so mob is 'missing'
 
 update test.gc
 set mobile = 'missing'
 where caller = 'previous covered job' and mobile is null;
 
 SET SQL_SAFE_UPDATES = 1;
 
 -- ------------------------------------------------------
 -- ---------------------------------------------------------------------------------------------------------------------------------
 
 --                                            Filling mfare coloumn, only 9 nulls

call nullgc();

select*
from test.gc
where mfare is null;


delete
from test.gc
where mfare is null and TYPE = 'cash';  -- 5 test-bookings deleted (because cash job must have price/ recognise by employees name and addresses)

select *
from test.gc
Where `date` = '22-03-2023';  -- check any clue find for this job on that day

SELECT *
FROM test.noshow
JOIN test.gc ON test.noshow.ref = test.gc.`our ref`
WHERE test.gc.MFARE is null and test.gc.FARE is null;  -- (as STATUS is no show of these journeys so no amount will be added 
                                                          -- in both column/ if driver were on pickup location would have any amount 
                                                          -- in fare column according to company policy. both null means driver and cus both were no show)

SET SQL_SAFE_UPDATES = 1;

update test.gc
set MFARE = 0
where `our ref` in (371, 392, 590);  -- 3 more column updated 

update test.gc
set FARE = 0
where `our ref` in (371, 392, 590);


select *
from test.gc
where `our ref` = 586;    -- have fare is 6 so assume mfare is also 6 because mostly are same

update test.gc
set MFARE = 6
where `our ref`= 586;       -- 1 row updated

-- -------------------------------------------------------
-- ------------------------------------------------------------------------------------------------------------------------------------------
                                        -- Filling 'OTHER INFO', 6776 values null
call mygc();
call nullgc();

select* 
from test.gc
where `OTHER INFO` is null;                   -- other info column basically have ref no. of company website or school/hospital 
							     -- account or other required companies job ref no. if there is null /empty mean not required or important

SET SQL_SAFE_UPDATES = 1;

update test.gc
set `OTHER INFO` = 'no'
where `OTHER INFO` is null;
-- -----------------------------------------------------------
-- ---------------------------------------------------------------------------------------------------------------------------------------
--                                    Filling 'FLIGHT NO', 4026 null values

call nullgc(); 
call mygc();    

-- use like command for %M90% , %Airport%, %MAN% for pickup column to identify airport jobs if found null then 'missing' if not then not an airport job
select*
from test.gc
where PICKUP like '%airport%' OR PICKUP  LIKE '%m90%' and `FLIGHT NO` is null;

SELECT *
FROM test.gc
WHERE (PICKUP LIKE '%airport%' or PICKUP LIKE '%m90%' or PICKUP LIKE '%terminal%') AND `FLIGHT NO` IS NULL;  -- 322 


                       -- need to check diff combination
SELECT *
FROM test.gc
WHERE 
-- PICKUP NOT LIKE '%airport%' and `FLIGHT NO` IS NULL;   -- 3747
 -- PICKUP NOT LIKE '%m90%' and `FLIGHT NO` IS NULL;         -- 3745
-- PICKUP NOT LIKE '%terminal%' and PICKUP NOT LIKE '%m90%' and `FLIGHT NO` IS NULL;   -- 3736
 -- (PICKUP NOT LIKE '%terminal%' or PICKUP NOT LIKE '%airport%') and `FLIGHT NO` IS NULL;  -- 3707
(PICKUP NOT LIKE '%airport%' and PICKUP NOT LIKE '%m90%') and `FLIGHT NO` IS NULL;  -- 3788


select*
from test.gc
where `FLIGHT NO` is null;

SET SQL_SAFE_UPDATES = 1;


update test.gc                            -- because it is not an airport pickup 
set `FLIGHT NO` = 'not required'   
where (PICKUP NOT LIKE '%airport%' and PICKUP NOT LIKE '%m90%') and `FLIGHT NO` IS NULL;  -- 3704 rows updated


update test.gc                            -- because it is an airport pickup 
set `FLIGHT NO` = 'missing'  
WHERE (PICKUP LIKE '%airport%' or PICKUP LIKE '%m90%' or PICKUP LIKE '%terminal%') AND `FLIGHT NO` IS NULL;  -- 322 updated
-- -------------------------------------------------------------
-- -----------------------------------------------------------------------------------------------------------------------------------
--                                            Filling 'FARE', 54 Nulls

select *
From test.gc
where FARE is null;

                                           -- According to company policy when thedriver were informed earlier about job cancellation..
select*                                        -- by cus/company before setting on road then no money is company gained so '0' driver
from test.gc                         
join noshow on test.gc.`OUR REF` = noshow.Ref
where test.gc.FARE is null;

SET SQL_SAFE_UPDATES = 0;

UPDATE test.gc                                  -- 33 rows updated
JOIN noshow ON test.gc.`OUR REF` = noshow.Ref
SET test.gc.FARE = 0           -- added 0 because company didnot get amount, 0.0 when company completed job by other sources
WHERE test.gc.FARE IS NULL;

--                                 As company didn't get the noshow amount when driver were not on pickup location

select*                                        
from test.gc                         
join noshow on test.gc.`OUR REF` = noshow.Ref
where test.gc.FARE = 0;

UPDATE test.gc                                  
JOIN noshow ON test.gc.`OUR REF` = noshow.Ref       -- MFare of these 33 jobs also be 0.0 intead of 0
SET test.gc.MFARE = 0.0
WHERE test.gc.FARE= 0;

--                             these job completed by company but not to dispatched driver 
							-- but to uber/other-company(some time drivers stuck into traffic) replace it with '0.0' instead '0'
select*                                        
from test.gc                         
join sheet on test.gc.`OUR REF` = sheet.`Ref. No.`
where test.gc.FARE is null;

UPDATE test.gc                                  -- 33 rows updated
JOIN sheet ON test.gc.`OUR REF` = sheet.`Ref. No.`
SET test.gc.FARE = 0.0          -- '0.0' is because to diff with '0'(above), because in mfare compant pay to other company not its own driver
WHERE test.gc.FARE IS NULL;

SET SQL_SAFE_UPDATES = 1;
-- -------------------------------------------------------------------
-- ---------------------------------------------------------------------------------------------------------------------------------------
--                                   filling account, 2291 nulls


call nullgc();

select*
from test.gc
where `ACCOUNT` is null and `OTHER INFO` != 'no';

select count(*), `TYPE`
from test.gc
where `ACCOUNT` is null                      -- `TYPE`
group by `TYPE`;            --  Account = 2243,  card = 47,   cash = 1  (null values) 

--   As we know that `TYPE` card having `OTHER INFO` any serial number(eg 12, 123) means it booked from company website so account will be GETCAB


select*
from test.gc
where ACCOUNT is null and `type` = 'card' and `other info` != 'no';

SET SQL_SAFE_UPDATES = 0;

update test.gc
set `ACCOUNT` = 'Gc'
where `ACCOUNT` is null and `type` = 'card' and `other info` != 'no';     -- 44 rows updated

-- 44/47 rows are named as gectcab account remaining 3 are named as 'unknown' because these were mistakenly booked as without account by by operator  

update test.gc
set `ACCOUNT` = 'unknown'
where `ACCOUNT` is null and `TYPE` ='card';


select*
from test.gc
where `account` is null and `TYPE` = 'cash';   -- as it is noticed that `OTHER INFO` has '109' ref number it means it was booked from company website


update test.gc
set `ACCOUNT` = 'Gc'
where `account` is null and `TYPE` = 'cash';

--                  as system is integrated with one company 'Mc' so only that account or direct from website 
--                 jobs booked automatically, jobs from other accounts manually booked by operators and have to select account(mandatory)
--                      so missing acount having 'no' in `OTHER INFO` column means it booked from 'Minicabit'


update test.gc
set `ACCOUNT` = 'Mc'
where `ACCOUNT` is null and `OTHER INFO` = 'no';    -- 2230 rows updated


--                Remaining 13 rows having `OTHER INFO` serial number resembles to company website ref num so Account will be Gc

update test.gc
set `ACCOUNT` = 'Gc'
where `ACCOUNT` is null and `OTHER INFO` != 'no';

SET SQL_SAFE_UPDATES = 1;
-- ------------------------------------------------------------
-- -----------------------------------------------------------------------------------------------------------------------------------------
--                                        Filling 'DRIVER', 1565 null values

call nullgc();

select*
from test.gc
where DRIVER is null;

select count(*), DRIVER
from test.gc 
group by DRIVER;           -- space: ' ' = 1     comma: ',' = 2       nulls = 1565          (need to replace all three kinds)

select*
from test.gc
where DRIVER= ' ';

SET SQL_SAFE_UPDATES = 1;

update test.gc
set  DRIVER= null
where DRIVER= ' ';       -- 1 row updated 

select*
from test.gc
where DRIVER= ',';

update test.gc
set  DRIVER= null
where DRIVER= ',';       -- 2 row updated 


--                           Jobs covered without drivers, named as 'OutSourced', are divided following category: 
-- 1- jobs coverd from other companies(e.g. uber, due to vehicle break-down): 2- drivers app were not launched: 3- some drivers newon app don't know how to use app in starting

update test.gc
set  DRIVER= 'OutSourced'
where DRIVER is null;                            --  1568 rows updated 

SET SQL_SAFE_UPDATES = 1;

-- -------------------------------------------------------------
-- ----------------------------------------------------------------------------------------------------------------------------------------
--                                   Filling 'VEHICLE', 2695 Nulls


call nullgc();

select*
from test.gc
where VEHICLE is null;

select count(*), `Vehicle Type`
from test.sheet
group by `Vehicle Type`;


select `OUR REF`, `DATE`, `TIME`, `ACCOUNT`, `TYPE`, CALLER, DRIVER, VEHICLE, `No.Of Passenger`, `Job Status`
from test.gc
join test.sheet on test.gc.`OUR REF` = test.sheet.`Ref. No.`
where test.gc.VEHICLE is null AND (test.sheet.`No.Of Passenger` >= 9);



select count(*), `No.Of Passenger`      -- analyse the range of no. of people
from test.sheet
where `Vehicle Type` = ''
group by `No.Of Passenger`
order by `No.Of Passenger`;

--                        As number of passengers are '0' that means unknown so fall into category of unknown

SET SQL_SAFE_UPDATES = 0;

UPDATE test.gc
join test.sheet on test.gc.`OUR REF` = test.sheet.`Ref. No.`
SET test.gc.vehicle = 'Unknown'
WHERE test.gc.vehicle IS NULL
AND test.sheet.`No.Of Passenger` = 0;     -- 16 rows updated


--             As number of passengers are greater than 0 and less than 5 that it assumed that they booked 'Saloon'

          
UPDATE test.gc
join test.sheet on test.gc.`OUR REF` = test.sheet.`Ref. No.`
SET test.gc.vehicle = 'Saloon'   -- 
WHERE test.gc.vehicle IS NULL
AND (test.sheet.`No.Of Passenger` <= 4 AND test.sheet.`No.Of Passenger` != 0);  -- 2560 rows updated


--                              As number of passengers are 5 it assumed they booked 'MPV'

UPDATE test.gc
join test.sheet on test.gc.`OUR REF` = test.sheet.`Ref. No.`
SET test.gc.vehicle = 'MPV'
WHERE test.gc.vehicle IS NULL
AND test.sheet.`No.Of Passenger` = 5;     -- 43  rows updated


--            As number of passengers are greater than 5 and less than 9 that assumed that they booked 'Mini Van'

UPDATE test.gc
join test.sheet on test.gc.`OUR REF` = test.sheet.`Ref. No.`
SET test.gc.vehicle = 'Mini Van'       
WHERE test.gc.vehicle IS NULL
AND (test.sheet.`No.Of Passenger` >= 6 AND test.sheet.`No.Of Passenger` <= 8);  -- 55 rows updated


--                        As number of passengers are more than 8 means they booked '16 Seater'


UPDATE test.gc
join test.sheet on test.gc.`OUR REF` = test.sheet.`Ref. No.`
SET test.gc.vehicle =  '16 Seater'    
WHERE test.gc.vehicle IS NULL
AND (test.sheet.`No.Of Passenger` >= 9);   -- 19 rows updated

SET SQL_SAFE_UPDATES = 1;
-- ----------------------------------------------------
-- ---------------------------------------------------------------------------------------------------------------------------------------
                                 -- Extract 'Pickup' Postcodes
-- MySQL provides several string functions that can help extract postcodes from addresses. Some useful functions include SUBSTRING_INDEX, LOCATE, REGEXP_SUBSTR, TRIM.
-- data might have different scenarios, such as postcodes being at different positions within the addresses or some addresses not containing postcodes at all, we'll need to handle these scenarios differently.

  
--  if the postcodes are not uniform and have different patterns, the 'IN' clause might not be suitable because it checks for exact matches. 
--                    In this case, using LIKE with wildcard characters might be more appropriate.

	--  With the help of this method we are able to get 85% postcodes / due to complex nature of data(address) ---> (most successful method)

SELECT 
    pickup, 
    SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1) AS postcode,
    LEFT(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), CHAR_LENGTH(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1)) - LOCATE(' ', REVERSE(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -3)))) AS p,
    TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) AS next_column
FROM test.gc;


SELECT 
    count(pickup), 
    -- SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1) AS postcode,
    -- LEFT(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), CHAR_LENGTH(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1)) - LOCATE(' ', REVERSE(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -3)))) AS p,
    TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) AS next_column
FROM test.gc
group by next_column;



-- with the help of postcode  5328/7342 is sorted by this method remaining 2014 also divided into below major group/ above 3(of group)
-- sorted by tracing these below postcodes almost 826/2014 sorted remaing are sent to unknown section very hard to handle them (only one by one)


/*        
'1NN UK'= 11           '1QX UK' =722               '5PH UK'=31               '(M1 2PZ)'= 21               'Road Manchester'= 21    
'Terminal 2'=16       'terminal 3'=14             'Terminal 1'=28            '(M11 3FF)'=12                '(M3 1AR)'=10    
'(M90 1QX)'=9        'Piccadilly Manchester'= 9    'off point'= 9              '(M16 0RA)'=7               '4LR UK'=5
'3AW UK'=5           'Hill Manchester'= 5         '(L24 1YD)'=4                '4QW UK'=4                 'Village Liverpool'= 4
'5PG UK'=4             'UK Address'=19(GC)       'UK Adddress'=215(GC)       'United Kingdom'=18        'Racecourse Newton-le-Willows'=4
'5PH UK'=31           'Alton Stoke-on-Trent'=4         '6HT UK'=4               'Airport MAN'= 9           'UK Restaurant'=11
'1YD UK'=10
*/


SELECT 
    count(pickup) as TotalNumber, 
        CASE 
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'M1%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'M2%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'M3%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'M4%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'M5%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'M6%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'M7%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'M8%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'M9%' THEN 'Manchester Airport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LS1%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LS2%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LS3%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LS4%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LS5%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LS6%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LS7%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LS8%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LS9%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'L1%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'L2%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'L3%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'L4%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'L5%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'L6%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'L7%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'L8%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'L9%' THEN 'Liverpool'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'B1%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'B2%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'B3%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'B4%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'B5%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'B6%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'B7%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'B8%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'B9%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'BB%' THEN 'Blackburn'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'BD%' THEN 'Bradford'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'BL1%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'BL2%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'BL3%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'BL4%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'BL5%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'BL6%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'BL8%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'BL7%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'BL9%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'BS%' THEN 'Bristol'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'BT%' THEN 'Belfast'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'CA%' THEN 'Carlisle'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'CB%' THEN 'Cambridge'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'CF%' THEN 'Cardiff'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'CH1%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'CH2%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'CH3%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'CH4%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'CH5%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'CH6%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'CH7%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'CH8%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'CH9%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'CT%' THEN 'Canterbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'CV%' THEN 'Coventry'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'CW%' THEN 'Crewe'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'DA%' THEN 'Dartford'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'DE%' THEN 'Derby'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'DG%' THEN 'Dumfries'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'DH%' THEN 'Durham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'DL%' THEN 'Darlington'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'DN%' THEN 'Doncaster'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'DT%' THEN 'Dorchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'DY%' THEN 'Dudley'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'E1%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'E2%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'E3%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'E4%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'E5%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'E6%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'E7%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'E8%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'E9%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'EC%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'EH%' THEN 'Edinburgh'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'EN%' THEN 'ENFIELD'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'EX%' THEN 'Exeter'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'FY%' THEN 'Blackpool'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'FK%' THEN 'Falkirk'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'G1%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'G2%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'G3%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'G4%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'G5%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'G6%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'G7%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'G8%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'G9%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'GL%' THEN 'Gloucester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'HA%' THEN 'Wembley'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'HD%' THEN 'Huddersfield'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'HG%' THEN 'Harrogate'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'HP%' THEN 'Hemel Hempstead'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'HU%' THEN 'HULL'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'HX%' THEN 'HALIFAX'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'KT%' THEN 'Kingston upon Thames'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'Kw%' THEN 'WICK'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LA1%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LA2%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LA3%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LA4%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LA5%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LA6%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LA7%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LA8%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LA9%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LE1%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LE2%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LE3%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LE4%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LE5%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LE6%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LE7%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LE8%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LE9%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LL%' THEN 'Wrexham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LN%' THEN 'LINCOLN'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'N1%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'N2%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'N3%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'N4%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'N5%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'N6%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'N7%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'N8%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'N9%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'NE%' THEN 'Newcastle upon Tyne'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'NG%' THEN 'Nottingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'NP%' THEN 'Pontypool'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'NW%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'OL1%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'OL2%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'OL3%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'OL4%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'OL5%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'OL6%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'OL7%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'OL8%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'OL9%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'OX%' THEN 'Oxford'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'PE%' THEN 'Peterborough'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'PL%' THEN 'Plymouth'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'PO%' THEN 'Portsmouth'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'PR1%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'PR2%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'PR3%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'PR4%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'PR5%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'PR6%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'PR7%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'PR8%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'PR9%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'S1%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'S2%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'S3%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'S4%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'S5%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'S6%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'S7%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'S8%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'S9%' THEN 'Sheffield'
	
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SE%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SK5%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SK2%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SK3%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SK4%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SK1%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SK6%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SK7%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SK8%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SK9%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SL%' THEN 'Slough'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SO%' THEN 'Southampton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'ST1%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'ST2%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'ST3%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'ST4%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'ST5%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'ST6%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'ST7%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'ST8%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'ST9%' THEN 'Stoke-on-Trent'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SW%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SY1%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SY2%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SY3%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SY4%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SY5%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SY6%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SY7%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SY8%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'SY9%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'TF%' THEN 'Telford'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'TS%' THEN 'Middlesbrough'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'UB%' THEN 'Southall'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'WA1%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'WA2%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'WA3%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'WA4%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'WA5%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'WA6%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'WA7%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'WA8%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'WA9%' THEN 'Warrington'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'WF%' THEN 'Wakefield'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'WV%' THEN 'Wolverhampton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'WN%' THEN 'Wigan'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'YO%' THEN 'York'
                             -- Remaining groups after sorting by cities
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE '1QX%' then 'Manchester Airport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE '5PH%' then 'Manchester Airport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE '1NN%' then 'Manchester Airport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'Terminal%' then 'Manchester Airport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE '(M%' then 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'off%' then 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE '%Manchester%' then 'Manchester'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'UK Airport%' then 'Manchester Airport'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'Airport%' then 'Manchester Airport'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE '4LR UK%' then 'Stockport'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE '1YD UK%' then 'Liverpool'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE '(L%' then 'Liverpool'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE '3AW%' then 'Manchester'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE '4QW%' then 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE '4LX%' then 'Manchester'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'Alton Stoke-on-Trent%' then 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'Village Liverpool%' then 'Liverpool'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'Racecourse Newton-le-Willows%' then 'Warrington'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE '5PG%' then 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE '4TA%' then 'Blackpool'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE '6HT%' then 'Derby'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'Liverpool%' then 'Liverpool'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'Street Liverpool%' then 'Liverpool'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'MAN%' then 'Manchester Airport'
         ELSE  case
				when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%manchester%' then 'Manchester'
				when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%Liverpool%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%preston%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%leeds%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%Sheffield%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%Warrington%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%Stockport%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%Oldham%' then 'Oldham'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%Birmingham%' then 'Birmingham'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%Blackpool%' then 'Blackpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%Blackburn%' then 'Blackburn'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%Bolton%' then 'Bolton'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%Chester%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%York%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%Warrington%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%m1%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%m2%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%m3%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%m4%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%m5%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%m6%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%m7%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%m8%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%m9%' then 'Manchester Airport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%sk1%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%sk2%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%sk3%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%sk4%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%sk5%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%sk6%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%sk7%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%sk8%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%sk9%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%cw%' then 'Crewe'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wa1%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wa2%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wa3%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wa4%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wa5%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wa6%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wa7%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wa8%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wa9%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%bb%' then 'Blackburn'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'ls1%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'ls2%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'ls3%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'ls4%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'ls5%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'ls6%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'ls7%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'ls8%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'ls9%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%bd%' then 'Bradford'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%hd%' then 'Huddersfield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%hx%' then 'HALIFAX'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%fy%' then 'Blackpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'NG%' then 'Nottingham'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'L1%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'L2%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'L3%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'L4%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'L5%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'L6%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'L7%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'L8%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'L9%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ch1%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ch2%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ch3%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ch4%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ch5%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ch6%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ch7%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ch8%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ch9%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 's1%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 's2%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 's3%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 's4%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 's5%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 's6%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 's7%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 's8%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 's9%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%yo1%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%yo2%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%yo3%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%yo4%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%yo5%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%yo6%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%yo7%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%yo8%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%yo9%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'la1%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'la2%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'la3%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'la4%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'la5%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'la6%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'la7%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'la8%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'la9%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wn1%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wn2%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wn3%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wn4%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wn5%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wn6%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wn7%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wn8%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%wn9%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ca1%' then 'Carlisle'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like 'hg%' then 'Harrogate'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%tw6%' then 'Hounslow'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%pr1%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%pr2%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%pr3%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%pr4%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%pr5%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%pr6%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%pr7%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%pr8%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%pr9%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%SY1%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%SY2%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%SY3%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%SY4%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%SY5%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%SY6%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%SY7%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%SY8%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%SY9%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ST1%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ST2%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ST3%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ST4%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ST5%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ST6%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ST7%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ST8%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%ST9%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%8DE United Kingdom%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%WF1%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%WF2%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%WF3%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%WF4%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%WF5%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%WF6%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%WF7%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%WF8%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -3) like '%WF9%' then 'Wakefield'
                
			    else case 
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%manchester%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%Liverpool%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%preston%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%leeds%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%Sheffield%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%Warrington%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%Stockport%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%Oldham%' then 'Oldham'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%Birmingham%' then 'Birmingham'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%Blackpool%' then 'Blackpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%Blackburn%' then 'Blackburn'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%Bolton%' then 'Bolton'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%Chester%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%York%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%Warrington%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%m1%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%m2%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%m3%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%m4%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%m5%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%m6%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%m7%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%m8%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%m9%' then 'Manchester Airport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%sk1%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%sk2%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%sk3%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%sk4%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%sk5%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%sk6%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%sk7%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%sk8%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%sk9%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%cw%' then 'Crewe'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wa1%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wa2%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wa3%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wa4%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wa5%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wa6%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wa7%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wa8%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wa9%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%bb%' then 'Blackburn'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'ls1%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'ls2%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'ls3%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'ls4%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'ls5%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'ls6%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'ls7%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'ls8%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'ls9%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%bd%' then 'Bradford'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%hd%' then 'Huddersfield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%hx%' then 'HALIFAX'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%fy%' then 'Blackpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'NG%' then 'Nottingham'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'L1%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'L2%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'L3%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'L4%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'L5%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'L6%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'L7%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'L8%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'L9%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ch1%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ch2%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ch3%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ch4%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ch5%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ch6%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ch7%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ch8%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ch9%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 's1%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 's2%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 's3%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 's4%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 's5%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 's6%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 's7%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 's8%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 's9%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%yo1%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%yo2%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%yo3%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%yo4%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%yo5%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%yo6%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%yo7%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%yo8%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%yo9%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'la1%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'la2%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'la3%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'la4%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'la5%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'la6%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'la7%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'la8%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'la9%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wn1%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wn2%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wn3%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wn4%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wn5%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wn6%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wn7%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wn8%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%wn9%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ca1%' then 'Carlisle'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like 'hg%' then 'Harrogate'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%tw6%' then 'Hounslow'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%pr1%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%pr2%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%pr3%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%pr4%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%pr5%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%pr6%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%pr7%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%pr8%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%pr9%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%SY1%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%SY2%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%SY3%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%SY4%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%SY5%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%SY6%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%SY7%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%SY8%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%SY9%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ST1%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ST2%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ST3%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ST4%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ST5%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ST6%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ST7%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ST8%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%ST9%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%8DE United Kingdom%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%WF1%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%WF2%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%WF3%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%WF4%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%WF5%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%WF6%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%WF7%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%WF8%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4) like '%WF9%' then 'Wakefield'
					else  'Missing Pcode/City' -- SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM pickup), ' ', -4)
             end   
		end
   END AS Cities
FROM test.gc
group by Cities
order by count(pickup) desc;
 
 --    ts  wf nn4   ws1   bs   de123   ne1   dn12   ox12    dl112   tf1    st12    dn1   ll15 wa12   co15

--    
select *
from test.gc
where PICKUP like '%Stockport%' or PICKUP like '%sk4%'; 



-- 2014 addresses not categorized need to check them all what category they fall





SELECT 
    pickup, 
    CASE 
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'M%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'LS%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) LIKE 'L%' THEN 'Liverpool'
        

        ELSE TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), ' ', -2)) 
    END AS next_column
FROM test.gc;


SELECT REGEXP_SUBSTR(pickup, '[A-Z]{1,2}\d{1,2} \d[A-Z]{2}$') AS postcode
FROM test.gc;



SELECT 
    pickup, 
    SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1) AS postcode,
    right(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), CHAR_LENGTH(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1)) - LOCATE(' ', REVERSE(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1)))) AS p,
    REGEXP_SUBSTR(LEFT(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1), CHAR_LENGTH(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1)) - LOCATE(' ', REVERSE(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM pickup), ', ', -1)))), '[A-Z]{1,2}\d{1,2} \d[A-Z]{2}$') AS postcode_from_p
FROM test.gc;

SELECT *
FROM test.gc
WHERE PICKUP LIKE '%yo%';


select*
from test.gc
where PICKUP LIKE '%sk1%'
or  PICKUP LIKE '%sk2%'
or  PICKUP LIKE '%sk3%'
or  PICKUP LIKE '%sk4%'
or  PICKUP LIKE '%sk5%'
or  PICKUP LIKE '%sk6%'
or  PICKUP LIKE '%sk7%'
or  PICKUP LIKE '%sk8%';
-- like '%Sheffield%'   --     '4TA UK'=  
-- -----------------------------------------------------------------------------------------------------------------------------------
-- ----------------------------------------------------------------------------------------------------------------------------------
--                                                   Extract 'Drop' Postcodes


call mygc();



SELECT 
    `DROP`, 
    SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1) AS postcode,
    LEFT(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), CHAR_LENGTH(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1)) - LOCATE(' ', REVERSE(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -3)))) AS p,
    TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) AS next_column
FROM test.gc;



SELECT 
   count(`DROP`), 
   -- SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1) AS postcode,
   -- LEFT(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), CHAR_LENGTH(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1)) - LOCATE(' ', REVERSE(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -3)))) AS p,
    TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) AS next_column
FROM test.gc
group by next_column
order by count(`DROP`) desc;




SELECT 
    count(`Drop`), 
    CASE 
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'M1%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'M2%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'M3%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'M4%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'M5%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'M6%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'M7%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'M8%' THEN 'Manchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'M9%' THEN 'Manchester Airport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LS1%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LS2%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LS3%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LS4%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LS5%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LS6%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LS7%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LS8%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LS9%' THEN 'Leeds'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'L1%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'L2%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'L3%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'L4%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'L5%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'L6%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'L7%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'L8%' THEN 'Liverpool'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'L9%' THEN 'Liverpool'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'B1%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'B2%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'B3%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'B4%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'B5%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'B6%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'B7%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'B8%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'B9%' THEN 'Birmingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'BB%' THEN 'Blackburn'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'BD%' THEN 'Bradford'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'BL1%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'BL2%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'BL3%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'BL4%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'BL5%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'BL6%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'BL8%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'BL7%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'BL9%' THEN 'Bolton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'BS%' THEN 'Bristol'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'BT%' THEN 'Belfast'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'CA%' THEN 'Carlisle'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'CB%' THEN 'Cambridge'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'CF%' THEN 'Cardiff'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'CH1%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'CH2%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'CH3%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'CH4%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'CH5%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'CH6%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'CH7%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'CH8%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'CH9%' THEN 'Chester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'CT%' THEN 'Canterbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'CV%' THEN 'Coventry'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'CW%' THEN 'Crewe'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'DA%' THEN 'Dartford'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'DE%' THEN 'Derby'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'DG%' THEN 'Dumfries'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'DH%' THEN 'Durham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'DL%' THEN 'Darlington'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'DN%' THEN 'Doncaster'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'DT%' THEN 'Dorchester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'DY%' THEN 'Dudley'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'E1%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'E2%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'E3%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'E4%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'E5%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'E6%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'E7%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'E8%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'E9%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'EC%' THEN 'London'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'EH%' THEN 'Edinburgh'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'EN%' THEN 'ENFIELD'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'EX%' THEN 'Exeter'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'FY%' THEN 'Blackpool'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'FK%' THEN 'Falkirk'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'G1%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'G2%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'G3%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'G4%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'G5%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'G6%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'G7%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'G8%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'G9%' THEN 'Glasgow'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'GL%' THEN 'Gloucester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'HA%' THEN 'Wembley'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'HD%' THEN 'Huddersfield'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'HG%' THEN 'Harrogate'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'HP%' THEN 'Hemel Hempstead'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'HU%' THEN 'HULL'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'HX%' THEN 'HALIFAX'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'KT%' THEN 'Kingston upon Thames'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'Kw%' THEN 'WICK'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LA1%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LA2%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LA3%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LA4%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LA5%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LA6%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LA7%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LA8%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LA9%' THEN 'LANCASTER'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LE1%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LE2%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LE3%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LE4%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LE5%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LE6%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LE7%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LE8%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LE9%' THEN 'Leicester'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LL%' THEN 'Wrexham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'LN%' THEN 'LINCOLN'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'N1%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'N2%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'N3%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'N4%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'N5%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'N6%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'N7%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'N8%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'N9%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'NE%' THEN 'Newcastle upon Tyne'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'NG%' THEN 'Nottingham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'NP%' THEN 'Pontypool'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'NW%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'OL1%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'OL2%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'OL3%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'OL4%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'OL5%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'OL6%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'OL7%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'OL8%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'OL9%' THEN 'Oldham'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'OX%' THEN 'Oxford'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'PE%' THEN 'Peterborough'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'PL%' THEN 'Plymouth'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'PO%' THEN 'Portsmouth'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'PR1%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'PR2%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'PR3%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'PR4%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'PR5%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'PR6%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'PR7%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'PR8%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'PR9%' THEN 'PRESTON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'S1%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'S2%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'S3%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'S4%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'S5%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'S6%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'S7%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'S8%' THEN 'Sheffield'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'S9%' THEN 'Sheffield'
	
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SE%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SK5%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SK2%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SK3%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SK4%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SK1%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SK6%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SK7%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SK8%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SK9%' THEN 'Stockport'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SL%' THEN 'Slough'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SO%' THEN 'Southampton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'ST1%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'ST2%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'ST3%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'ST4%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'ST5%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'ST6%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'ST7%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'ST8%' THEN 'Stoke-on-Trent'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'ST9%' THEN 'Stoke-on-Trent'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SW%' THEN 'LONDON'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SY1%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SY2%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SY3%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SY4%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SY5%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SY6%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SY7%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SY8%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'SY9%' THEN 'Shrewsbury'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'TF%' THEN 'Telford'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'TS%' THEN 'Middlesbrough'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'UB%' THEN 'Southall'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'WA1%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'WA2%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'WA3%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'WA4%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'WA5%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'WA6%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'WA7%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'WA8%' THEN 'Warrington'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'WA9%' THEN 'Warrington'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'WF%' THEN 'Wakefield'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'WV%' THEN 'Wolverhampton'
        WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'WN%' THEN 'Wigan'
		WHEN TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(TRIM(TRAILING ', UK' FROM `DROP`), ', ', -1), ' ', -2)) LIKE 'YO%' THEN 'York'
        else case
				when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%manchester%' then 'Manchester'
				when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%Liverpool%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%preston%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%leeds%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%Sheffield%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%Warrington%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%Stockport%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%Oldham%' then 'Oldham'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'OL1%' then 'Oldham'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%Birmingham%' then 'Birmingham'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%Blackpool%' then 'Blackpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%Blackburn%' then 'Blackburn'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%Bolton%' then 'Bolton'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'BL1%' then 'Bolton'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%Chester%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'LANCASTER%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%York%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%Wa16%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%m1%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%m2%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%m3%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%m4%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%m5%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%m6%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%m7%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%m8%' then 'Manchester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%m9%' then 'Manchester Airport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%sk1%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%sk2%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%sk3%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%sk4%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%sk5%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%sk6%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%sk7%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%sk8%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%sk9%' then 'Stockport'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%cw%' then 'Crewe'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wa1%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wa2%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wa3%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wa4%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wa5%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wa6%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wa7%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wa8%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wa9%' then 'Warrington'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%bb%' then 'Blackburn'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'ls1%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'ls2%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'ls3%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'ls4%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'ls5%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'ls6%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'ls7%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'ls8%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'ls9%' then 'Leeds'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%bd%' then 'Bradford'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%hd%' then 'Huddersfield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%hx%' then 'HALIFAX'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%fy%' then 'Blackpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'NG%' then 'Nottingham'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'L1%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'L2%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'L3%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'L4%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'L5%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'L6%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'L7%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'L8%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'L9%' then 'Liverpool'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ch1%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ch2%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ch3%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ch4%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ch5%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ch6%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ch7%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ch8%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ch9%' then 'Chester'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 's1%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 's2%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 's3%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 's4%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 's5%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 's6%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 's7%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 's8%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 's9%' then 'Sheffield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%yo1%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%yo2%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%yo3%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%yo4%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%yo5%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%yo6%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%yo7%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%yo8%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%yo9%' then 'York'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'la1%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'la2%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'la3%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'la4%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'la5%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'la6%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'la7%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'la8%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'la9%' then 'LANCASTER'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wn1%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wn2%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wn3%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wn4%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wn5%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wn6%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wn7%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wn8%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wn9%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ca1%' then 'Carlisle'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like 'hg%' then 'Harrogate'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%tw6%' then 'Hounslow'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%pr1%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%pr2%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%pr3%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%pr4%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%pr5%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%pr6%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%pr7%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%pr8%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%pr9%' then 'PRESTON'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%SY1%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%SY2%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%SY3%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%SY4%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%SY5%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%SY6%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%SY7%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%SY8%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%SY9%' then 'Shrewsbury'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ST1%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ST2%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ST3%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ST4%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ST5%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ST6%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ST7%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ST8%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%ST9%' then 'Stoke-on-Trent'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%8DE United Kingdom%' then 'Wigan'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%WF1%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%WF2%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%WF3%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%WF4%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%WF5%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%WF6%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%WF7%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%WF8%' then 'Wakefield'
                when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%WF9%' then 'Wakefield'
                
			    else  case
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%manchester%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%Liverpool%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%preston%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%leeds%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%Sheffield%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%Warrington%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%Stockport%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%Oldham%' then 'Oldham'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'OL1%' then 'Oldham'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%Birmingham%' then 'Birmingham'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%Blackpool%' then 'Blackpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%Blackburn%' then 'Blackburn'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%Bolton%' then 'Bolton'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'BL1%' then 'Bolton'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%Chester%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'LANCASTER%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%York%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%Wa16%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%m1%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%m2%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%m3%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%m4%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%m5%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%m6%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%m7%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%m8%' then 'Manchester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%m9%' then 'Manchester Airport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%sk1%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%sk2%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%sk3%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%sk4%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%sk5%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%sk6%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%sk7%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%sk8%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%sk9%' then 'Stockport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%cw%' then 'Crewe'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wa1%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wa2%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wa3%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wa4%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wa5%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -3) like '%wa6%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wa7%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wa8%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wa9%' then 'Warrington'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%bb%' then 'Blackburn'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'ls1%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'ls2%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'ls3%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'ls4%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'ls5%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'ls6%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'ls7%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'ls8%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'ls9%' then 'Leeds'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%bd%' then 'Bradford'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%hd%' then 'Huddersfield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%hx%' then 'HALIFAX'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%fy%' then 'Blackpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'NG%' then 'Nottingham'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'L1%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'L2%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'L3%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'L4%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'L5%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'L6%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'L7%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'L8%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'L9%' then 'Liverpool'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ch1%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ch2%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ch3%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ch4%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ch5%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ch6%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ch7%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ch8%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ch9%' then 'Chester'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 's1%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 's2%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 's3%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 's4%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 's5%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 's6%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 's8%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 's9%' then 'Sheffield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%yo1%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%yo2%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%yo3%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%yo4%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%yo5%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%yo6%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%yo7%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%yo8%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%yo9%' then 'York'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'la1%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'la2%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'la3%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'la4%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'la5%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'la6%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'la7%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'la8%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'la9%' then 'LANCASTER'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wn1%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wn2%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wn3%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wn4%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wn5%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wn6%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wn7%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wn8%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%wn9%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ca1%' then 'Carlisle'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'hg%' then 'Harrogate'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%tw6%' then 'Hounslow'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%pr1%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%pr2%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%pr3%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%pr4%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%pr5%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%pr6%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%pr7%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%pr8%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%pr9%' then 'PRESTON'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%SY1%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%SY2%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%SY3%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%SY4%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%SY5%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%SY6%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%SY7%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%SY8%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%SY9%' then 'Shrewsbury'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ST1%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ST2%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ST3%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ST4%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ST5%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ST6%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ST7%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ST8%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%ST9%' then 'Stoke-on-Trent'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%8DE United Kingdom%' then 'Wigan'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%WF1%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%WF2%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%WF3%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%WF4%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%WF5%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%WF6%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%WF7%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%WF8%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%WF9%' then 'Wakefield'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like 'MAN%' then 'Manchester Airport'
					when SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4) like '%drop off point%' then 'Manchester'
					else  'Missing P-code/City'       -- SUBSTRING_INDEX(TRIM(TRAILING 'UK' FROM `DROP`), ' ', -4)
		  end  
	 end   
END AS next_column
FROM test.gc
group by next_column
order by count(`DROP`) desc;

-- 'MAN'     'drop off point'     'Airport Terminal 3';
 
select*
from test.gc
where `DROP` like '%drop off point%';
-- some cities from first part of coding with one name and postcode add if possible
-- ------------------------------------------------------------------------------------------------------------------------------------
-- ---------------------------------------------------------------------------------------------------------------------------------------
--                                      total no. of jobs per day or avg

call mygc();

--                                           Maximum jobs Dates

SELECT 
    `DATE`, COUNT(*) AS Tcount, max(week_of_day) as DayNames
FROM test.gc
GROUP BY `DATE`
ORDER BY COUNT(*) DESC
limit 30;
--                                   No. of jobs per Day Graph


SELECT `DATE`, COUNT(*) AS Tcount 
FROM test.gc 
GROUP BY `DATE` 
ORDER BY STR_TO_DATE(`DATE`, '%d/%m/%Y');

--                                 Weak and Busy Days of week

SELECT week_of_day, COUNT(*) AS Tcount 
FROM test.gc 
GROUP BY week_of_day
ORDER BY Tcount desc;

--                                         Avg of jobs on day

SELECT 
    week_of_day,
    COUNT(*) AS Tcount,
    COUNT(*) / 52 AS avg_jobs_per_day
FROM test.gc 
GROUP BY week_of_day
order BY Tcount DESC;

--                                     WeekEnd vs WeekDays

SELECT 
    CASE 
        WHEN week_of_day IN ('Friday', 'Saturday', 'Sunday') THEN 'Week_end (Fri-Sun)'
        ELSE 'Week_days (Mon-Thu)'
    END AS Day_category,
    COUNT(*) AS Tcount 
FROM 
    test.gc 
GROUP BY 
    CASE 
        WHEN week_of_day IN ('Friday', 'Saturday', 'Sunday') THEN 'Week_end (Fri-Sun)'
        ELSE 'Week_days (Mon-Thu)'
    END
ORDER BY Tcount DESC;


--                                   Avg jobs per day

SELECT
    AVG(Tcount) AS AvgJobsPerDay
FROM
    (SELECT `DATE`, COUNT(*) AS Tcount 
    FROM test.gc 
    GROUP BY `DATE`) AS JobCounts;
    
    --                                    Gc Anual job per day
    SELECT
    AVG(Tcount) AS GC_AvgJobsPerDay
FROM
    (SELECT `DATE`, COUNT(*) AS Tcount 
    FROM test.gc 
    WHERE `ACCOUNT` = 'Gc' -- and `OTHER INFO` != 'no'
    GROUP BY `DATE`) AS JobCounts;

 --                          Gc anual jobs ghraph (only website)

SELECT 
     
    CASE 
        WHEN `DATE` LIKE '%-03-2023%' THEN 'Mar'
        WHEN `DATE` LIKE '%-04-2023%' THEN 'Apr'
        WHEN `DATE` LIKE '%-05-2023%' THEN 'May'
        WHEN `DATE` LIKE '%-06-2023%' THEN 'Jun'
        WHEN `DATE` LIKE '%-07-2023%' THEN 'Jul'
        WHEN `DATE` LIKE '%-08-2023%' THEN 'Aug'
        WHEN `DATE` LIKE '%-09-2023%' THEN 'Sep'
        WHEN `DATE` LIKE '%-10-2023%' THEN 'Oct'
        WHEN `DATE` LIKE '%-11-2023%' THEN 'Nov'
        WHEN `DATE` LIKE '%-12-2023%' THEN 'Dec'
        WHEN `DATE` LIKE '%-01-2024%' THEN 'Jan'
        WHEN `DATE` LIKE '%-02-2024%' THEN 'Feb'
        ELSE '1st day of next year'
    END AS Month, count(`DATE`) AS TotalJobs
FROM test.gc
WHERE `ACCOUNT` = 'Gc' and `OTHER INFO` != 'no'
GROUP BY Month;


--                               Gc Account anual jobs ghraph(from all means)

SELECT 
     
    CASE 
        WHEN `DATE` LIKE '%-03-2023%' THEN 'Mar'
        WHEN `DATE` LIKE '%-04-2023%' THEN 'Apr'
        WHEN `DATE` LIKE '%-05-2023%' THEN 'May'
        WHEN `DATE` LIKE '%-06-2023%' THEN 'Jun'
        WHEN `DATE` LIKE '%-07-2023%' THEN 'Jul'
        WHEN `DATE` LIKE '%-08-2023%' THEN 'Aug'
        WHEN `DATE` LIKE '%-09-2023%' THEN 'Sep'
        WHEN `DATE` LIKE '%-10-2023%' THEN 'Oct'
        WHEN `DATE` LIKE '%-11-2023%' THEN 'Nov'
        WHEN `DATE` LIKE '%-12-2023%' THEN 'Dec'
        WHEN `DATE` LIKE '%-01-2024%' THEN 'Jan'
        WHEN `DATE` LIKE '%-02-2024%' THEN 'Feb'
        ELSE '1st day of next year'
    END AS Month, count(`DATE`) AS TotalJobs
FROM test.gc
WHERE `ACCOUNT` = 'Gc' -- and `OTHER INFO` != 'no'
GROUP BY Month;


--                                       for all jobs anual graph 

SELECT 
     
    CASE 
        WHEN `DATE` LIKE '%-03-2023%' THEN 'Mar'
        WHEN `DATE` LIKE '%-04-2023%' THEN 'Apr'
        WHEN `DATE` LIKE '%-05-2023%' THEN 'May'
        WHEN `DATE` LIKE '%-06-2023%' THEN 'Jun'
        WHEN `DATE` LIKE '%-07-2023%' THEN 'Jul'
        WHEN `DATE` LIKE '%-08-2023%' THEN 'Aug'
        WHEN `DATE` LIKE '%-09-2023%' THEN 'Sep'
        WHEN `DATE` LIKE '%-10-2023%' THEN 'Oct'
        WHEN `DATE` LIKE '%-11-2023%' THEN 'Nov'
        WHEN `DATE` LIKE '%-12-2023%' THEN 'Dec'
        WHEN `DATE` LIKE '%-01-2024%' THEN 'Jan'
        WHEN `DATE` LIKE '%-02-2024%' THEN 'Feb'
        ELSE '1st day of next year'
    END AS Month, count(`DATE`) AS TotalJobs
FROM test.gc
GROUP BY Month;

--                                      total out sourced jobs


select count(*)
FROM test.gc
WHERE DRIVER = 'OutSourced';

SELECT 
    CASE 
        WHEN `DATE` LIKE '%-03-2023%' THEN 'Mar'
        WHEN `DATE` LIKE '%-04-2023%' THEN 'Apr'
        WHEN `DATE` LIKE '%-05-2023%' THEN 'May'
        WHEN `DATE` LIKE '%-06-2023%' THEN 'Jun'
        WHEN `DATE` LIKE '%-07-2023%' THEN 'Jul'
        WHEN `DATE` LIKE '%-08-2023%' THEN 'Aug'
        WHEN `DATE` LIKE '%-09-2023%' THEN 'Sep'
        WHEN `DATE` LIKE '%-10-2023%' THEN 'Oct'
        WHEN `DATE` LIKE '%-11-2023%' THEN 'Nov'
        WHEN `DATE` LIKE '%-12-2023%' THEN 'Dec'
        WHEN `DATE` LIKE '%-01-2024%' THEN 'Jan'
        WHEN `DATE` LIKE '%-02-2024%' THEN 'Feb'
        ELSE '1st day of next year'
    END AS Month, count(DRIVER) AS Total_OS_Jobs
FROM test.gc
WHERE DRIVER = 'OutSourced'
GROUP BY Month;



--                                     total vs out sourced jobs


WITH MonthlyJobs AS (
    SELECT 
        CASE 
            WHEN `DATE` LIKE '%-03-2023%' THEN 'Mar'
            WHEN `DATE` LIKE '%-04-2023%' THEN 'Apr'
            WHEN `DATE` LIKE '%-05-2023%' THEN 'May'
            WHEN `DATE` LIKE '%-06-2023%' THEN 'Jun'
            WHEN `DATE` LIKE '%-07-2023%' THEN 'Jul'
            WHEN `DATE` LIKE '%-08-2023%' THEN 'Aug'
            WHEN `DATE` LIKE '%-09-2023%' THEN 'Sep'
            WHEN `DATE` LIKE '%-10-2023%' THEN 'Oct'
            WHEN `DATE` LIKE '%-11-2023%' THEN 'Nov'
            WHEN `DATE` LIKE '%-12-2023%' THEN 'Dec'
            WHEN `DATE` LIKE '%-01-2024%' THEN 'Jan'
            WHEN `DATE` LIKE '%-02-2024%' THEN 'Feb'
            ELSE '1st day of next year'
        END AS Month, 
        COUNT(`DATE`) AS TotalJobs
    FROM 
        test.gc
    GROUP BY 
        Month
)
SELECT 
    MonthlyJobs.Month, 
    MonthlyJobs.TotalJobs,
    COALESCE(OutSourcedJobs.Total_OS_Jobs, 0) AS Total_OS_Jobs
FROM 
    MonthlyJobs
LEFT JOIN (
    SELECT 
        CASE 
            WHEN `DATE` LIKE '%-03-2023%' THEN 'Mar'
            WHEN `DATE` LIKE '%-04-2023%' THEN 'Apr'
            WHEN `DATE` LIKE '%-05-2023%' THEN 'May'
            WHEN `DATE` LIKE '%-06-2023%' THEN 'Jun'
            WHEN `DATE` LIKE '%-07-2023%' THEN 'Jul'
            WHEN `DATE` LIKE '%-08-2023%' THEN 'Aug'
            WHEN `DATE` LIKE '%-09-2023%' THEN 'Sep'
            WHEN `DATE` LIKE '%-10-2023%' THEN 'Oct'
            WHEN `DATE` LIKE '%-11-2023%' THEN 'Nov'
            WHEN `DATE` LIKE '%-12-2023%' THEN 'Dec'
            WHEN `DATE` LIKE '%-01-2024%' THEN 'Jan'
            WHEN `DATE` LIKE '%-02-2024%' THEN 'Feb'
            ELSE '1st day of next year'
        END AS Month, 
        COUNT(DRIVER) AS Total_OS_Jobs
    FROM 
        test.gc
    WHERE 
        DRIVER = 'OutSourced'
    GROUP BY 
        Month
) AS OutSourcedJobs ON MonthlyJobs.Month = OutSourcedJobs.Month;

SELECT caller, COUNT(*) AS visit_count, `ACCOUNT`
FROM test.gc
where `ACCOUNT` = 'oT'
GROUP BY CALLER
ORDER BY visit_count DESC
limit 5;


select round(sum(FARE)*0.1, 2) as Commision, round(sum(MFARE), 2) as Acctual_Amount, round(sum(MFARE-FARE), 2) as Diff_of_Add
from test.gc;
-- where DATE like '%-03-2023%';


select `OUR REF`, FARE, MFARE, (MFARE-FARE)
from test.gc
where DATE like '%-03-2023%';

--                                              No Show Amount

select sum(test.gc.MFARE - test.gc.FARE)  -- test.gc.FARE, test.gc.MFARE, test.sheet.`Job Status`, 
from test.gc
join test.sheet on test.gc.`OUR REF` = test.sheet.`Ref. No.`
where test.sheet.`Job Status` = 'noshow' and test.gc.FARE in (5, 16);



-- total no. of companies acct percentage

select `ACCOUNT`, count(*) as Tcount
from test.gc
group by `ACCOUNT`
order by Tcount desc
limit 4;

SELECT 
    `ACCOUNT`, 
    COUNT(*) AS Tcount,
    COUNT(*) * 100.0 / (SELECT COUNT(*) FROM test.gc) AS Percentage
FROM test.gc
GROUP BY `ACCOUNT`
ORDER BY Tcount DESC
LIMIT 4;
