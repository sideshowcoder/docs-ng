<a id="couchbase-sampledata"></a>

# Sample buckets

Couchbase Server comes with sample buckets that contain both data and MapReduce
queries to demonstrate the power and capabilities.

This appendix provides information on the structure, format and contents of the
sample databases. The available sample buckets include:

 * [Game Simulation data](#couchbase-sampledata-gamesim)

 * [Beer and Brewery data](#couchbase-sampledata-beer)

<a id="couchbase-sampledata-gamesim"></a>

## Game Simulation sample bucket

The Game Simulation sample bucket is designed to showcase a typical gaming
application that combines records showing individual gamers, game objects and
how this information can be merged together and then reported on using views.

For example, a typical game player record looks like the one below:


```
{
    "experience": 14248,
    "hitpoints": 23832,
    "jsonType": "player",
    "level": 141,
    "loggedIn": true,
    "name": "Aaron1",
    "uuid": "78edf902-7dd2-49a4-99b4-1c94ee286a33"
}
```

A game object, in this case an Axe, is shown below:


```
{
   "jsonType" : "item",
   "name" : "Axe_14e3ad7b-8469-444e-8057-ac5aefcdf89e",
   "ownerId" : "Benjamin2",
   "uuid" : "14e3ad7b-8469-444e-8057-ac5aefcdf89e"
}
```

In this example, you can see how the game object has been connected to an
individual user through the `ownerId` field of the item JSON.

Monsters within the game are similarly defined through another JSON object type:


```
{
    "experienceWhenKilled": 91,
    "hitpoints": 3990,
    "itemProbability": 0.19239324085462631,
    "jsonType": "monster",
    "name": "Wild-man9",
    "uuid": "f72b98c2-e84b-4b17-9e2a-bcec52b0ce1c"
}
```

For each of the three records, the `jsonType` field is used to define the type
of the object being stored.

<a id="couchbase-sampledata-gamesim-views-leaderboard"></a>

### leaderboard view

The `leaderboard` view is designed to generate a list of the players and their
current score:


```
function (doc) {
  if (doc.jsonType == "player") {
  emit(doc.experience, null);
  }
}
```

The view looks for records with a `jsonType` of "player", and then outputs the
`experience` field of each player record. Because the output from views is
naturally sorted by the key value, the output of the view will be a sorted list
of the players by their score. For example:


```
{
   "total_rows" : 81,
   "rows" : [
      {
         "value" : null,
         "id" : "Bob0",
         "key" : 1
      },
      {
         "value" : null,
         "id" : "Dustin2",
         "key" : 1
      },
…
      {
         "value" : null,
         "id" : "Frank0",
         "key" : 26
      }
   ]
}
```

To get the top 10 highest scores (and ergo players), you can send a request that
reverses the sort order (by using `descending=true`, for example:


```
http://127.0.0.1:8092/gamesim-sample/_design/dev_players/_view/leaderboard?descending=true&connection_timeout=60000&limit=10&skip=0
```

Which generates the following:


```
{
   "total_rows" : 81,
   "rows" : [
      {
         "value" : null,
         "id" : "Tony0",
         "key" : 23308
      },
      {
         "value" : null,
         "id" : "Sharon0",
         "key" : 20241
      },
      {
         "value" : null,
         "id" : "Damien0",
         "key" : 20190
      },
…
      {
         "value" : null,
         "id" : "Srini0",
         "key" :9
      },
      {
         "value" : null,
         "id" : "Aliaksey1",
         "key" : 17263
      }
   ]
}
```

<a id="couchbase-sampledata-gamesim-views-playerlist"></a>

### playerlist view

The `playerlist` view creates a list of all the players by using a map function
that looks for "player" records.


```
function (doc, meta) {
  if (doc.jsonType == "player") {
    emit(meta.id, null);
  }
}
```

This outputs a list of players in the format:


```
{
   "total_rows" : 81,
   "rows" : [
      {
         "value" : null,
         "id" : "Aaron0",
         "key" : "Aaron0"
      },
      {
         "value" : null,
         "id" : "Aaron1",
         "key" : "Aaron1"
      },
      {
         "value" : null,
         "id" : "Aaron2",
         "key" : "Aaron2"
      },
      {
         "value" : null,
         "id" : "Aliaksey0",
         "key" : "Aliaksey0"
      },
      {
         "value" : null,
         "id" : "Aliaksey1",
         "key" : "Aliaksey1"
      }
   ]
}
```

<a id="couchbase-sampledata-beer"></a>

## Beer sample bucket

The beer sample data demonstrates a combination of the document structure used
to describe different items, including references between objects, and also
includes a number of sample views that show the view structure and layout.

The primary document type is the 'beer' document:


```
{
   "name": "Piranha Pale Ale",
   "abv": 5.7,
   "ibu": 0,
   "srm": 0,
   "upc": 0,
   "type": "beer",
   "brewery_id": "110f04166d",
   "updated": "2010-07-22 20:00:20",
   "description": "",
   "style": "American-Style Pale Ale",
   "category": "North American Ale"
}
```

Beer documents contain core information about different beers, including the
name, alcohol by volume ( `abv` ) and categorization data.

Individual beer documents are related to brewery documents using the
`brewery_id` field, which holds the information about a specific brewery for the
beer:


```
{
   "name": "Commonwealth Brewing #1",
   "city": "Boston",
   "state": "Massachusetts",
   "code": "",
   "country": "United States",
   "phone": "",
   "website": "",
   "type": "brewery",
   "updated": "2010-07-22 20:00:20",
   "description": "",
   "address": [
   ],
   "geo": {
       "accuracy": "APPROXIMATE",
       "lat": 42.3584,
       "lng": -71.0598
   }
}
```

The brewery record includes basic contact and address information for the
brewery, and contains a spatial record consisting of the latitude and longitude
of the brewery location.

To demonstrate the view functionality in Couchbase Server, three views are
defined.

<a id="couchbase-sampledata-beer-views-brewerybeers"></a>

### brewery_beers view

The `brewery_beers` view outputs a composite list of breweries and beers they
brew by using the view output format to create a 'fake' join, as detailed in
[Solutions for Simulating Joins](#couchbase-views-sample-patterns-joins). This
outputs the brewery ID for brewery document types, and the brewery ID and beer
ID for beer document types:


```
function(doc, meta) {
  switch(doc.type) {
  case "brewery":
    emit([meta.id]);
    break;
  case "beer":
    if (doc.brewery_id) {
      emit([doc.brewery_id, meta.id]);
    }
    break;
  }
}
```

The raw JSON output from the view:


```
{
   "total_rows" : 7315,
   "rows" : [
      {
         "value" : null,
         "id" : "110f0013c9",
         "key" : [
            "110f0013c9"
         ]
      },
      {
         "value" : null,
         "id" : "110fdd305e",
         "key" : [
            "110f0013c9",
            "110fdd305e"
         ]
      },
      {
         "value" : null,
         "id" : "110fdd3d0b",
         "key" : [
            "110f0013c9",
            "110fdd3d0b"
         ]
      },
…
      {
         "value" : null,
         "id" : "110fdd56ff",
         "key" : [
            "110f0013c9",
            "110fdd56ff"
         ]
      },
      {
         "value" : null,
         "id" : "110fe0aaa7",
         "key" : [
            "110f0013c9",
            "110fe0aaa7"
         ]
      },
      {
         "value" : null,
         "id" : "110f001bbe",
         "key" : [
            "110f001bbe"
         ]
      }
   ]
}
```

The output could be combined with the corresponding brewery and beer data to
provide a list of the beers at each brewery.

<a id="couchbase-sampledata-beer-views-by_location"></a>

### by_location view

Outputs the brewery location, accounting for missing fields in the source data.
The output creates information either by country, by country and state, or by
country, state and city.


```
function (doc, meta) {
  if (doc.country, doc.state, doc.city) {
    emit([doc.country, doc.state, doc.city], 1);
  } else if (doc.country, doc.state) {
    emit([doc.country, doc.state], 1);
  } else if (doc.country) {
    emit([doc.country], 1);
  }
}
```

The view also includes the built-in `_count` function for the reduce portion of
the view. Without using the reduce, the information outputs the raw location
information:


```
{
   "total_rows" : 1413,
   "rows" : [
      {
         "value" : 1,
         "id" : "110f0b267e",
         "key" : [
            "Argentina",
            "",
            "Mendoza"
         ]
      },
      {
         "value" : 1,
         "id" : "110f035200",
         "key" : [
            "Argentina",
            "Buenos Aires",
            "San Martin"
         ]
      },
…
      {
         "value" : 1,
         "id" : "110f2701b3",
         "key" : [
            "Australia",
            "New South Wales",
            "Sydney"
         ]
      },
      {
         "value" : 1,
         "id" : "110f21eea3",
         "key" : [
            "Australia",
            "NSW",
            "Picton"
         ]
      },
      {
         "value" : 1,
         "id" : "110f117f97",
         "key" : [
            "Australia",
            "Queensland",
            "Sanctuary Cove"
         ]
      }
   ]
}
```

With the `reduce()` enabled, grouping can be used to report the number of
breweries by the country, state, or city. For example, using a grouping level of
two, the information outputs the country and state counts:


```
{"rows":[
{"key":["Argentina",""],"value":1},
{"key":["Argentina","Buenos Aires"],"value":1},
{"key":["Aruba"],"value":1},
{"key":["Australia"],"value":1},
{"key":["Australia","New South Wales"],"value":4},
{"key":["Australia","NSW"],"value":1},
{"key":["Australia","Queensland"],"value":1},
{"key":["Australia","South Australia"],"value":2},
{"key":["Australia","Victoria"],"value":2},
{"key":["Australia","WA"],"value":1}
]
}
```

<a id="couchbase-views-troubleshooting"></a>
