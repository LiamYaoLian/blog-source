---
title: MySQL Stored Procedures
date: 2022-12-21 15:31:57
tags:
---
# Practice
https://leetcode.com/problems/dynamic-pivoting-of-a-table/description/
# Create
```
DELIMITER $$

CREATE PROCEDURE GetCustomers()
BEGIN
	SELECT
		customerName,
		city,
		state,
		postalCode,
		country
	FROM
		customers
	ORDER BY customerName;    
END$$
DELIMITER ;
```
# Call
```
CALL GetCustomers();
```
# Drop
```
DROP PROCEDURE [IF EXISTS] stored_procedure_name;
```
# Declare variable
inside `begin`:
```
DECLARE variable_name datatype(size) [DEFAULT default_value];
```
# Parameter
`[IN | OUT | INOUT] parameter_name datatype[(length)]`
* in: read-only
```
DELIMITER $$

CREATE PROCEDURE GetOrderCountByStatus (
	IN  orderStatus VARCHAR(25),
	OUT total INT
)
BEGIN
	SELECT COUNT(orderNumber)
	INTO total
	FROM orders
	WHERE status = orderStatus;
END$$

DELIMITER ;
```
```
CALL GetOrderCountByStatus('Shipped',@total);
SELECT @total;
```
# Condition
* IF-THEN-ELSEIF- ELSE
* Case
```
CASE case_value
   WHEN when_value1 THEN statements
   WHEN when_value2 THEN statements
   ...
   [ELSE else-statements]
END CASE;
```
```
CASE
    WHEN search_condition1 THEN statements
    WHEN search_condition1 THEN statements
    ...
    [ELSE else-statements]
END CASE;
```
# Loop
## Loop
```
[begin_label:] LOOP
    statement_list
END LOOP [end_label]
```
```
DROP PROCEDURE LoopDemo;

DELIMITER $$
CREATE PROCEDURE LoopDemo()
BEGIN
	DECLARE x  INT;
	DECLARE str  VARCHAR(255);

	SET x = 1;
	SET str =  '';

	loop_label:  LOOP
		IF  x > 10 THEN
			LEAVE  loop_label;
		END  IF;

		SET  x = x + 1;
		IF  (x mod 2) THEN
			ITERATE  loop_label;
		ELSE
			SET  str = CONCAT(str,x,',');
		END  IF;
	END LOOP;
	SELECT str;
END$$

DELIMITER ;
```
## While
```
[begin_label:] WHILE search_condition DO
    statement_list
END WHILE [end_label]
```
## Repeat
```
[begin_label:] REPEAT
    statement
UNTIL search_condition
END REPEAT [end_label]
```

# Leave
```
LEAVE label;
```
```
DELIMITER $$

CREATE PROCEDURE CheckCredit(
    inCustomerNumber int
)
sp: BEGIN

    DECLARE customerCount INT;

    -- check if the customer exists
    SELECT
        COUNT(*)
    INTO customerCount
    FROM
        customers
    WHERE
        customerNumber = inCustomerNumber;

    -- if the customer does not exist, terminate
    -- the stored procedure
    IF customerCount = 0 THEN
        LEAVE sp;
    END IF;

    -- other logic
    -- ...
END$$

DELIMITER ;
Cod
```
```
DELIMITER $$

CREATE PROCEDURE LeaveDemo(OUT result VARCHAR(100))
BEGIN
    DECLARE counter INT DEFAULT 1;
    DECLARE times INT;
    -- generate a random integer between 4 and 10
    SET times  = FLOOR(RAND()*(10-4+1)+4);
    SET result = '';
    disp: LOOP
        -- concatenate counters into the result
        SET result = concat(result,counter,',');

        -- exit the loop if counter equals times
        IF counter = times THEN
            LEAVE disp;
        END IF;
        SET counter = counter + 1;
    END LOOP;
END$$

DELIMITER ;
```

