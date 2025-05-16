# Add-Edit Operation in ASP.NET Core MVC

**Prerequisite**: Procedures for Insert, Update and select by primary key.

### Insert Procedure

```sql
CREATE OR ALTER PROCEDURE PR_COUNTRY_INSERT
	@CountryName	Varchar(100),
	@CountryCode	Varchar(50),
	@UserID			Int
AS
BEGIN
	INSERT INTO [dbo].[Country](CountryName,CountryCode,UserID)
	VALUES (@CountryName,@CountryCode,@UserID)
END
```

### Update Procedure

```sql
CREATE OR ALTER PROCEDURE PR_COUNTRY_UPDATE
	@CountryID		Int,
	@CountryName	Varchar(100),
	@CountryCode	Varchar(50)
AS
BEGIN
	UPDATE [dbo].[Country]
	SET	CountryName = @CountryName,
		CountryCode = @CountryCode
	WHERE CountryID = @CountryID
END
```

### Select By Primary Key Procedure

```sql
CREATE OR ALTER PROCEDURE PR_COUNTRY_SELECTBYID
	@CountryID INT 
AS
BEGIN 
	SELECT C.CountryID,
		C.CountryName,
        C.CountryCode,
        C.CreatedDate,
        C.UserID
    FROM [dbo].[Country] C 
	WHERE C.CountryID = @CountryID
END
```
<br>

## Step 1: Update the Logic for Insert-Update Operations in the `CountryAddEdit` Action Method

```csharp
public IActionResult CountryAddEdit(CountryModel model)
{
    if (ModelState.IsValid)
    {
        string connectionString = this._configuration.GetConnectionString("ConnectionString");
        SqlConnection connection = new SqlConnection(connectionString);
        connection.Open();
        SqlCommand command = connection.CreateCommand();
        command.CommandType = CommandType.StoredProcedure;

        if (model.CountryID == 0)
        {
            command.CommandText = "PR_COUNTRY_INSERT";
        }
        else
        {
            command.CommandText = "PR_COUNTRY_UPDATE";
            command.Parameters.Add("@CountryID", SqlDbType.Int).Value = model.CountryID;
        }
        command.Parameters.Add("@CountryName", SqlDbType.VarChar).Value = model.CountryName;
        command.Parameters.Add("@CountryCode", SqlDbType.VarChar).Value = model.CountryCode;
        command.Parameters.Add("@UserID", SqlDbType.Int).Value = CommonVariable.UserID();
        command.ExecuteNonQuery();
        return RedirectToAction("CountryList");
    }

    return View("CountryForm", model);
}
```

## Step 2: Verify Insert Operation

<br>

## Step 3: Update Operation (Two Parts)

### Part 1: Fetch and Populate Existing Data in the Text Fields

To retrieve the existing country data and display it in the form for editing, update the `CountryForm` method:

```csharp
public IActionResult CountryForm(int? countryID)
{
    if (countryID == null)
    {
        var m = new CountryModel
        {
            CreationDate = DateTime.Now
        };
        return View(m);
    }

    string connectionString = this._configuration.GetConnectionString("ConnectionString");
    SqlConnection connection = new SqlConnection(connectionString);
    connection.Open();
    SqlCommand command = connection.CreateCommand();
    command.CommandType = CommandType.StoredProcedure;
    command.CommandText = "PR_COUNTRY_SELECTBYID";
    command.Parameters.AddWithValue("@CountryID", countryID);
    command.Parameters.AddWithValue("@UserID", CommonVariable.UserID());
    SqlDataReader reader = command.ExecuteReader();
    DataTable table = new DataTable();
    table.Load(reader);
    CountryModel model = new CountryModel();

    foreach (DataRow dataRow in table.Rows)
    {
        model.CountryName = dataRow["CountryName"].ToString();
        model.CountryCode = dataRow["CountryCode"].ToString();
    }
    return View(model);
}
```

### Part 2: Update the Country Data in the Database

The logic for updating the country data has already been implemented in the `CountryAddEdit` method.

<br>

## Step 4: Add Edit Button with Link in the List Page

```csharp
<a asp-controller="Country" asp-action="CountryForm" asp-route-CountryID="@dataRow["CountryID"]" class="btn btn-primary btn-sm" style="padding: 2px 6px; font-size: 12px;">
    <i class="fa fa-pencil"></i>
</a>
```

<br>

## Step 5: Verify Update Operation
