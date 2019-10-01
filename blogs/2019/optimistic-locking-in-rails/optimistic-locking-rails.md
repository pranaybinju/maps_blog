# Optimistic vs. Pessimistic locking in Rails

While performing concurrent operations, a database must ensure data integrity. [ACID](https://en.wikipedia.org/wiki/ACID) compliant relational database ensures this data integrity through its locking mechanism.

ACID = Atomicity, Consistency, Isolation, Durability

Locks can be at the database, table, page, or row level. [Here is a beginner's guide](https://vladmihalcea.com/a-beginners-guide-to-database-locking-and-the-lost-update-phenomena/) to database locking in [PostgreSQL](https://www.postgresql.org/about/).

In this article, let's see how [Rails](https://rubyonrails.org/) provides a mechanism for optimistic locking on [ActiveRecord](https://guides.rubyonrails.org/active_record_basics.html) models. However, before we proceed, let us first understand the basics of optimistic and pessimistic locking.

## What is optimistic locking?

Let's take an example of two admin users, Mohan and Ritesh, managing the product inventory in their e-commerce portal. Mohan recently added a new laptop product in the inventory with the title as "Macbook" and informed Ritesh. Upon noticing the title of this product, Ritesh started improving by editing it to be {title: Macbook Air}. Mohan simultaneously started editing the same product to have {title: MacBook Air 13"}. Mohan saved the product first and then Ritesh saves it. What is the title of the product now? Yep, you guessed it that the title would be the one Ritesh saved as {title: Macbook Air} which leads to Mohan losing a better and clearer title.

```
# Product.last.title = "Mackbook"

# user Mohan
user1_product = Product.last

# user Ritesh
user2_product = Product.last

user1_product.title = "Mackbook Pro 13"
user1_product.save #success

user2_product.title = "Macbook Pro"
user2_product.save #success

# Product.last.title = "Mackbook Pro" # and not "Mackbook Pro 13"
```

When multiple users are editing the same record, it's necessary to have a mechanism where the data is not lost or overwritten, or at least the user is notified. Optimistic locking is a mechanism to prevent data overrides by assuming that a database transaction conflict rarely happens.

[Image Attached: concurrency.jpg]

Optimistic locking uses a "version-number" column to track changes in each table that needs to implement concurrent access. Apart from the version number, other approaches involve dates, timestamps, checksums/hashes or even an entire state of the row. But we will discuss optimistic locking here using version number column. Every-time a record in such a table changes, its version number is updated. If two users update a record simultaneously, only one of the users gets their changes accepted and other users will receive an error message because the version number won't match the version in the table. In the example above, using optimistic locking, Ritesh gets an alert that the title has changed while he was editing it. Ritesh can refresh the page or front-end can be structured such that it shows the updated title while Ritesh is still editing it.

### Please note that:

- Optimistic locking is just a mechanism to prevent processes from overwriting changes by another process. Optimistic locking is not a magic wand to manage or auto-merge any conflicting changes. It can only allow users to alert or notify about such conflicting changes.
- Optimistic locking works by just comparing the value of the "version" column. Thus, optimistic locking is not a real database lock.


## What is pessimistic locking?

When one user is editing a record and we maintain an exclusive lock on that record, another user is prevented from modifying this record until the lock is released or the transaction is completed. This explicit locking is known as a pessimistic lock. In our earlier example discussed above, when Ritesh started editing the title, Mohan is prevented from editing the same product.

Let's try and understand these concepts with a flow diagram.
[Image Attached: ovsp_locks.jpg]

In the majority of the cases, we would only need optimistic locking, thereby avoiding pessimistic locking at the most. We also gain performance by moving from an explicit (pessimistic) lock to an optimistic lock which checks the record at the time of an update instead of locking the record for the entire transaction.

## How to implement optimistic locking in Rails?

It's relatively simple and straightforward to implement optimistic locking in Rails as ActiveRecord provides a built-in mechanism.

By adding a `lock_version` column to any ActiveRecord model, Rails automatically checks this column before updating a record. Every update operation on the record increments the `lock_version` column value. If a record is instantiated twice, it raises a `StaleObjectError` for the last saved record once the first record is updated.

From our earlier example:
```
# Product.last.title = "Mackbook"

# user Mohan
user1_product = Product.last

# user Ritesh
user2_product = Product.last

user1_product.title = "Mackbook Pro 13"
user1_product.save

user2_product.title = "Macbook Pro"
user2_product.save # Raises an ActiveRecord::StaleObjectError
```

Optimistic locking also checks for stale data when objects are destroyed and not just when they are updated.

[Rails API here](https://api.rubyonrails.org/classes/ActiveRecord/Locking/Optimistic.html) is an excellent source to know more about Optimistic locking in Rails and how to handle locking in a multi-process application. [Here](https://api.rubyonrails.org/classes/ActiveRecord/Locking/Pessimistic.html) is the guide to implement pessimistic locking in Rails.

### References and further reading

https://api.rubyonrails.org/classes/ActiveRecord/Locking/Optimistic.html
https://sipsandbits.com/2018/05/30/optimistic-locking-of-activerecord-models/
https://gist.github.com/ryanermita/464bf88e2fc292e75c9353820c2f0475

### Helpful video on GoRails
https://gorails.com/episodes/optimistic-locking-with-rails

### Source of the picture and flow diagram (thanks @nipunasilva)
https://nipunasilva.blogspot.com/2009/03/pessemestic-vs-optimistic-concurrency.html
