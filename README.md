# Bank-Account-Management-System-Database-Design
Bu senaryoda bir banka hesap yönetim sistemi veritabanı tasarlanacaktır. Sistemde **kişi, çalışan, adres, işçi, müşteri, hesap, hesap türü, işlem, işlem türü, şube, kredi ve kredi türü süreçleri** bulunacaktır. 

#### **1. Tabloların Oluşturulması**
**Aşağıdaki tablolalar oluşturulacaktır:**
* **Addresses (Adresler)**
* **Persons (Kişiler)**
* **Branchs (Şubeler)**
* **Employees (Çalışanlar)**
* **Customers (Müşteriler)**
* **AccountTypes (Hesap Türrleri)**
* **Accounts (Hesaplar)**
* **Transfers (Transferler)**
* **TransactionTypes (İşlem Türleri)**
* **Transactions (İşlemler)**
* **LoanTypes (Kredi Türleri)**
* **Loans (Krediler)**

#### 2. ER Diyagramı
`Crow's Foot Notation ile ER Diyagramı`
![[BankAcc_ER.drawio.png]]
#### **3. Zor Seviye SQL Sorguları**
* Belirli bir tarih aralığında yapılan işlemleri listeleme.
* Hangi müşterinin hangi işlemleri yaptığını listeleme.
* En çok işlem gören 5 hesabı sıralama.

#### **4. Trigger Örnekleri**
* Accounts tablosuna veri eklendiğinde müşteri durum otomatik olarak aktif olsun.
* 

#### **5. Stored Procedure Örnekleri**
* Girilen bir müşterinin yaptığı son işlemi gösteren bir procedure.
* Hesaplar arasında para transferi gerçekleştiren bir procedure (T-SQL). 

#### **6. View Kullanımı**
* Her müşterinin yaptığı son işlemi listeleyen bir view.
* Her çalışanı ve bağlı oldukları şubeleri listeleyen bir view.

----------------------------------
# 1. Tabloların Oluşturulması

`Addresses`
```sql
CREATE TABLE Addresses (
    AddressID INT PRIMARY KEY,
	Country VARCHAR(20) NOT NULL,
	City VARCHAR(20) NOT NULL,
	Street VARCHAR(20) NOT NULL,
	PostalCode VARCHAR(20) NOT NULL
);
```

`Persons`
```sql
CREATE TABLE Persons (
     PersonID INT PRIMARY KEY,
	 FirstName VARCHAR(20) NOT NULL,
	 LastName VARCHAR(20) NOT NULL,
	 DateOfBirth DATE NOT NULL,
	 Email VARCHAR(100) UNIQUE NOT NULL,
	 PhoneNumber VARCHAR(11) UNIQUE NOT NULL
	    CHECK (LEN(PhoneNumber) = 11),
	 AddressID INT NOT NULL,
	 FOREIGN KEY(AddressID) REFERENCES Addresses(AddressID)
);
```

`Branchs`
```sql
CREATE TABLE Branchs (
    BranchID INT PRIMARY KEY,
	BranchName VARCHAR(20) NOT NULL,
	AddressID INT NOT NULL,
	PhoneNumber VARCHAR(11) NOT NULL
	   CHECK (LEN(PhoneNumber) = 11),
	FOREIGN KEY(AddressID) REFERENCES Addresses(AddressID)
);
```

``Employees``
```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
	PersonID INT NOT NULL,
	Position VARCHAR(20) NOT NULL,
	BranchID INT NOT NULL,
	FOREIGN KEY(BranchID) REFERENCES Branchs(BranchID),
	FOREIGN KEY(PersonID) REFERENCES Persons(PersonID)
);
```

`Customers`
```sql
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
	PersonID INT NOT NULL,
	CustomerType VARCHAR(20) NOT NULL,
	FOREIGN KEY(PersonID) REFERENCES Persons(PersonID)
);
```

``AccountTypes``
```sql
CREATE TABLE AccountTypes (
    AccountTypeID INT PRIMARY KEY,
	TypeName VARCHAR(20) UNIQUE NOT NULL
	   CHECK (TypeName in('Vadeli','Vadesiz','Kredi','Yatırım')),
);
```

