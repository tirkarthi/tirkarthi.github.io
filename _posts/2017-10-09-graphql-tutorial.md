---
layout: post
title:  A GraphQL tutorial
date:   2017-10-09 07:00:29 +0530
categories: clojure
---

*Thanks to Walmart Labs for their Lacinia library and the helpful members at the Slack channel*

### Early days of REST

Most of us use REST everyday for building our web applications and I think it's a great step forward from the early days of writing HTML where every endpoint was pretty tightly coupled to the resource you are trying to access like `get-books`, `update-book`, `delete-book`, etc. Apart from the naming every project used to follow different conventions which was a hurdle as we move from one project to another and REST fixed a lot of the problems. But we hit some limitations of REST in our daily programming as the teams were split into backend and frontend.

### Limitations of REST 

* Frontend needs keys that are different from what the backend supplies to them. Most of the time backend team serializes the record using the database column names for the objects. But the frontend team needs the value with different key and hence it leads to back and forth updates to the backend code and docs. Eg. Frontend might require the column `last_updated_at` as `last_modified_at`.
* Frontend team was consuming data they don't need at all for some of the pages thus increasing the bandwidth for those pages. Eg. A summary list with name and subject through GET request was returning a lot of data they don't need and REST doesn't provide good ways to get only the required keys.
* Backend needs to maintain separate endpoints for different resources and hence when the frontend team needs to present as a cohesive page with data from different identities then they need to make varying amount of REST calls to get the data and then manipulate the data in the browser forming a cohesive state. Eg. A page with information about the book and the author along with all the other books written by the author would perform a GET request for the book information and an another one for the author followed by another for the list of books by the author with each call having the same problem returning a lot of unused data.

### Evaluating GraphQL

So I was evaluating the usage of GraphQL to solve some of the problems and to identify the ecosystem around GraphQL to see if we can utilise the current tooling to solve our use case. The following is a sample application that I have implemented using Lacinia, an open source GraphQL implementation for Clojure for Walmart Labs explaining my experience with it and the limitations.

### Common myths surrounding GraphQL

__GraphQL and security__ : Since GraphQL involves querying from the browser it often scares people around performing queries over database from the browser which could lead to data breach and so forth. We will get to know that it's much more sandboxed in the amount of access we give to our users along with some of the limitations surrounding permissions.

__GraphQL and RDBMS__ : It's a common misconception due to the name that GraphQL requires some sort of graph database or NoSQL database. But GraphQL is only a spec and the implementation of the spec doesn't really limit it to graph database. GraphQL can be used with RDBMS, NoSQL and even in-memory stores. This tutorial will use SQLite.

__GraphQL and writes__ : Another misconception surrounding the usage of GraphQL is that very less people show cool demos with the read aspects of the database making it feel like writes are very hard in GraphQL. But writes use the same underlying concept as reads do. But it's often recommended to be more specific about writes unlike REST where we can update almost all the fields using the same endpoint.

### A music collection app

The tutorial will involve building a simple music collection app with two tables. The artist table contains id and name of the artist. The track table contains the track ID, name along with a foreign key reference to the artist ID for each track.

### GraphQL schema

GraphQL wants us to define a schema file that essentially allows us to define the fields enabled for access along with other meta data and how we will be retrieving the given data. Thus we can define here to make sure only specified fields are exposed. The schema file we will be using uses EDN but you can use JSON or any other format necessary. The schema file for the project will be as below : 

```clojure
{:objects
 {:track
  {:fields {:trackid {:type Int}
            :trackname {:type String}
            :trackartist {:type (list :artist)
                          :resolve :get-artist}}}
  :artist
  {:fields {:artistid {:type Int}
            :artistname {:type String}
            :tracks {:type (list :track)
                     :resolve :get-tracks}}}}
 :mutations
 {:createArtist
  {:resolve :create-artist
   :type (list :artist)
   :args {:artistname {:type String}}}}
 :queries
 {:artist
  {:resolve :get-artist
   :type (list :artist)
   :args {:id {:type Int}
          :name {:type String}}}
  :track
  {:resolve :get-tracks
   :type (list :track)
   :args {:trackid {:type Int}
          :first {:type Int}
          :trackname {:type String}}}}}
```

