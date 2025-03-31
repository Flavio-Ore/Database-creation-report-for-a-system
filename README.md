# Database Design of the Loan System for the National Library of Peru

> [!IMPORTANT]  
> University work whose proposal was not implemented because it was carried out for practical purposes.

## 📌 Project Overview
This repository contains the **Database Design of the Loan System for the National Library of Peru**, implemented using **SQL Server**. The system ensures efficient management of book loans, returns, sanctions, and user tracking within the library.

## 🚀 Features
- **User and book loan management** using SQL Server
- **Stored procedures** for handling book loans and returns
- **Triggers** for automatic status updates
- **Functions** for retrieving book details efficiently
- **Cursores** for checking table data availability
- **Comprehensive data tracking** for users, books, and fines
- **SQL scripts** for database setup and testing

## 🏗️ Technologies Used
- **SQL Server** (Microsoft SQL Server)
- **T-SQL (Transact-SQL)**
- **SSMS (SQL Server Management Studio)**

## 🛠️ Database Schema
The system consists of the following key tables:

```sql
CREATE DATABASE BIBLIOTECA;

USE BIBLIOTECA;

SET DATEFORMAT DMY;

CREATE TABLE STOCK (
    STOCK_ID INT NOT NULL IDENTITY(1,1) PRIMARY KEY,
    STO_DIS INT NOT NULL DEFAULT 0
);

CREATE TABLE MONTO (
    MONTO_ID INT NOT NULL IDENTITY(1,1) PRIMARY KEY,
    MON_PRE SMALLMONEY NOT NULL
);

CREATE TABLE IDIOMA (
    IDIOMA_ID INT NOT NULL IDENTITY(1,1) PRIMARY KEY,
    IDI_NOM VARCHAR(35) NOT NULL UNIQUE
);

CREATE TABLE CATEGORIA (
    CATEGORIA_ID INT NOT NULL IDENTITY(1,1) PRIMARY KEY,
    CAT_NOM VARCHAR(100) NOT NULL
);

CREATE TABLE LIBRO (
    LIBRO_ID INT IDENTITY(1,1) PRIMARY KEY,
    CATEGORIA_ID INT NOT NULL REFERENCES CATEGORIA(CATEGORIA_ID),
    IDIOMA_ID INT NOT NULL REFERENCES IDIOMA(IDIOMA_ID),
    STOCK_ID INT NOT NULL REFERENCES STOCK(STOCK_ID),
    MONTO_ID INT NOT NULL REFERENCES MONTO(MONTO_ID),
    LIB_NOM VARCHAR(255) NOT NULL,
    LIB_DESC VARCHAR(255) NOT NULL,
    LIB_AUT VARCHAR(255) NOT NULL
);

CREATE TABLE SANCION (
    SANCION_ID INT IDENTITY(1,1) NOT NULL PRIMARY KEY,
    SAN_TIP VARCHAR(255) NOT NULL,
    SAN_MON SMALLMONEY NOT NULL,
    SAN_FEC DATE NOT NULL
);

CREATE TABLE USUARIO (
    USUARIO_ID INT IDENTITY(1,1) NOT NULL PRIMARY KEY,
    SANCION_ID INT REFERENCES SANCION(SANCION_ID),
    USU_NOM VARCHAR(100) NOT NULL,
    USU_APE VARCHAR(100) NOT NULL,
    USU_TEL VARCHAR(100) NOT NULL,
    USU_DNI VARCHAR(14) NOT NULL,
    USU_DIR VARCHAR(255) NOT NULL
);

CREATE TABLE PRESTAMO (
    PRESTAMO_ID INT IDENTITY(1,1) NOT NULL PRIMARY KEY,
    USUARIO_ID INT NOT NULL REFERENCES USUARIO(USUARIO_ID),
    LIBRO_ID INT NOT NULL REFERENCES LIBRO(LIBRO_ID),
    PRE_FEC_RES DATE NOT NULL,
    PRE_FEC_RET DATE NOT NULL
);

CREATE TABLE DETALLE_PRESTAMO (
    DETALLE_ID INT REFERENCES PRESTAMO(PRESTAMO_ID),
    USUARIO_ID INT REFERENCES USUARIO(USUARIO_ID),
    LIBRO_ID INT REFERENCES LIBRO(LIBRO_ID),
    NUM_RESERVACION CHAR(10) UNIQUE NOT NULL
);
```

