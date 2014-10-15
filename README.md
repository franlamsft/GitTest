html start here
```html
	<h3>Other</h3>
	<ul>
		<li><a href="http://babel.altavista.com/tr">Babel Fish Translation Service</a></li>
		<li><a href="http://www.loc.gov/poetry/180/">Poetry 180 - Home Page</a>
		</li>
		<li>
		<a href="http://seattletimes.nwsource.com/flatpages/sports/sports/tvradiolistings.html?from=stnvs2">
		Seattle Times Sports TV-Radio</a></li>
		<li><a href="http://www.summitatsnoqualmie.com/">Snoqualmie Summit</a></li>
		<li><a href="http://www.totalimmersion.net/">Total Immersion Swimming</a>
		</li>
		<li><a href="http://www.uscollegehockey.com/">U.S. College Hockey Online</a>
		</li>
	</ul>
	<h3>Pictures</h3>
	<ul>
		<li><a href="http://www.flickr.com">Flickr</a></li>
		<li><a href="http://www.shutterfly.com">shutterfly</a></li>
		<li><a href="photos.htm">Photos</a></li>
	</ul>

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
