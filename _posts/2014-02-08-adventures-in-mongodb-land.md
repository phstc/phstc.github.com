---
layout: post
title: "[WIP] Adventures in MongoDB land"
---

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

## Expensive counts

When you have collections with million of documents, you will notice the searches aren't that fast, even when using indexes. It also applies for counts, if you use a simple count without any criteria, it will be fine, fast, because MongoDB knows the collection size, but [if you need to filter the count, it might be very expensive](https://www.google.com/webhp?#q=slow%20count%20mongodb):

    Notification.where(account_id: '5347c18a69702d1ef10c0000').count
    => # Completed in 15.354520466 seconds

It took 15 seconds to return the filtered count in a collection with 6120054 documents. To be fair with MongoDB, I have to mention that there are lot of documents (4951369) that matches the criteria above, which makes the query even more expensive, but it's a real use case in my application.

To make it faster, we created a `capped_count`, so instead of showing "4951369" found we show "99+".

    Notification.where(account_id: '5371ec6869702d131d010000').limit(-100).only(:_id).count(true)
    => # Completed in 0.021699112 seconds

Pretty fast, hum?

* `count(true)` - sets the [applySkipLimit](http://docs.mongodb.org/manual/reference/method/cursor.count/) = true, so the count respects the limit and skip when supplied. By default the count ignores the limit and skip.

* `limit(-100)` - a negative limit will ask [MongoDB to return that number and close the cursor](http://docs.mongodb.org/manual/reference/method/cursor.limit/).

* `only(:_id)` - returns only the _id.

For sure the `count(true)` is the most responsible for the performance improvement, but you will also notice that the negative limit and specifying only the _id to return helps with the performance too. Performance improvements is usually a lot of minor milliseconds improvements.

If you are using [Mongoid](http://mongoid.org/), you can add the `capped_count` to your Models.

    class Notification
      def self.capped_count(limit = -100)
        self.limit(limit).only(:_id).count(true)
      end
    end

    # Notification.where(account_id: '5371ec6869702d131d010000').capped_count

## mongorestore --drop

Use `--drop` for a full restore. By default the [mongorestore](http://docs.mongodb.org/manual/reference/program/mongorestore/) utility only inserts documents. Any existing data in the target database will be left intact. The `--drop` option drops every collection from the target database before restoring the collection from the dumped backup.

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

Log all messages via syslog to a separate mongodb.log:

    sudo vim /etc/asl/org.mongodb


    ? [= Sender mongod.27017] claim only
    * file /var/log/mongodb.log mode=0640 compress format=bsd rotate=seq file_max=5M all_max=20M

**or**

Save all messages to system.log:


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