__Objects__ : An object is an entity containing the given fields. Here track object has the fields trackid, trackname and trackartist with types. We could see that trackartist is by itself an object unlike the other fields of primitive types. We also specify that it can be resolved using get-artist function. Similarly we define the fields for an artist and it also contains tracks object which is a list of tracks and it's resolved by get-tracks function. We will be writing the resolvers soon thus confirming the fact that graph databases are not necessary for using GraphQL. Only the specified fields will be accessible from the client thus we don't need to worry about data breach unless we specify secret fields here.

__Queries__ : Queries determine the queries we will be performing using GraphQL. They don't need to have the same name as the objects. We specify that the query can be resolved using get-artist function and it returns a list of artists. We  can also specify the fields through args that we will be accepting from the client for querying. So there are a set of fixed queries and thus we will not be exposing all the tables to the client.

__Mutations__ : Mutations are GraphQL's way of specifying the writes and as mentioned above it's recommended to use specific mutations unlike REST where we have single endpoint for multiple updates. I have `createArtist` mutation that enables me to accept an artist name to create a record. It also specifies that I will be returning a list of artist and the write logic is handled in create-artist function.

### GraphQL resolvers

GraphQL resolvers are a way to return the appropriate results for the queries along with handling our writes too. Thus a simple resolver to for an artist can be return as below : 

```clojure
(defn get-artist [context arguments value]
  (let [{:keys [id name]} arguments
        {id :trackartist, :or {id id}} value]
   '({:artistid 1 :artistname "Kiara"}))
```

The `get-artist` takes three parameters context, argument and value. Context will be nil here for the most part. `arguments` contains the arguments passed as part of query and `value` is something we will get to soon. Since we specified in the schema file that id, limit and name are the fields through which we can filter our records we use them. 

### GraphQL queries

```
query {
  artist(id: 1) {
    artistname
  }
}
```

In GraphQL there is a single endpoint where we post the query and get the result instead of using a separate endpoints for each resource. We can post the query as payload and get the results. In our case all the queries are via in query params to `localhost:8888` to return the result using a rich IDE. This is not our production use case where sending data through query params will result in data breaches. The `query` attribute defines that this will be a read-query and we are querying for an artist with ID as 1 and we need the artistname. Since we didn't specify artistid which is also a part of the object it's not returned. Thus the above query will return the below result : 

```
{
  "data": {
    "artist": [
      {
        "artistname": "Kiara"
      }
    ]
  }
}
```

### Let us go deeper

Since we have the artistid and artistname returned directly as part of the result from the table how do we retrieve the related tracks for the artist which is also a part of the object. We can see from the schema file we have attached the function `get-tracks` with it. Hence to retrieve the tracks information for the given artist GraphQL passes each artist entry from the returned list to the get track function as the value argument. Imagine the `get-tracks` function as below : 

```clojure
(defn get-tracks [context arguments value]
  (let [{:keys [artistid trackname trackid]} value
        {:keys [trackname trackid first], :or {trackname trackname trackid trackid}} arguments]
    '({:trackid 1 :trackname "Heavy"})))
```

This gets the artistid in the value map and thus we can return all the tracks for the artist given the artistid. We simply return a fixed set of results but we can use the information to query from the database or in-memory store and return the results. Thus the query to get an artist information along with all the tracknames by the artist will be as below : 

__Query__ :

```
query {
  artist(id: 1) {
    artistname
    tracks {
      trackname
    }
  }
}
```

__Result__ : 

```
{
  "data": {
    "artist": [
      {
        "artistname": "Kiara",
        "tracks": [
          {
            "trackname": "Heavy"
          }
        ]
      }
    ]
  }
}
```

And the magic part is that we can one more level deeper to get the artistname for the track again since we have declared an artist field for the track. Thus the track row with track ID, name and artist ID will be passed to the `get-artist` function again. Since we have the artist ID we can get the artistname again one level deeper. We can keep doing this but we have to remember the fact that it this callback is done for each row. So if an artist has 10 tracks and we do the same thing it will result in 10 function calls to get the result. Hence as we go deeper and deeper the amount of calls increase a lot. Instead of returning fixed set of results we can use our data retrieval logics there. The below function is the actual `get-artist` function as part of the repo that uses HoneySQL to construct the query and returns the results.

