URL start here
```
https://github.com/
```
Ends here

C# code starts here
```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Xml.Linq;


namespace ConsoleApplication1
{
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
}
```
Ends here
