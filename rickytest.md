The following is the app manifest for the continuing example of this article.


```json
{
  "extensions": [
        {
            "type": "FileHandler",
            "properties": {
                "extension": "drw",
                "fileIcon": "https://fabrikam.com/images/fileicon.png",
                "openUrl": "https://fabrikam.com/CADFileHandler/index",
                "previewUrl": "https://fabrikam.com/CADFileHandler/preview"
            }
        }
    ]
}
```



HTML

```html
<h3>Email/blog</h3>
	<ul>
		<li><a href="https://outlook.com/microsoft.com">Microsoft Outlook Web Access</a></li>
		<li><a href="https://login.microsoftonline.com/login.srf?wa=wsignin1%2E0&amp;rpsnv=2&amp;ct=1366076511&amp;rver=6%2E1%2E6206%2E0&amp;wp=MBI&amp;wreply=https%3A%2F%2Fmicrosoft%2Dmy%2Esharepoint%2Ecom%2F%5Fforms%2Fdefault%2Easpx&amp;lc=1033&amp;id=500046&amp;guests=1">Outlook.com</a></li>
		<li><a href="http://dashboard.bloglines.com/privatepage/1">Bloglines</a></li>
        <li><a href="https://mail.google.com/mail/#inbox">gmail</a></li>
	</ul>
	<h3>Banking</h3>
	<ul>
		<li><a href="https://online.firsttechfed.com/hb">1stTech HomeBanking</a> </li>
		<li><a href="http://www.ebay.com">ebay</a></li>
		<li><a href="http://www.mint.com">Mint</a></li>
		<li><a href="http://www.paypal.com">paypal</a></li>
	</ul>
	```
	
	C# goes here:



```cs

class Program
    {
        static void Main(string[] args)
        {
            //Load xml
            XDocument xdoc = XDocument.Load(@"c:\users\franla\documents\visual studio 2012\Projects\ConsoleApplication1\ConsoleApplication1\data.xml");

            //Run query
            var lv1s = from lv1 in xdoc.Descendants("level1")
                       select new
                       {
                           Header = lv1.Attribute("name").Value,
                           Children = lv1.Descendants("level2")
                       };

            //Loop through results
            StringBuilder result = new StringBuilder(); 
            foreach (var lv1 in lv1s)
            {
                result.AppendLine(lv1.Header);
                foreach (var lv2 in lv1.Children)
                    result.AppendLine("     " + lv2.Attribute("name").Value);
            }
            Console.Write(result);
        
        }
    }
    ```
More text