# Cursor
```
DECLARE cursor_name CURSOR FOR SELECT_statement;
OPEN cursor_name;
FETCH cursor_name INTO variables list;
CLOSE cursor_name;
DECLARE CONTINUE HANDLER FOR NOT FOUND SET finished = 1;
```
```
DELIMITER $$
CREATE PROCEDURE createEmailList (
	INOUT emailList varchar(4000)
)
BEGIN
	DECLARE finished INTEGER DEFAULT 0;
	DECLARE emailAddress varchar(100) DEFAULT "";

	-- declare cursor for employee email
	DEClARE curEmail
		CURSOR FOR
			SELECT email FROM employees;

	-- declare NOT FOUND handler
	DECLARE CONTINUE HANDLER
        FOR NOT FOUND SET finished = 1;

	OPEN curEmail;

	getEmail: LOOP
		FETCH curEmail INTO emailAddress;
		IF finished = 1 THEN
			LEAVE getEmail;
		END IF;
		-- build email list
		SET emailList = CONCAT(emailAddress,";",emailList);
	END LOOP getEmail;
	CLOSE curEmail;

END$$
DELIMITER ;
```

# Handling Error
* `DECLARE action HANDLER FOR condition_value statement;`
* action
  - `CONTINUE`
  - `EXIT`
```
DECLARE EXIT HANDLER FOR SQLEXCEPTION
BEGIN
    ROLLBACK;
    SELECT 'An error has occurred, operation rollbacked and the stored procedure was terminated';
END;
```
```
DECLARE CONTINUE HANDLER FOR NOT FOUND
SET RowNotFound = 1;
```
```
DECLARE CONTINUE HANDLER FOR 1062
SELECT 'Error, duplicate key occurred';
```

```
CREATE PROCEDURE InsertSupplierProduct(
    IN inSupplierId INT,
    IN inProductId INT
)
BEGIN
    -- exit if the duplicate key occurs
    DECLARE EXIT HANDLER FOR 1062
    BEGIN
 	SELECT CONCAT('Duplicate key (',inSupplierId,',',inProductId,') occurred') AS message;
    END;

    -- insert a new row into the SupplierProducts
    INSERT INTO SupplierProducts(supplierId,productId)
    VALUES(inSupplierId,inProductId);

    -- return the products supplied by the supplier id
    SELECT COUNT(*)
    FROM SupplierProducts
    WHERE supplierId = inSupplierId;

END$$

DELIMITER ;
```
## Precedence
* error code
* An SQLSTATE may map to many MySQL error codes, therefore, it is less specific.
* An SQLEXCPETION or an SQLWARNING is the shorthand for a class of SQLSTATES values so it is the most generic
```
DROP PROCEDURE IF EXISTS InsertSupplierProduct;

DELIMITER $$

CREATE PROCEDURE InsertSupplierProduct(
    IN inSupplierId INT,
    IN inProductId INT
)
BEGIN
    -- exit if the duplicate key occurs
    DECLARE EXIT HANDLER FOR 1062 SELECT 'Duplicate keys error encountered' Message;
    DECLARE EXIT HANDLER FOR SQLEXCEPTION SELECT 'SQLException encountered' Message;
    DECLARE EXIT HANDLER FOR SQLSTATE '23000' SELECT 'SQLSTATE 23000' ErrorCode;

    -- insert a new row into the SupplierProducts
    INSERT INTO SupplierProducts(supplierId,productId)
    VALUES(inSupplierId,inProductId);

    -- return the products supplied by the supplier id
    SELECT COUNT(*)
    FROM SupplierProducts
    WHERE supplierId = inSupplierId;

END$$

DELIMITER ;
```
## Using a named error condition
`DECLARE condition_name CONDITION FOR condition_value;`
```
DROP PROCEDURE IF EXISTS TestProc;

DELIMITER $$

CREATE PROCEDURE TestProc()
BEGIN
    DECLARE TableNotFound CONDITION for 1146 ;

    DECLARE EXIT HANDLER FOR TableNotFound
	SELECT 'Please create table abc first' Message;
    SELECT * FROM abc;
END$$

DELIMITER ;
```
# Raising Error
```
SIGNAL SQLSTATE | condition_name;
SET condition_information_item_name_1 = value_1,
    condition_information_item_name_1 = value_2, etc;
```
```
DELIMITER $$

CREATE PROCEDURE AddOrderItem(
		         in orderNo int,
			 in productCode varchar(45),
			 in qty int,
                         in price double,
                         in lineNo int )
BEGIN
	DECLARE C INT;

	SELECT COUNT(orderNumber) INTO C
	FROM orders
	WHERE orderNumber = orderNo;

	-- check if orderNumber exists
	IF(C != 1) THEN
		SIGNAL SQLSTATE '45000'
			SET MESSAGE_TEXT = 'Order No not found in orders table';
	END IF;
	-- more code below
	-- ...
END
```