## SQL Features Implemented
### Stored Procedure for Retrieving User Loaned Books
```sql
CREATE PROCEDURE GetBooksBorrowedByUser
    @UserID INT
AS
BEGIN
    SELECT
        DP.NUM_RESERVACION,
        L.LIB_NOM,
        L.LIB_DESC,
        L.LIB_AUT,
        P.PRE_FEC_RES,
        P.PRE_FEC_RET
    FROM DETALLE_PRESTAMO DP
    INNER JOIN LIBRO L ON DP.LIBRO_ID = L.LIBRO_ID
    INNER JOIN PRESTAMO P ON DP.DETALLE_ID = P.PRESTAMO_ID
    WHERE DP.USUARIO_ID = @UserID;
END;
```

### Trigger for reservation number generation
```sql
CREATE TRIGGER trg_Generate_NumReservacion
ON DETALLE_PRESTAMO
AFTER INSERT
AS
BEGIN
    SET NOCOUNT ON;
    DECLARE @NewNumReservacion CHAR(10);
    UPDATE DETALLE_PRESTAMO
    SET @NewNumReservacion = 'U' + CONVERT(VARCHAR, YEAR(GETDATE())) + RIGHT('00000' + CONVERT(VARCHAR, inserted.DETALLE_ID), 5)
    FROM inserted
    WHERE DETALLE_PRESTAMO.DETALLE_ID = inserted.DETALLE_ID;
    UPDATE DETALLE_PRESTAMO
    SET NUM_RESERVACION = @NewNumReservacion
    FROM DETALLE_PRESTAMO dp
    INNER JOIN inserted i ON dp.DETALLE_ID = i.DETALLE_ID;
END
```

### Function for Retrieving Book Information
```sql
CREATE FUNCTION GetLibroInfo()
RETURNS TABLE
AS
RETURN
(
    SELECT
        L.LIB_NOM,
        L.LIB_DESC,
        L.LIB_AUT,
        C.CAT_NOM AS Categoria,
        I.IDI_NOM AS Idioma,
        S.STO_DIS AS Stock_Disponible,
        M.MON_PRE AS Monto_Prestamo
    FROM LIBRO L
    INNER JOIN CATEGORIA C ON L.CATEGORIA_ID = C.CATEGORIA_ID
    INNER JOIN IDIOMA I ON L.IDIOMA_ID = I.IDIOMA_ID
    INNER JOIN STOCK S ON L.STOCK_ID = S.STOCK_ID
    INNER JOIN MONTO M ON L.MONTO_ID = M.MONTO_ID
);
```

### Cursores for Checking Table Data Availability
```sql
DECLARE @StockCount INT
DECLARE @MontoCount INT

DECLARE StockCursor CURSOR FOR
SELECT COUNT(*) FROM STOCK

OPEN StockCursor
FETCH NEXT FROM StockCursor INTO @StockCount

IF @StockCount <> 0
    PRINT 'STOCK table has data'
ELSE
    PRINT 'STOCK table is empty'

CLOSE StockCursor
DEALLOCATE StockCursor

DECLARE MontoCursor CURSOR FOR
SELECT COUNT(*) FROM MONTO

OPEN MontoCursor
FETCH NEXT FROM MontoCursor INTO @MontoCount

IF @MontoCount <> 0
    PRINT 'MONTO table has data'
ELSE
    PRINT 'MONTO table is empty'

CLOSE MontoCursor
DEALLOCATE MontoCursor
```

## 📂 Installation and Setup
1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/library-loan-system.git
   ```
2. Import the SQL scripts into **SQL Server Management Studio (SSMS)**
3. Execute the **database schema scripts** to create tables and stored procedures
4. Start using the system!

## 📜 License
This project is licensed under the **MIT License**.

## 🤝 Contributing
Feel free to submit pull requests or open issues to improve this project.