```clojure
(defn get-artist [context arguments value]
  (let [{:keys [id name]} arguments
        {id :trackartist, :or {id id}} value
        query (-> (select :*)
                  (merge-where (if (some? id) [:= :artistid id]))
                  (merge-where (if (some? name) [:= :artistname name]))
                  (from :artist)
                  sql/format)
        result (jdbc/query db query)]
    result))
```

### GraphQL mutations

So we are done with the read part but what about the write part which is a significant other half in the application. GraphQL uses the concept of mutations
to handle writes. The mutations part is similar to query except that we make insert instead of using select queries. The below function is the `create-artist` resolver that gets the artist's name and creates an artist entry in the table. Here we get the artist name and insert it into the table. The last inserted row is returned. Since we have specified the return type to return artist object we can filter the output fields as in the queries. The following mutation creates an artist named Amy and returns the ID and name.

```clojure
(defn create-artist [context arguments value]
  (let [{:keys [artistname]} arguments
        insert-stmt (-> (insert-into :artist)
                        (columns :artistname)
                        (values [[artistname]])
                        (sql/format))
        result (jdbc/execute! db insert-stmt)
        res-query (-> (select :*)
                      (from :artist)
                      (order-by [:artistid :desc])
                      (limit 1)
                      sql/format)
        artist (jdbc/query db res-query)]
    artist))
```
__Mutation__ :

```
mutation {
  createArtist(artistname: "Amy") {
    artistname
    artistid
  }
}
```

__Result__ :

```
{
  "data": {
    "createArtist": [
      {
        "artistname": "Amy",
        "artistid": 17
      }
    ]
  }
}
```

### Renaming keys

In addition to the query part GraphQL allows the frontend team to rename the keys in the result as they need. This is done by specifying the key name before the field name. The below query returns the result with artistname key as simply name

```
query {
  artist(id: 1) {
    name: artistname
  }
}
```

```
{
  "data": {
    "artist": [
      {
        "name": "Kiara"
      }
    ]
  }
}
```

### Declarative queries

Since the queries are declarative in nature we can easily maintain them and also change them over time. Since it's far easy to resolve the relations we can also write less and to fill a lot of use cases. In a music app when the user loves a song the artist and genre details can be retrieved easily. Similarly we can list all the other songs by the artist to the user thus building cohesive profiles using data from lot of endpoints in REST will be a single query in GraphQL.

### IGraphQL

IGraphQL is sort of an IDE that makes the development a lot more interactive. It also generates docs from the schema file thus we can make explore on the fields with more ease and interactivity compared to something like a Swagger setup.

### Sample use cases

* In case of source code we can get the commits and get the author details along with other repositories by the other using the same language as the source code.
* In case of a shopping cart app we can display an item and retrieve all the other different items to show to the user with retrieving only the necessary fields.
* In case of comparing two items we can let the user determine the fields over which they need to compare and get only the relevant fields for the resource saving a lot of bandwidth in mobile apps.
* It enables developers to build more out of the platform instead of using GraphQL than glueing up REST calls.

### Testing

Testing of queries is lot more easier with executing the query and then comparing the result. This makes writing more extensive tests to suit the complexity in highly related queries and also ensures that if we upgrade from one schema to another the test suite is more robust indicating the breakage involved.

### Disadvantages

* GraphQL is just a spec and hence a lot of custom code is required for supporting pagination, rate-limiting, permissions, etc. depending upon the language. Some frameworks like Django have Django REST Framework type of tooling built around it with [Graphene](https://github.com/graphql-python/graphene-django) reducing a lot of work but with respect to Clojure we need to write some more code for better control trade-off.
* Though the learning curve is pretty easy there is a paradigm shift involved in writing GraphQL queries after dealing with REST endpoints for a lot of years.
* Licensing is another major issue surrounding GraphQL though it was [relicensed](https://github.com/facebook/graphql/issues/351#issuecomment-332270583).

The code for the post is available at [GitHub](https://github.com/tirkarthi/lacinia-tutorial) . Kindly add in if I am wrong on any of the above or missed out anything in the post.