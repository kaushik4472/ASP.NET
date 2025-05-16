# Export to Excel Function in ASP.NET Core

This function exports data from a database into an Excel file using the `EPPlus` library. The Excel file includes headers and data rows from a stored procedure.

## Function: `ExportToExcel`

### Description

The `ExportToExcel` function retrieves data from the database using a stored procedure (`PR_City_SelectAll`) and exports it to an Excel file. It uses the `EPPlus` library to create the Excel file, with the file returned as a downloadable response.

### Prerequisites

1. **EPPlus NuGet Package**  
   Ensure the `EPPlus` library is installed in your project:
   ```bash
   Install-Package EPPlus

##  Write Logic to Download Excel File in the List Page Action Method

In the action method for your list page, add the following logic to fetch data from the SQL Server database:

```csharp
public IActionResult ExportToExcel()
{
    string connectionString = _configuration.GetConnectionString("ConnectionString");
    SqlConnection sqlConnection = new SqlConnection(connectionString);
    sqlConnection.Open();

    SqlCommand sqlCommand = sqlConnection.CreateCommand();
    sqlCommand.CommandType = System.Data.CommandType.StoredProcedure;
    sqlCommand.CommandText = "PR_COUNTRY_SELECTALL";
    /*sqlCommand.Parameters.Add("@UserID", SqlDbType.Int).Value = CommonVariable.UserID();*/

    SqlDataReader sqlDataReader = sqlCommand.ExecuteReader();
    DataTable data = new DataTable();
    data.Load(sqlDataReader);

    // Set the license context
    ExcelPackage.LicenseContext = LicenseContext.NonCommercial;

    using (var package = new ExcelPackage())
    {
        var worksheet = package.Workbook.Worksheets.Add("Country DataSheet");

        // Add headers
        worksheet.Cells[1, 1].Value = "CountryID";
        worksheet.Cells[1, 2].Value = "CountryName";
        worksheet.Cells[1, 3].Value = "CountryCode";
        worksheet.Cells[1, 4].Value = "CreatedDate";
        worksheet.Cells[1, 4].Value = "ModifiedDate";


        // Add data
        int row = 2;
        foreach (DataRow item in data.Rows)
        {
            worksheet.Cells[row, 1].Value = item["CountryID"];
            worksheet.Cells[row, 2].Value = item["CountryName"];
            worksheet.Cells[row, 3].Value = item["CountryCode"];
            worksheet.Cells[row, 4].Value = item["CreatedDate"];
            worksheet.Cells[row, 5].Value = item["ModifiedDate"];
            row++;
        }

        var stream = new MemoryStream();
        package.SaveAs(stream);
        stream.Position = 0;

        string excelName = $"Data-{DateTime.Now:yyyyMMddHHmmss}.xlsx";
        return File(stream, "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", excelName);
    }
}
```
