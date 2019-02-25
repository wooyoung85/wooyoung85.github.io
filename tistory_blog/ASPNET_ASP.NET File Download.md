## ASP.NET Web Form

다양한 방법이 있겠지만 **주로 `Handler(.ashx)`를 사용하여 구현함**

- Default.aspx
    ```csharp
    <asp:Button ID="btnDownload" runat="server" Text="파일 다운로드" 
                OnClick="btnDownload_Click"/>
    ```
- Default.aspx.cs
    ```csharp
    protected void btnDownload_Click(object sender, EventArgs e)
    {
        Response.Redirect("PathToHttpHandler/FileDownload.ashx");
    }
    ```

- FileDownload.ashx
    ```csharp
    public class FileDownload : IHttpHandler 
    {
        public void ProcessRequest(HttpContext context)
        {
            HttpResponse response = context.Response;

            string fileName = "텍스트 파일.txt";
            string filePath = Server.MapPath(Path.Combine("~/Files", "ansxkfl.txt"));

            response.ClearContent();
            response.Clear();
            response.ContentType = "text/plain";
            response.AddHeader("content-disposition", 
                            "attachment; filename=" + fileName + ";");
            response.TransmitFile(filePath);
            //response.WriteFile(filePath);

            response.Flush();    
            response.End();
        }

        public bool IsReusable
        {
            get
            {
                return false;
            }
        }
    }
    ```

## ASP.NET MVC
**`Controller`에 `Action`을 만들 때 return값을 FileContentResult 형태로 반환해야 함**

- Index.cshtml
    ```csharp
    @Html.ActionLink("파일 다운로드", "Download", "File")
    ```
- FileController.cs
    ```csharp
    public FileResult Download(){
        string fileName = "텍스트 파일.txt";
        string filePath = Server.MapPath(Path.Combine("~/Files", "ansxkfl.txt"));

        byte[] fileBytes = System.IO.File.ReadAllBytes(filePath);

        return File(fileBytes, System.Net.Mime.MediaTypeNames.Text.Plain, fileName);
    }
    ```
> ASP.NET MVC 에서 WebForm 방식과 동일하게 (HTTP Response 스트림에 파일을 쓰는 방식) 구현해도 파일 다운로드가 잘 되는 듯 했으나 운영 서버에서 문제가 발생하였다. ㅠ.ㅠ  
MVC 프로젝트에선 FileContentResult를 반환하는 방식으로 구현하는게 맞는 것 같다.