Content API Scala Client
========================

A Scala client for the Guardian's [Content API] (http://explorer.content.guardianapis.com/).


## Setup

Add the following line to your SBT build definition, and set the version number to be the latest from the [releases page] (https://github.com/guardian/content-api-scala-client/releases):

```scala
libraryDependencies += "com.gu" %% "content-api-client" % "x.y"
```

If you don't have an API key, go to [guardian.mashery.com] (http://guardian.mashery.com/) to get one. You will then need to create a new instance of the client and set the key:

```scala
val client = new GuardianContentClient("your-api-key")
```

## Usage

There are then four different types of query that can be performed: for a single item, or to filter through content, tags, or sections. You make a request of the Content API by creating a query and then using the client to get a response, which will come back in a `Future`.

Use these imports for the following code samples (substituting your own execution context for real code):

```scala
import com.gu.contentapi.client.GuardianContentClient
import com.gu.contentapi.client.model._
import org.joda.time.DateTime
import scala.concurrent.ExecutionContext.Implicits.global
```

### Single item

Every item on http://www.theguardian.com/ can be retrieved on the same path at http://content.guardianapis.com/. They can be either content items, tags, or sections. For example:

```scala
// query for a single content item and print its web title
val itemQuery = ItemQuery("commentisfree/2013/jan/16/vegans-stomach-unpalatable-truth-quinoa")
client.getResponse(itemQuery).foreach { itemResponse =>
  println(itemResponse.content.get.webTitle)
}

// print web title for a tag
val tagQuery = ItemQuery("music/metal")
client.getResponse(tagQuery).foreach { tagResponse =>
  println(tagResponse.content.get.webTitle)
}

// print web title for a section
val sectionQuery = ItemQuery("environment")
client.getResponse(sectionQuery).foreach { sectionResponse =>
  println(sectionResponse.content.get.webTitle)
}
```

Individual content items contain information not available from the `/search` endpoint described below. For example:

```scala
// print the body of a given content item
val itemBodyQuery = ItemQuery("politics/2014/sep/15/putin-bad-as-stalin-former-defence-secretary")
  .showFields("body")
client.getResponse(itemBodyQuery) map { response =>
  for (fields <- response.content.get.fields) println(fields("body"))
}

// print the web title of every tag a content item has
val itemWebTitleQuery = ItemQuery("environment/2014/sep/14/invest-in-monitoring-and-tagging-sharks-to-prevent-attacks")
  .showTags("all")
client.getResponse(itemWebTitleQuery) map { response =>
  for (tag <- response.content.get.tags) println(tag.webTitle)
}

// print the web title of each content item in the editor's picks for the film tag
val editorsFilmsQuery = ItemQuery("film/film").showEditorsPicks()
client.getResponse(editorsFilmsQuery) map { response =>
  for (result <- response.editorsPicks) println(result.webTitle)
}

// print the web title of the most viewed content items from the world section
val mostViewedTitleQuery = ItemQuery("world").showMostViewed()
client.getResponse(mostViewedTitleQuery) map { response =>
  for (result <- response.mostViewed) println(result.webTitle)
}
```

### Content

Filtering or searching for multiple content items happens at http://content.guardianapis.com/search. For example:

```scala
// print the total number of content items
val allContentSearch = SearchQuery()
client.getResponse(allContentSearch) map { response =>
  println(response.total)
}

// print the web titles of the 15 most recent content items
val lastTenSearch = SearchQuery().pageSize(15)
client.getResponse(lastTenSearch) map { response =>
  for (result <- response.results) println(result.webTitle)
}

// print the web titles of the 10 most recent content items matching a search term
val toastSearch = SearchQuery().q("cheese on toast")
client.getResponse(toastSearch) map { response =>
  for (result <- response.results) println(result.webTitle)
}

// print the web titles of the 10 (default page size) most recent content items with certain tags
val tagSearch = SearchQuery().tag("lifeandstyle/cheese,type/gallery")
client.getResponse(tagSearch) map { response =>
  for (result <- response.results) println(result.webTitle)
}

// print the web titles of the 10 most recent content items in the world section
val sectionSearch = SearchQuery().section("world")
client.getResponse(sectionSearch) map { response =>
  for (result <- response.results) println(result.webTitle)
}

// print the web titles of the last 10 content items published a week ago
val timeSearch = SearchQuery().toDate(new DateTime().minusDays(7))
client.getResponse(timeSearch) map { response =>
  for (result <- response.results) println(result.webTitle)
}
```

### Tags

Filtering or searching for multiple tags happens at http://content.guardianapis.com/tags. For example:

```scala
// print the total number of tags
val allTagsQuery = TagsQuery()
client.getResponse(allTagsQuery) map { response =>
  println(response.total)
}

// print the web titles of the first 50 tags
val fiftyTagsQuery = TagsQuery().pageSize(50)
client.getResponse(fiftyTagsQuery) map { response =>
  for (result <- response.results) println(result.webTitle)
}

// print the web titles and bios of the first 10 contributor tags which have them
val contributorTagsQuery = TagsQuery().tagType("contributor")
client.getResponse(contributorTagsQuery) map { response =>
  for (result <- response.results.filter(_.bio.isDefined)) {
    println(result.webTitle + "\n" + result.bio.get + "\n")
  }
}

// print the web titles and numbers of the first 10 books tags with ISBNs
val isbnTagsSearch = TagsQuery()
  .section("books")
  .referenceType("isbn")
  .showReferences("isbn")
client.getResponse(isbnTagsSearch) map { response =>
  for (result <- response.results) {
    println(result.webTitle + " -- " + result.references.head.id)
  }
}
```

### Sections

Filtering or searching for multiple sections happens at http://content.guardianapis.com/sections. For example:

```scala
// print the web title of each section
val allSectionsQuery = SectionsQuery()
client.getResponse(allSectionsQuery) map { response =>
  for (result <- response.results) println(result.webTitle)
}

// print the web title of each section with 'network' in the title
val networkSectionsQuery = SectionsQuery().q("network")
client.getResponse(networkSectionsQuery) map { response =>
  for (result <- response.results) println(result.webTitle)
}
```

### Editions

Filtering or searching for multiple Editions happens at http://content.guardianapis.com/editions. For example:

```scala
// print the apiUrl of each edition
val allEditionsQuery = EditionsQuery()
client.getResponse(allEditionsQuery) map { response =>
  for (result <- response.results) println(result.apiUrl)
}

// print the webUrl of the edition with 'US' in edition field.
val usEditionsQuery = EditionsQuery().q("US")
client.getResponse(usEditionsQuery) map { response =>
  for (result <- response.results) println(result.webUrl)
}
```

## Troubleshooting

If you have any problems you can speak to other developers at the [Guardian API talk group] (http://groups.google.com/group/guardian-api-talk).
