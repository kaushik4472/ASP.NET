# Database Schema for all tables

## User Table

```sql
CREATE TABLE [User] (
    UserID INT PRIMARY KEY IDENTITY(1,1),
    UserName NVARCHAR(100) NOT NULL,
    MobileNo NVARCHAR(10) NOT NULL,
    Email NVARCHAR(50) NOT NULL,
    CreatedDate DATETIME NOT NULL DEFAULT GETDATE(),
    ModifiedDate DATETIME NULL
);
```
<br>

## Country Table
```sql
CREATE TABLE Country (
    CountryID INT PRIMARY KEY IDENTITY(1,1),
    CountryName NVARCHAR(100) NOT NULL,
    CountryCode NVARCHAR(10) NOT NULL,
    CreatedDate DATETIME NOT NULL DEFAULT GETDATE(),
    ModifiedDate DATETIME NULL,
UserID INT NOT NULL,
    FOREIGN KEY (UserID) REFERENCES [User](UserID)
);
```
<br>

## State Table
```sql
CREATE TABLE [State] (
    StateID INT PRIMARY KEY IDENTITY(1,1),
    CountryID INT NOT NULL,
    StateName NVARCHAR(100) NOT NULL,
    StateCode NVARCHAR(10),
    CreatedDate DATETIME NOT NULL DEFAULT GETDATE(),
    ModifiedDate DATETIME NULL,
UserID INT NOT NULL,
    FOREIGN KEY (CountryID) REFERENCES Country(CountryID),
    FOREIGN KEY (UserID) REFERENCES [User](UserID)
);
```
<br>

## City Table
```sql
CREATE TABLE City (
    CityID INT PRIMARY KEY IDENTITY(1,1),
    StateID INT NOT NULL,
    CountryID INT NOT NULL,
    CityName NVARCHAR(100) NOT NULL,
    CityCode NVARCHAR(10),
    CreatedDate DATETIME NOT NULL DEFAULT GETDATE(),
    ModifiedDate DATETIME NULL,
UserID INT NOT NULL,
    FOREIGN KEY (StateID) REFERENCES [State](StateID),
    FOREIGN KEY (CountryID) REFERENCES Country(CountryID),
    FOREIGN KEY (UserID) REFERENCES [User](UserID)
);
```
<br>


