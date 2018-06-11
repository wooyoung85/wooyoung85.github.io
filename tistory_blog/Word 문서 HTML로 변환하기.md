## Word 문서 HTML로 변환하기

### 1. OpenXml 활용
- **`.docx` 파일만 변환 가능**
  + .docx 파일은 Open Office XML(OOXML) 구조를 사용
  + 좀 더 자세하게 설명하자면 내부적으로 xml 파일들로 구성되어 있고 이름 ZIP형식으로 압축하여 저장함
  + .docx 파일의 내부 구조가 궁금하다면 확장자를 .zip으로 변경 후 압축 해제하면 볼 수 있다.

- OpenXml 관련 라이브러리 추가
  + Document.Format.OpenXml
  + OpenXmlPowerTools --> 이 라이브러리를 추가하면 Document.Format.OpenXml도 같이 추가됨
  
- 이미지가 포함되어 있지 않은 Word문서 변환 Code
```csharp
byte[] byteArray = File.ReadAllBytes("Test.docx");
using (MemoryStream memoryStream = new MemoryStream())
{
    memoryStream.Write(byteArray, 0, byteArray.Length);
    using (WordprocessingDocument doc =
        WordprocessingDocument.Open(memoryStream, true))
    {
        HtmlConverterSettings settings = new HtmlConverterSettings()
        {
            PageTitle = "My Page Title"
        };
        XElement html = HtmlConverter.ConvertToHtml(doc, settings);        

        File.WriteAllText("Test.html", html.ToStringNewLineOnAttributes());
    }
}
```

- 이미지가 포함된 Word문서 변환 Code
```csharp
string sourceDocumentFileName = "Test.docx";
FileInfo fileInfo = new FileInfo(sourceDocumentFileName);
string imageDirectoryName = fileInfo.Name.Substring(0, fileInfo.Name.Length - fileInfo.Extension.Length) + "_files";
DirectoryInfo dirInfo = new DirectoryInfo(imageDirectoryName);

if (dirInfo.Exists)
{
    // Delete the directory and files.
    foreach (var f in dirInfo.GetFiles())
        f.Delete();
    dirInfo.Delete();
}

int imageCounter = 0;
byte[] byteArray = File.ReadAllBytes(sourceDocumentFileName);

using (MemoryStream memoryStream = new MemoryStream())
{
    memoryStream.Write(byteArray, 0, byteArray.Length);
    using (WordprocessingDocument doc =
        WordprocessingDocument.Open(memoryStream, true))
    {
        HtmlConverterSettings settings = new HtmlConverterSettings()
        {
            PageTitle = "Test Title",
            ConvertFormatting = false,
        };
        XElement html = HtmlConverter.ConvertToHtml(doc, settings,
            imageInfo =>
            {
                DirectoryInfo localDirInfo = new DirectoryInfo(imageDirectoryName);
                if (!localDirInfo.Exists)
                    localDirInfo.Create();
                ++imageCounter;
                string extension = imageInfo.ContentType.Split('/')[1].ToLower();
                ImageFormat imageFormat = null;
                if (extension == "png")
                {
                    // Convert the .png file to a .jpeg file.
                    extension = "jpeg";
                    imageFormat = ImageFormat.Jpeg;
                }
                else if (extension == "bmp")
                    imageFormat = ImageFormat.Bmp;
                else if (extension == "jpeg")
                    imageFormat = ImageFormat.Jpeg;
                else if (extension == "tiff")
                    imageFormat = ImageFormat.Tiff;

                if (imageFormat == null)
                    return null;

                string imageFileName = imageDirectoryName + "/image" +
                    imageCounter.ToString() + "." + extension;
                try
                {
                    imageInfo.Bitmap.Save(imageFileName, imageFormat);
                }
                catch (System.Runtime.InteropServices.ExternalException)
                {
                    return null;
                }
                XElement img = new XElement(Xhtml.img,
                    new XAttribute(NoNamespace.src, imageFileName),
                    imageInfo.ImgStyleAttribute,
                    imageInfo.AltText != null ?
                        new XAttribute(NoNamespace.alt, imageInfo.AltText) : null);
                return img;
            });            

        File.WriteAllText(fileInfo.Directory.FullName + "/" + fileInfo.Name.Substring(0,
            fileInfo.Name.Length - fileInfo.Extension.Length) + ".html",
            html.ToStringNewLineOnAttributes());
    }
}
```
- 변환 시 CSS를 제공하여 단락(Paragraph) 수준에서 일부 서식을 구성할 수 있는 기능도 제공 </br>[Supplying a CSS to the Transformation](https://msdn.microsoft.com/en-us/library/office/ff628051(v=office.14).aspx#Anchor_4)

- 한계점
  - `.doc` 파일은 변환 불가 (하지만 .docx로 저장 후 변환하면 됨)
  - **TextBox나 도형, SmartArt 같은 Drawing 영역은 변환하지 못함**


### 2. Microsoft.Office.Interop.Word 활용
- C#에서 Microsoft Office와의 호환성을 위해 제공하는 어셈블리
- 변환작업 진행 시 Microsoft Word 프로그램이 실제로 실행되기 때문에 **Microsoft Word 프로그램이 설치된 환경에서만 가능**
- 라이브러리 추가
  + Microsoft.Office.Interop.Word
- Word문서 Html로 변환하는 Code
```csharp
using Word = Microsoft.Office.Interop.Word;
using System;

public static void ConvertDocToHtml(object Sourcepath, object TargetPath)
{    
    Word._Application wordApp = new Word.Application();
    Word.Documents doc = wordApp.Documents;
    object Unknown = Type.Missing;
    object format = Word.WdSaveFormat.wdFormatHTML;

    Word.Document od = doc.Open(ref Sourcepath, ref Unknown,
                                ref Unknown, ref Unknown, ref Unknown,
                                ref Unknown, ref Unknown, ref Unknown,
                                ref Unknown, ref Unknown, ref Unknown,
                                ref Unknown, ref Unknown, ref Unknown, ref Unknown);

    wordApp.ActiveDocument.SaveAs(ref TargetPath, ref format,
                ref Unknown, ref Unknown, ref Unknown,
                ref Unknown, ref Unknown, ref Unknown,
                ref Unknown, ref Unknown, ref Unknown,
                ref Unknown, ref Unknown, ref Unknown,
                ref Unknown, ref Unknown);

    wordApp.Documents.Close(Word.WdSaveOptions.wdDoNotSaveChanges);
    wordApp.Quit(Word.WdSaveOptions.wdDoNotSaveChanges);
}
```
- 장점 및 단점
  + OpenXml에서 변환하기 힘들었던 TextBox나 도형, SmartArt 같은 Drawing 영역을 변환해 줌
  + Microsoft Word 프로그램이 실행되어만 하기 때문에 환경적인 제약이 크다.


## 참고사이트
[Transforming Open XML WordprocessingML to XHTML Using the Open XML SDK 2.0](https://msdn.microsoft.com/en-us/library/office/ff628051(v=office.14).aspx)

[Microsoft Word 2007 (.docx)에서 이미지 파일 변환 형태](http://forensic-proof.com/archives/325)

[Open-Xml-PowerTools Github Repository](https://github.com/OfficeDev/Open-Xml-PowerTools)

[Convert from Word document to HTML](https://stackoverflow.com/questions/2266097/convert-from-word-document-to-html)