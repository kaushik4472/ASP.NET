# Steps on How to Upload a file
**Prerequisite:** Database with Table and Stored Procedure to store file related information
### Step 1: Prepare a Model class
Here we have created a FileUploadModel class with only required properties. You can modify model class as per your requirement.
```csharp
public class FileUploadModel
  {
    public int FileID { get; set; }
    public string FilePath { get; set; }
    public IFormFile File { get; set; }
  }
```
### Step 2: Create a `HomeController` class with required action methods
Here we have created two action methods one for view page and another one to store uploaded file in database.
  - Index() action method
    ```csharp
    public IActionResult Index(int? FileID)
        {
            if (FileID != null)
            {
                FileUploadModel modelFileUpload = new FileUploadModel();
                SqlConnection objConn = new SqlConnection(this.Configuration.GetConnectionString("ABConnectionString"));
                objConn.Open();
                SqlCommand objCmd = objConn.CreateCommand();
                objCmd.CommandType = CommandType.StoredProcedure;
                objCmd.CommandText = "PR_FileUpload_SelectByPK";
                objCmd.Parameters.AddWithValue("FileUploadID", FileID);
                SqlDataReader objSDR = objCmd.ExecuteReader();
                if (objSDR != null && objSDR.HasRows)
                {
                    DataTable dt = new DataTable();
                    dt.Load(objSDR);
                    foreach (DataRow dr in dt.Rows)
                    {
                        modelFileUpload.FilePath = dr["FilePath"].ToString();
                    }
                    //Here we have stored file path to temp data temporary which we will use later on.
                    TempData["FilePath"] = modelFileUpload.FilePath.ToString();
                }
                return View(modelFileUpload);
            }
            return View();
        }
    ```

  - Save() action method
    ```csharp
    [HttpPost]
        public IActionResult Save(FileUploadModel model_FileUpload)
        {
            using (SqlConnection objConn = new SqlConnection(this.Configuration.GetConnectionString("ABConnectionString")))
            {
                objConn.Open();
                using (SqlCommand objCmd = objConn.CreateCommand())
                {
                    try
                    {
                        //check if record contains file id?
                        //If contains then check file exists or not.
                        //If not it means user does not selected any file during edit operation.
                        //So pass existing file path.
                        if (model_FileUpload.FileID != 0)
                        {
                            if (model_FileUpload.File == null && TempData["FilePath"] != null)
                                model_FileUpload.FilePath = TempData["FilePath"].ToString();
                        }
                        //check if model_FileUpload.File contains any file
                        if (model_FileUpload.File != null)
                        {
                            //determine uploaded file type (i.e. .docx, .pdf, .jpg, etc.) and create folder according to file type
                            string filePath = System.IO.Path.GetExtension(model_FileUpload.File.FileName);
                            //Define the physical directory path to store the file to particular location and append our file path to it.(change your project physical path as required)
                            string directoryPath = @"E:\Project\wwwroot\" + filePath;
                            //check if directory exists on our defined directory path ?
                            //if not then create our directory path
                            if (!Directory.Exists(directoryPath))
                            {
                                //create directory at specified path
                                Directory.CreateDirectory(directoryPath);
                            }
                            //upload file to root directory of our project
                            string folderPath = Path.Combine("wwwroot/" +
                            filePath + "/", model_FileUpload.File.FileName);
                            using (FileStream fs = System.IO.File.Create(folderPath))
                            {
                                model_FileUpload.File.CopyTo(fs);
                            }
                            model_FileUpload.FilePath = "/" + filePath + "/" + model_FileUpload.File.FileName;
                        }
                        objCmd.CommandType = CommandType.StoredProcedure;
                        if (model_FileUpload.FileID == null || model_FileUpload.FileID == 0)
                        {
                            objCmd.CommandText = "PR_FileUpload_Insert";
                        }
                        else
                        {
                            objCmd.CommandText = "PR_FileUpload_Update";
                            objCmd.Parameters.Add("@FileUploadID", SqlDbType.Int).Value = model_FileUpload.FileID;
                        }
                        objCmd.Parameters.Add("@FilePath", SqlDbType.VarChar).Value = model_FileUpload.FilePath;

                        if (Convert.ToBoolean(objCmd.ExecuteNonQuery()))
                        {
                            if (model_FileUpload.FileID == null || model_FileUpload.FileID == 0)
                                TempData["Message"] = "Record Inserted Successfully";
                            else
                                return RedirectToAction("Index");
                        }
                    }
                    catch (Exception ex)
                    {
                        TempData["Message"] = ex.InnerException.Message.ToString();
                    }
                    finally
                    {
                        if (objConn.State == ConnectionState.Open)
                            objConn.Close();
                    }
                }
            }
            return RedirectToAction("Index");
        }
    ```
### Step 3: Prepare a view page with required inputs
Here we have created `Index.cshtml` page to get input and pass to controller

```csharp
<p>@TempData["Message"]</p>
<form role="form" method="post" enctype="multipart/form-data" asp-area="" aspcontroller="FileUpload" asp-action="Save">
    @Html.HiddenFor(x=>x.FileID)

    <div class="form-group col-md-4">
        <label for="exampleFormControlSelect1">Select File</label>
        <input type="file" name="File" class="form-control" />
    </div>
    
    @*Here we have taken file path just for understanding purpose*@
    @if (Model != null)
    {
        <div class="form-group col-md-4">
            <label>@Model.FilePath</label>
        </div>
    }

    <br />
    <button type="submit" class="btn btn-primary">Save</button>
</form>
```