``Accounts``
```sql
CREATE TABLE Accounts (
    AccountID INT PRIMARY KEY,
	CustomerID INT NOT NULL,
	BranchID INT NOT NULL,
	AccountNumber INT UNIQUE NOT NULL,
	CurrentBalance DECIMAL(10,2) NOT NULL
	   CHECK (CurrentBalance >= 0),
	CreateDate DATE NOT NULL,
	AccountTypeID INT NOT NULL,
    AccountStatus VARCHAR(20) NOT NULL 
	   CHECK (AccountStatus IN ('Aktif', 'Aktif Değil', 'Kapalı')),
	FOREIGN KEY(AccountTypeID) REFERENCES AccountTypes(AccountTypeID),
	FOREIGN KEY(BranchID) REFERENCES Branchs(BranchID),
	FOREIGN KEY(CustomerID) REFERENCES Customers(CustomerID)
);
```

``Transfers``
```sql
CREATE TABLE Transfers (
    TransferID INT PRIMARY KEY,
	FromAccountID INT NOT NULL,
	ToAccountID INT NOT NULL,
	Amount DECIMAL(10,2) NOT NULL
	   CHECK (Amount > 0),
	Descriptions VARCHAR(150)
);
```

``TransactionTypes``
```sql
CREATE TABLE TransactionTypes (
    TransactionTypeID INT PRIMARY KEY,
	TransactionName VARCHAR(20) UNIQUE NOT NULL
	   CHECK (TransactionName IN ('Para Yatırma','Para Çekme','Havale','Ödeme'))
);
```

``Transactions``
```sql
CREATE TABLE Transactions (
    TransactionID INT PRIMARY KEY,
	AccountID INT NOT NULL,
	TransactionTypeID INT NOT NULL,
	Amount DECIMAL(10,2) NOT NULL
	   CHECK (Amount > 0),
	TransactionDate DateTime,
	FOREIGN KEY(TransactionTypeID) REFERENCES TransactionTypes(TransactionTypeID),
 	FOREIGN KEY(AccountID) REFERENCES Accounts(AccountID)
);
```

``LoanTypes``
```sql
CREATE TABLE LoanTypes (
    LoanTypeID INT PRIMARY KEY,
	LoanName VARCHAR(20) UNIQUE NOT NULL
);
```

``Loans``
```sql
CREATE TABLE Loans (
    LoanID INT PRIMARY KEY,
	LoanTypeID INT NOT NULL,
	CustomerID INT NOT NULL,
	Amount DECIMAL(10,2) NOT NULL
	   CHECK (Amount > 0),
	StartDate DATE NOT NULL,
	EndDate DATE NOT NULL,
	FOREIGN KEY(LoanTypeID) REFERENCES LoanTypes(LoanTypeID),
	FOREIGN KEY(CustomerID) REFERENCES Customers(CustomerID)
);
```


