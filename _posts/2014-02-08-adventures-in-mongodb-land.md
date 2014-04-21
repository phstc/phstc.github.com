---
layout: post
title: "[WIP] Adventures in MongoDB land"
tags: [MongoDB]
---
{% include JB/setup %}

Is it one more post about MongoDB thoughts? Yes, it is: Pablo's adventures in MongoDB-land™.

## Primary key on steroids

MongoDB's primary key `_id` by default is an [ObjectId](http://docs.mongodb.org/manual/reference/object-id/), which is automatically added to your documents while inserting them into a collection. 

But you can also set your own `_id` if you want.

     db.my_collection.insert({ _id: 'user@example.com' })

Have the ability of writing your own `_id` can be very useful. We have an internal system which uses MongoDB as a queue database. When we move messages across the queues, we preserve the original `_id`.

    // find a message to be processed 
    message = db.incoming.find_one({ _id: ObjectId("52f578c1ecf69b714400004a") })

    // some processing

    db.accepted.insert(message)
    // => ObjectId("52f578c1ecf69b714400004a")

    // remove the message from the incoming queue
    db.incoming.remove({ _id: message["_id"] })

So the message will have the same `_id` no matter where it is sitting on.

*MongoDB adds an index and a unique constraint on `_id` by default.*

### ObjectId

> ObjectId is a 12-byte BSON type, constructed using:
>
> a 4-byte value representing the seconds since the Unix epoch,
>
> a 3-byte machine identifier,
>
> a 2-byte process id, and
>
> a 3-byte counter, starting with a random value.

**a 4-byte value representing the seconds since the Unix epoch** means `created_at` field for free! Which we can use to sort, search and retrieve the creation date. 

#### Sort by _id

    // sort by created_at in descending order
    db.my_collection.find().sort({ _id: -1 })

    // sort by created_at in ascending order
    db.my_collection.find().sort({ _id: 1 })

#### _id to timestamp

    ObjectId("507f191e810c19729de860ea").getTimestamp()
    // => ISODate("2012-10-17T20:46:22Z")

Ruby example:

    ...
    <thead>
      <tr>
        <th>Id</th>
        <th>Foo</th>
        <th>Created at</th>
      </tr>
    </thread>
    <tbody>
      <% @documents.each do |document|
        <tr>
          <td><%= document['_id'] %></td>
          <td><%= document['foo'] %></td>
          <td><%= document['_id'].generation_time %></td>
        </tr>
      <% end %>
    </tbody>
    ...

##### Filter by _id

If you don't have a `created_at` field by your own and someone asks you to give a list of documents created between `something` and `something else`, can you do it? Yes, you can!

    time_something = Time.utc(2010, 1, 1)
    time_something_else = Time.utc(2010, 1, 10)

    time_something_id = ObjectId.from_time(time_something)
    time_something_else_id = ObjectId.from_time(time_something_else)

    collection.find('_id' => { '$gt' => time_something, '$lt' =>  time_something_else })

Unfortunately `ObjectId.from_time(time)` isn't available in the MongoDB Shell. But it is available in the [mongo-ruby-driver](http://api.mongodb.org/ruby/current/BSON/ObjectId.html#from_time-class_method). Ruby rocks! :)

## WARNING: Collection names should not begin with underscores

If you are using one of the most advanced backup techniques which consists in renaming collections with a underscore prefix, you should beware of that.

    db.my_collection.renameCollection('_my_collection')
    show collections
    // => _my_collections

    db._my_collections.find()
    // => Sat Feb  8 10:25:21.233 TypeError: Cannot call method 'find' of undefined

    db['_my_collections'].find()
    // => Sat Feb  8 10:25:21.233 TypeError: Cannot call method 'find' of undefined

MongoDB issue: [Can't reference collection names beginning with an underscore in the mongo shell
](https://jira.mongodb.org/browse/SERVER-445)

To retrieve the underscored collection you can use: `db.getCollection('_my_collection')`.

Underscore suffixes work fine:

    db.my_collection.renameCollection('my_collection_')
    db.my_collections_.find()
    // => success

## MongoDB likes your storage as much as you

### Log

Pay attention to the log files.

    du -h /usr/local/var/log/mongodb/mongo.log
    9.5G	/usr/local/var/log/mongodb/mongo.log

Short-term solution:

    rm  /usr/local/var/log/mongodb/mongo.log

Long-term solution: 

Edit your `/usr/local/etc/mongod.conf` and enable [Syslog Rotation](http://docs.mongodb.org/manual/tutorial/rotate-log-files/#syslog-log-rotation):

    # Append logs to /usr/local/var/log/mongodb/mongo.log
    # logpath = /usr/local/var/log/mongodb/mongo.log
    # logappend = true
    syslog = true

[On OS X >= 10.9.1 we need to set up some extra rules to enable syslog](https://groups.google.com/d/msg/mongodb-user/XyGzMZ4dTYY/XzjyLfdL7RIJ), otherwise it will not fail, but we will not be able to get the log outputs.

    sudo vim /etc/asl/org.mongodb

    # Log all messages via syslog to a separate mongodb.log
    ? [= Sender mongod.27017] claim only
    * file /var/log/mongodb.log mode=0640 compress format=bsd rotate=seq file_max=5M all_max=20M

or

     # Save all messages to system.log
     ? [= Sender mongod.27017] file /var/log/system.log

Then reload the syslogd configuration:

    pkill -HUP -f syslogd
     
### Local databases 

Check your local databases you might have some which you don't use anymore.

    du -h /usr/local/var/mongodb
    13G    /usr/local/var/mongodb

    mongo
    show dbs
    // => foo
    use foo
    db.dropDatabase()