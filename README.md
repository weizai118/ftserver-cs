## Lightweight Full Text Search Server for CSharp

### Setup

```s
Download Project / git clone https://github.com/iboxdb/ftserver-cs.git
cd FTServer
dotnet run -c Release
```


![](https://github.com/iboxdb/ftserver/raw/master/FTServer/web/css/fts2.png)

### Dependencies
[iBoxDB](http://www.iboxdb.com/)

[AngleSharp](https://github.com/AngleSharp/AngleSharp)

[Semantic-UI](http://semantic-ui.com/)



### The Results Order
the results order based on the ID number in IndexTextNoTran(.. **long id**, ...),  descending order.

every page has two index-IDs, normal-id and rankup-id, the rankup-id is a big number and used to keep the important text on the **top**.  (the front results from SearchDistinct(IBox, String) )
````cs
Engine.IndexTextNoTran(..., p.Id, p.Content, ...);
Engine.IndexTextNoTran(..., p.RankUpId(), p.RankUpDescription(), ...);
````					

the RankUpId()
````cs
public long RankUpId()
{
    return id | (1L << 60);
}
````

if you have more more important text , you can add one more index-id
````cs
public long AdvertisingId()
{
    return id | (1L << 61);
}
````
````cs
public static long RankDownId(long id)
{
    return id & (~(1L << 60 | 1L << 61)) ;
}
public static bool IsAdvertisingId(long id)
{
    return id > (1L << 61) ;
}
````		


the Page.GetRandomContent() method is used to keep the Search-Page-Content always changing, doesn't affect the real page order.

if you have many pages(>100,000),  use the ID number to control the order instead of loading all pages to memory.


#### Search Format

[Word1 Word2 Word3] => text has **Word1** and **Word2** and **Word3**

["Word1 Word2 Word3"] => text has **"Word1 Word2 Word3"** as a whole


#### Search Method
searchDistinct (... String keywords, long **startId**, long **count**)

**startId** => which ID(the id when you called IndexText(,**id**,text)) to start, use (startId=Long.MaxValue) to read from the top, descending order

**count** => records to read,  **important parameter**, the search speed depends on this parameter, not how big the data is.

##### Next Page
set the startId as the last id from the results of searchDistinct() minus one

```cs
keywords = function(searchDistinct(box, "keywords", startId, count));
nextpage_startId = keywords[last].ID - 1 
...
//read next page
searchDistinct(box, "keywords", nextpage_startId, count)
```

mostly, the nextpage_startId is posted from client browser when user reached the end of webpage, and set the default nextpage_startId=Long.MaxValue, in javascript the big number have to write as String ("'" + nextpage_startId + "'")


#### The Page-Text and the Text-Index -Process flow

When Insert

1.insert page --> 2.insert index
````cs
DB.Insert ("Page", page);
Engine.IndexTextNoTran( IsRemove = false );
...IndexTextNoTran...
````


When Delete  

1.delete index --> 2.delete page
````cs
Engine.IndexTextNoTran( IsRemove = true );
...IndexTextNoTran...
DB.Delete("Page", page.Id);
````				

#### Memory
````cs
indexText(IBox, id, String, bool) // faster, more memories

indexTextNoTran(AutoBox, commitCount, id, String, bool) // less memory
````

#### Tools
Ubuntu 18.04 + Visual Studio Code + ASP.NET Core

#### More
[Transplant from Full Text Search Java JSP Version](https://github.com/iboxdb/ftserver)