### Primary Key ve Unique Index Tanımlamaları: 
``Addresses Tablosu:`` 
- Primary Key: ``AddressID``  - Her adres için benzersiz kimlik.
- Unique Index: Yok, çünkü aynı sokak gibi değerler farklı kişiler için tanımlanabilir.
``Persons Tablosu:``
- Primary Key: ``PersonID`` - Her kişi için benzersiz kimlik.
- Unique Index: ``(Email, PhoneNumber)`` - Her kişi için kendine ait email ve telefon numarası.
``Branchs Tablosu:``    
- Primary Key: ``BranchID`` - Her şube için benzersiz kimlik.
- Unique Index: Yok, çünkü farklı şehirlerde aynı şube adı olabilir. 
``Employees Tablosu:``
- Primary Key: ``EmployeeID`` -  Her çalışan için benzersiz kimlik. 
* Unique Key: ``PersonID`` - Bir kişi yalnızca bir kez çalışan olarak gösterilebilir. 
``Customers Tablosu:``
- Primary Key: ``CustomerID``  - Her müşteri için benzersiz kimlik.
- Unique Key: ``PersonID`` - Bir kişi yalnızca bir kez müşteri olarak gösterilebilir.
``AccountTypes Tablosu:``
- Primary Key: ``AccountTypeID`` - Her hesap türü için benzersiz kimlik. 
- Unique Key: ``TypeName`` - Her hesap türünün ismi benzersiz olmalı.
``Account Tablosu:``    
- Primary Key: ``AccountID`` - Her hesap için benzersiz kimlik. 
- Unique Key: ``AccountNumber`` - Her hesabın numarası farklı olmalıdır. 
``Transfers Tablosu:``
- Primary Key: ``TransferID`` - Her transfer için benzersiz kimlik. 
- Unique Key: Yok, çünkü aynı hesap arasında birden fazla transfer mümkündür.
``TransactionTypes Tablosu:``
-  Primary Key: ``TransactionTypeID`` - Her işlem türü için benzersiz kimlik. 
- Unique Key: ``TransactionName`` - işlem türü isimleri benzersiz olmalıdır.
``Transactions Tablosu:``
- Primary Key: ``TransactionID`` - Her işlem için benzersiz kimlik.
- Unique Key: Yok, çünkü aynı hesaptan birden fazla işlem gerçekleşebilir. 
``LoanTypes:``
- Primary Key: ``LoanTypeID`` - Her kredi türü için benzersiz kimlik.
- Unique Key: ``TypeName`` - Her kredi türü adı benzersiz olmalıdır.
``Loans Tablosu:``
- Primary Key: ``LoanID`` - Her kredi için benzersiz kimlik. 
- Unique Key: Yok, çünkü bir müşteri birden fazla kredi çekebilir.


# 3. Zor Seviye SQL Sorguları
#### ``1. Belirli bir tarih aralığında yapılan işlemleri listeleme.``
```sql 
SELECT t.TransactionID, t.AccountID, t.Amount, 
t.TransactionDate, tt.TransactionName
FROM Transactions t
JOIN TransactionTypes tt
ON t.TransactionTypeID = tt.TransactionTypeID
WHERE TransactionDate BETWEEN '2023-05-12' AND '2023-05-15';
```
#### ``2. Hangi müşterinin hangi işlemleri yaptığını listeleme.``
```sql
SELECT c.CustomerID, c.CustomerType, p.FirstName,
p.LastName, a.AccountNumber, tt.TransactionName, 
t.TransactionDate, t.Amount
FROM Persons p
JOIN Customers c ON p.PersonID = c.PersonID
JOIN Accounts a ON c.CustomerID = a.CustomerID
JOIN Transactions t ON t.AccountID = a.AccountID
JOIN TransactionTypes tt ON t.TransactionTypeID = tt.TransactionTypeID;
```
#### ``3. En çok işlem gören 5 hesabı sıralama.``
```sql
SELECT TOP 5 a.AccountNumber, a.AccountStatus, a.CreateDate,
a.CurrentBalance, COUNT(TransactionID) as TotalTransaction
FROM Accounts a
JOIN AccountTypes aType 
ON a.AccountTypeID = aType.AccountTypeID
JOIN Transactions t 
ON a.AccountID = t.AccountID
JOIN TransactionTypes tt
ON t.TransactionTypeID = tt.TransactionTypeID
GROUP BY a.AccountNumber, a.AccountStatus, a.CreateDate,
a.CurrentBalance
ORDER BY TotalTransaction DESC;
```

# 4. Trigger Örnekleri
#### `1. Accounts tablosuna veri eklendiğinde müşteri durum otomatik olarak "aktif" olarak yazan trigger.` 
```sql
CREATE TRIGGER AccountStatus
ON Accounts
AFTER INSERT
AS
BEGIN
    UPDATE Accounts
    SET AccountStatus = 'Aktif'
    FROM Accounts a
    JOIN inserted i ON a.AccountID = i.AccountID
    WHERE i.AccountStatus IS NULL;
END;
```
#### `2. Transfers tablosuna yeni transfer eklenince açıklama boşsa "Genel Transfer" yazan trigger.` 
```sql
CREATE TRIGGER TransferDescription
ON Transfers
AFTER INSERT
AS
BEGIN
    UPDATE Transfers
    SET Descriptions = 'Genel Transfer'
    FROM Transfers t
    JOIN inserted i ON t.TransferID = i.TransferID
    WHERE i.Descriptions IS NULL;
END;
```

