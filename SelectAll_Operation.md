# Select All Operation in ASP.NET Core MVC

**Prerequisite**: Select All Procedure required for specific table. 

### Select All Procedure

```sql
CREATE OR ALTER PROCEDURE PR_COUNTRY_SELECTALL
AS
BEGIN 
	SELECT Country.CountryID,
		Country.CountryName,
        Country.CountryCode,
        Country.CreatedDate,
        Country.UserID,
		[User].UserID,
        [User].UserName,
        [User].MobileNo,
        [User].Email,
        [User].CreatedDate		
    FROM [dbo].[Country] JOIN [dbo].[User] 
	ON Country.UserID = [User].UserID
END
```

<br>

## Step 1: Add static data in your database. 

```sql
INSERT INTO [State] (CountryID, StateName, StateCode, UserID)
VALUES 
(1, 'California', 'CA', 1),
(1, 'Texas', 'TX', 2),
(2, 'Ontario', 'ON', 3),
(2, 'Quebec', 'QC', 4),
(3, 'Jalisco', 'JA', 5),
(4, 'England', 'ENG', 6),
(5, 'Bavaria', 'BY', 7),
(6, 'Île-de-France', 'IDF', 8),
(7, 'Maharashtra', 'MH', 9),
(8, 'Guangdong', 'GD', 10);
```

<br>

## Step 2: Configure the Connection String in `appsettings.json`

### For Windows Users:

Add the connection string for your SQL Server database in the `appsettings.json` file as shown below:

```json
"ConnectionStrings": {
    "ConnectionString": "Data Source=SQL Server Name;Initial Catalog=DatabaseName;Integrated Security=true;"
}
```

**Example:**

```json
"ConnectionStrings": {
    "ConnectionString": "Data Source=LAPTOP-LBMAFD6U\\SQLEXPRESS;Initial Catalog=StudentMaster;Integrated Security=true;"
}
```

### For Mac Users:

For Mac users, the connection string should include the user ID and password:

```json
"ConnectionStrings": {
    "ConnectionString": "Data Source=SQL Server Name;Initial Catalog=DatabaseName;User id=userID; password=Password;"
}
```

**Example:**

```json
"ConnectionStrings": {
    "ConnectionString": "Data Source=localhost;Initial Catalog=Practice;User id=SA; password=MyStrongPass123;"
}
```

<br>


## Step 3: Set Up the Configuration Variable in the Controller

In your controller, declare a configuration variable and initialize it using the constructor. This will allow you to access the connection string from the configuration.

```csharp
private IConfiguration configuration;

public ProductController(IConfiguration _configuration)
{
    configuration = _configuration;
}
```

<br>

## Step 4: Install the `System.Data.SqlClient` Package

Install the `System.Data.SqlClient` package from the NuGet Package Manager to enable SQL Server connectivity.

- Navigate to **Tools** -> **NuGet Package Manager** -> **Manage NuGet Packages for Solution**.
- Ensure the project is not running during the installation.

<br>

## Step 5: Write Logic to Fetch Data in the List Page Action Method

In the action method for your list page, add the following logic to fetch data from the SQL Server database:

```csharp
public IActionResult CountryList()
{
    string connectionString = this._configuration.GetConnectionString("ConnectionString");
    SqlConnection connection = new SqlConnection(connectionString);
    connection.Open();
    SqlCommand command = connection.CreateCommand();
    command.CommandType = CommandType.StoredProcedure;
    command.CommandText = "PR_COUNTRY_SELECTALL";
    SqlDataReader reader = command.ExecuteReader();
    DataTable table = new DataTable();
    table.Load(reader);
    return View(table);
}
```

<br>

## Step 6: Display the Data in the View

In your view page, import the `DataTable` and use a `foreach` loop to iterate through the data and display it.

```csharp
@model DataTable
@using System.Data

<table class="table">
    <thead>
        <tr>
            <th>CountryID</th>
            <th>CountryName</th>
            <th>CountryCode</th>
            <th>UserID</th>
            <th>CreatedDate</th>
        </tr>
    </thead>
    <tbody>
        @foreach (DataRow datarow in Model.Rows)
        {
            <tr>
                <td>@dataRow["CountryID"]</td>
                <td>@dataRow["CountryName"]</td>
                <td>@dataRow["CountryCode"]</td>
                <td>@dataRow["UserID"]</td>
                <td>@dataRow["CreatedDate"]</td>
            </tr>
        }
    </tbody>
</table>
```
