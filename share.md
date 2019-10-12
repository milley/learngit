```python
#!/usr/bin/python3

from xml.dom.minidom import parse
import xml.dom.minidom
from lxml import etree as ET

# 使用minidom解析器打开 XML 文档
DOMTree = xml.dom.minidom.parse("./movies.xml")
collection = DOMTree.documentElement
if collection.hasAttribute("shelf"):
   print ("Root element : %s" % collection.getAttribute("shelf"))

# 在集合中获取所有电影
movies = collection.getElementsByTagName("movie")

# 打印每部电影的详细信息
for movie in movies:
    print ("*****Movie*****")
    if movie.hasAttribute("title"):
        print ("Title: %s" % movie.getAttribute("title"))

    type = movie.getElementsByTagName('type')[0]
    print ("Type: %s" % type.childNodes[0].data)
    format = movie.getElementsByTagName('format')[0]
    print ("Format: %s" % format.childNodes[0].data)
    rating = movie.getElementsByTagName('rating')[0]
    print ("Rating: %s" % rating.childNodes[0].data)
    description = movie.getElementsByTagName('description')[0]
    print ("Description: %s" % description.childNodes[0].data)
   

try:
    with open('movies-res.xml', 'w', encoding='utf-8') as fr:
        DOMTree.writexml(fr, encoding='utf-8')
        print('Write TranResult.xml OK!')
except IOError as e:
    print(e)
finally:
    fr.close()

tree = ET.parse('movies.xml')

collection = tree.getroot()
print(collection.tag)

for movie in collection.iter("movie"):
    print(movie.tag, ":", movie.get("title"))

tree.write('movies-res-unordered.xml')
```

```xml
<collection shelf="New Arrivals">
<movie title="Enemy Behind" name="Enemy Behind" type="1" Action="a">
   <type>War, Thriller</type>
   <format>DVD</format>
   <year>2003</year>
   <rating>PG</rating>
   <stars>10</stars>
   <description>Talk about a US-Japan war</description>
</movie>
<movie title="Transformers" name="Transformers" type="2" Action="b">
   <type>Anime, Science Fiction</type>
   <format>DVD</format>
   <year>1989</year>
   <rating>R</rating>
   <stars>8</stars>
   <description>A schientific fiction</description>
</movie>
<movie title="Trigun" name="Trigun" type="3" Action="c">
   <type>Anime, Action</type>
   <format>DVD</format>
   <episodes>4</episodes>
   <rating>PG</rating>
   <stars>10</stars>
   <description>Vash the Stampede!</description>
</movie>
<movie title="Ishtar" name="Ishtar" type="4" Action="d">
   <type>Comedy</type>
   <format>VHS</format>
   <rating>PG</rating>
   <stars>2</stars>
   <description>Viewable boredom</description>
</movie>
</collection>
```