# 5. Stored Procedure Örnekleri
#### ``1. Girilen bir müşterinin son yaptığı işlemi getiren bir procedure.``
```sql
CREATE PROCEDURE CustomerLastTransactionInfo
@CustomerID INT
as
BEGIN
   SELECT TOP 1 p.FirstName + ' ' + p.LastName as FullName,
   c.CustomerID, a.AccountNumber, t.TransactionID ,t.Amount, t.TransactionDate,
   tt.TransactionName
   FROM Persons p
   JOIN Customers c ON p.PersonID = c.PersonID
   JOIN Accounts a ON c.CustomerID = a.CustomerID
   JOIN Transactions t ON a.AccountID = t.AccountID
   JOIN TransactionTypes tt ON t.TransactionTypeID = tt.TransactionTypeID
   WHERE c.CustomerID = @CustomerID
   ORDER BY t.TransactionDate DESC;
END;
--Kullanımı
EXEC CustomerLastTransactionInfo 68523
```

#### `2. Hesaplar arasında para transferi gerçekleştiren bir procedure (T-SQL).`
```sql
CREATE PROCEDURE TransferMoney
@FromAccountID INT,
@ToAccountID INT,
@Amount DECIMAL(10,2)
AS
BEGIN 
   BEGIN TRY
      BEGIN TRANSACTION

	  UPDATE Accounts 
	  SET CurrentBalance = CurrentBalance - @Amount
	  WHERE AccountID = @FromAccountID

	  UPDATE Accounts 
	  SET CurrentBalance = CurrentBalance + @Amount
	  WHERE AccountID = @ToAccountID
	
      COMMIT;
   END TRY
   BEGIN CATCH
   
      ROLLBACK;
	  PRINT 'Hata oluştu: ' + ERROR_MESSAGE();
   
   END CATCH
END;
--Kullanımı
EXEC TransferMoney 15000, 16000, 300.00
```

# 6. View Kullanımı
#### `1. Her müşterinin yaptığı son işlemi getiren bir view.`
```sql
CREATE VIEW CustomersLastTransactionInfo
AS
SELECT p.FirstName + ' ' + p.LastName as FullName,
c.CustomerID, a.AccountNumber, t.Amount,
t.TransactionDate, tt.TransactionName
FROM Persons p
JOIN Customers c ON p.PersonID = c.PersonID
JOIN Accounts a ON c.CustomerID = a.CustomerID
Outer Apply (
      SELECT TOP 1 t.*
	  FROM Transactions t
	  WHERE t.AccountID = a.AccountID
	  ORDER BY TransactionDate DESC) t
JOIN TransactionTypes tt ON t.TransactionTypeID = tt.TransactionTypeID;
--Kullanımı
SELECT * FROM CustomerLastTransactionInfo
```
#### `2. Her çalışanı ve bağlı oldukları şubeleri listeleyen bir view.`
```sql
create view EmployeeInfo
as
SElECT p.FirstName, p.LastName, p.PhoneNumber,
p.Email, e.Position, b.BranchName, b.PhoneNumber
as BranchPhone
FROM Persons p
JOIN Employees e ON p.PersonID = e.PersonID
JOIN Branchs b ON e.BranchID = b.BranchID;
--Kullanımı
SELECT * FROM EmployeeInfo;
```

# 7. Normalizasyon Süreçleri
#### `1. 1NF (Birinci Normal Form)` 
Senaryodaki her tablonun hücreleri atomik (bölünemez) veriler içerir. Bu nedenle tüm tablolar 1NF sürecine uygundur.

