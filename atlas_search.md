# Atlas Search

## Pointers
- We are going to use **sample_mflix** Database
- **sample_mflix** database contains
  - Strings
  - Arrays
  - Integer
  -  Date
- We are going to **search across all the fields**
- **MongoDB uses B-Trees structure** to store index, but **Atlas search uses Inverted indexes** to store indexes.
-  An inverted index is like the **glossary** at the back of a **book**, where each term or **token** as it's called in search terminology **points to all the documents in which it appears**
-  And this is exactly what makes searches extremely fast and efficient.
-  Atlas search **can add new fields as the schema evolves**
-  To **run Atlas search** queries we need to **build aggregation pipelines**

**Atlas Search Examples:**

*Before running the below examples create a default Atlas Search Index.*

**Wildcard Search**
```javascript
db.movies.aggregate([{
    $search: {
        index: 'movies',
        text: {
            query: 'godfather',
            path: {
                'wildcard': '*'
            }
        }
    }
}])
```
**Search on a field i.e title**
```javascript
db.movies.aggregate([{
    $search: {
        index: 'movies',
        text: {
            query: 'godfather',
            path: "title"
        }
    }

}, {
    $project: {
        "title": 1,
        "imdb": 1,
        "_id": 0
    }
}])
```

**Fuzzy Matching**
```javascript
db.movies.aggregate([{
    $search: {
        index: 'movies',
        text: {
            query: 'naw yark',
            path: 'title',
            fuzzy: {
                maxEdits: 1,
                maxExpansions: 100
            }
        }
    }
}, {
    $project: {
        title: 1,
        _id: 0
    }
}])
```
**Auto Complete**

Before querying for auto Complete we need to create an index:
```
{
  "mappings": {
    "dynamic": false,
    "fields": {
      "title": [
        {
          "foldDiacritics": false,
          "maxGrams": 7,
          "minGrams": 3,
          "tokenization": "edgeGram",
          "type": "autocomplete"
        }
      ]
    }
  }
}
```

After creating the above Index we can run this query


```javascript
db.movies.aggregate([{
    $search: {
        "autocomplete": {
            "path": "title",
            "query": "off"
        }
    }
}, {
    $project: {
        title: 1,
        _id: 0
    }
}])
```
