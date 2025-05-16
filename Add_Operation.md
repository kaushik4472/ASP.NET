# Add Operation in ASP.NET Core MVC

**Prerequisite**: Procedures for Insert and a data Model is required for specific table.

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
### Country Model

```csharp
    public class CountryModel
    {
        [Key]
        public int CountryID { get; set; }

        [Required(ErrorMessage = "Country name is required")]
        [StringLength(30, ErrorMessage = "Country name cannot exceed 30 characters")]
        public string CountryName { get; set; }

        [Required(ErrorMessage = "Country code is required")]
        [StringLength(3, ErrorMessage = "Country code cannot exceed 3 characters")]
        public string CountryCode { get; set; }

        public DateTime CreatedDate { get; set; } = DateTime.Now;

        public DateTime? ModifiedDate { get; set; } = DateTime.Now;

        [Required(ErrorMessage = "UserID is required")]
        public int UserID { get; set; }

    }
```


<br>


## Step 1: Create an Action Method named `CountryAddEdit` in your controller and implement the below logic.

```csharp
[HttpPost]
public IActionResult CountryAddEdit(CountryModel model)
{
    if (ModelState.IsValid)
    {
        try
        {
                    string connectionString = this._configuration.GetConnectionString("ConnectionString");

                    SqlConnection connection = new SqlConnection(connectionString)
            
                    connection.Open();
                    SqlCommand command = connection.CreateCommand();
                    command.CommandType = CommandType.StoredProcedure;
                    command.CommandText = "PR_COUNTRY_INSERT";

                    command.Parameters.Add("@CountryName", SqlDbType.NVarChar).Value = model.CountryName;
                    command.Parameters.Add("@CountryCode", SqlDbType.NVarChar).Value = model.CountryCode;
                    command.Parameters.Add("@UserID", SqlDbType.Int).Value = model.UserID;

                    command.ExecuteNonQuery();
            }

            return RedirectToAction("CountryList");
        }
       catch (Exception ex)
        {
            ViewBag.ErrorMessage = "Something went wrong while saving country data.";
            return View("CountryForm", model);
        }
    return View("CountryForm", model);
}
```
**View Code***

```csharp
@if (ViewBag.ErrorMessage != null)
{
    <div class="alert alert-danger">
        @ViewBag.ErrorMessage
    </div>
}
```

<br>


### Step 2: Fetch and Populate Existing Data in the Text Fields

To retrieve the existing country data and display it in the form for editing, update the `CountryForm` method:

```csharp
public IActionResult CountryForm()
{
    return View();
}
```

<br>


## Step 3: Add Edit Button with Link in the List Page

```csharp
<a asp-controller="Country" asp-action="CountryForm" class="btn btn-primary btn-sm" style="padding: 2px 6px; font-size: 12px;">
    ADD
</a>
```