#### `2NF (İkinci Normal Form)` 
Bütün tablolar 1NF sürecini sağlamıştır. Tablodaki tüm kolonlar tamamiyle birincil anahtara bağımlıdır. 
`Addresses`
- Adres bilgilerinin her biri, `AddressID` ile tanımlanır. Her bilgi tek ve bölünemez şekilde tutulmuştur. Kısmi bağımlılık yoktur.
`Persons`
- Her kişi bilgisi yalnızca `PersonID` ile tanımlanır. Tüm sütunlar anahtara doğrudan bağlıdır.
`Branchs`
- Şube bilgileri yalnızca `BranchID` ile belirlenir. Her bilgi tekil anahtara tam bağımlıdır.
`Employees`
- Her çalışan `EmployeeID` ile tanımlanır. Pozisyon bilgisi gibi veriler sadece bu anahtara bağlıdır. `PersonID` ve `BranchID` yabancı anahtar olarak ilişkilendirilmiştir.
`Customers`
- Müşteri bilgileri `CustomerID` ile doğrudan tanımlanır. `PersonID` ile olan ilişki yabancı anahtar kullanılarak kurulmuştur ve veri tekrarını engeller.
`AccountTypes`
- Hesap türü bilgileri `AccountTypeID` üzerinden tanımlanır. `TypeName` gibi bilgiler doğrudan bu anahtara bağlıdır.
`Accounts`
- Hesapta tüm bilgiler tekil olarak `AccountID` ile belirlenir. Diğer alanlar (müşteri, şube, tür bilgileri) yabancı anahtarlarla ilişkilendirilmiştir.
`Transfers`
- Her transfer işlemi `TransferID` ile tanımlanır. Gönderen ve alıcı hesaplar ile açıklama bilgisi, bu anahtara bağımlıdır.
`TransactionTypes`
- İşlem türleri `TransactionTypeID` ile tanımlanır. `TransactionName`, doğrudan bu anahtara bağlıdır.
`Transactions`
- İşlem bilgileri yalnızca `TransactionID` ile belirlenir. Hesap ve işlem türü bilgileri yabancı anahtarlar ile tanımlanmıştır.
`LoanTypes`
- Her kredi türü `LoanTypeID` ile belirlenmiştir. İsim bilgisi bu anahtara bağlıdır.
`Loans`
- Kredi bilgileri `LoanID` ile tanımlanır. Kredi türü ve müşteri bilgileri yabancı anahtar olarak ilişkilendirilmiştir.
### `3NF (Üçüncü Normal Form)`
Bütün tablolar 2NF sürecini sağlamıştır. Ve burada transitif (dolaylı) bağımlılıklar kaldırılmıştır.
`Addresses`
* Her sütun doğrudan `AddressID`’ye bağlı. Transitif bağımlılık yok.
`Persons`
* Tüm bilgiler `PersonID`’ye doğrudan bağlı. `AddressID` yabancı anahtar. Her şey ayrı tutulmuştur.
`Branchs`
* Tüm bilgiler `PersonID`’ye doğrudan bağlı. `AddressID` yabancı anahtar. Her şey ayrı tutulmuş.
`Employees`
* Pozisyon gibi bilgiler doğrudan `EmployeeID`’ye bağlı. Başka sütuna bağımlı alan yok.
`Customers`
* `CustomerType`, sadece `CustomerID`’ye bağlı. `PersonID` yabancı anahtarla verilmiş.
`AccountTypes`
* `TypeName`, sadece `AccountTypeID`’ye bağlı. Başka sütuna bağımlı değil.
`Accounts`
* Tüm alanlar `AccountID` ile doğrudan ilişkili. Yabancı anahtarlar net tanımlı.
`Transfers`
- Her bilgi `TransferID`’ye bağlı. İki hesap ayrı ayrı tanımlı. Bağımlılık yok.
`TransactionTypes`
- İşlem adı sadece `TransactionTypeID` ile tanımlanıyor. Ek bağımlılık yok.
`Transactions`
- Tüm bilgiler doğrudan `TransactionID`’ye bağlı. Hesap ve tür yabancı anahtarla tutulmuş.
`LoanTypes`
- Her kredi tipi `LoanTypeID` ile belirleniyor. Transitif ilişki yok.
`Loans`
- Tutar, tarih gibi bilgiler sadece `LoanID`’ye bağlı. Diğerleri yabancı anahtar.

--> Sonuç olarak bütün tablolar 3NF'e kadar uyumlu olduğunu görebiliriz.
