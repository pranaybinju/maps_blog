# Optimistic vs. Pessimistic locking in Rails

An [ACID](https://en.wikipedia.org/wiki/ACID) compliant relational database ensures data integrity.

ACID = Atomicity, Consistency, Isolation, Durability

While performing concurrent operations, a database must ensure data integrity. The locking mechanism ensures data integrity and consistency. Locks can be a database, table, page or row level. [Here is a beginner's guide](https://vladmihalcea.com/a-beginners-guide-to-database-locking-and-the-lost-update-phenomena/) to database locking in PostgreSQL.

In this article, we will see how [Rails](https://rubyonrails.org/) provides a mechanism for optimistic locking on [ActiveRecord](https://guides.rubyonrails.org/active_record_basics.html) models. But before we proceed, let us first understand the basics about optimistic and pessimistic locking.

## What is optimistic locking?

Let's take an example of two admin users, Mohan and Ritesh, managing the product inventory in their e-commerce portal. Mohan recently added a new laptop product in the inventory with the title as "Macbook" and informed Ritesh. Upon noticing the title of this product, Ritesh started improving by editing it to be {title: Macbook Air}. Mohan simultaneously started editing the same product to have {title: MacBook Air 13"}. Mohan saved the product first and then Ritesh saves it. What will be the title of the product now? Yep, you guessed it that the title will be the one Ritesh saved as {title: Macbook Air} which leads to Mohan losing a better and clearer title.

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

Let's try and understand this with the flow diagram below.
[Image Attached]

When multiple users are editing the same record, it's necessary to have a mechanism where the data is not lost or overwritten or at-least the user is notified. Optimistic locking is a mechanism to prevent data overrides by assuming that a database transaction conflict will rarely happen.

[Image Attached]

Optimistic locking uses "version number" column to track changes in each table that needs to implement concurrent access. Every-time a record in such a table changes, its version number is updated. If two users update a record simultaneously, only one of the users gets their changes accepted and other users will receive an error message because the version number won't match the version in the table. In the example above, Ritesh will get an alert that the title has been changed while he was editing it. So the application feature can now allow refreshing the page so that Ritesh sees the updated title for the product.

### Please note that:

- Optimistic locking is just a mechanism to prevent processes from overwriting changes by another process. Optimistic locking is not a magic wand to manage or auto-merge any conflicting changes. It can only allow users to alert or notify about such conflicting changes.
- Optimistic locking works by just comparing the value of the "version" column. Thus, optimistic locking is not a real database lock.


## What is pessimistic locking?

When one user is editing a record and we maintain an exclusive lock on that record, another user is prevented from modifying this record until the lock is released or the transaction is completed. This explicit locking is known as a pessimistic lock. In our earlier example discussed above, when Ritesh started editing the title, Mohan will be prevented from editing the same product.

In the majority of the cases, we would only need optimistic locking thereby avoiding pessimistic locking at the most. We also gain performance by moving from an explicit (pessimistic) lock to an optimistic lock which checks the record at the time of an update instead of locking the record for the entire transaction.

## How to implement optimistic locking in Rails?

It's fairly simple and straightforward to implement optimistic locking in Rails as ActiveRecord provides a built-in mechanism.

By adding a `lock_version` column to any ActiveRecord model, Rails will automatically check this column before updating a record. Every update operation on the record increments the `lock_version` column value. If a record is instantiated twice, it will raise a `StaleObjectError` for the last saved record once the first record is updated.

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

### Source of the picture and flow diagram (thanks @nipunasilva)
https://nipunasilva.blogspot.com/2009/03/pessemestic-vs-optimistic-concurrency.html
