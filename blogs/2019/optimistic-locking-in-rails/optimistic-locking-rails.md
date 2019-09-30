# Optimistic vs. Pessimistic locking in Rails

To ensure data integrity, a relational database has to be ACID compliant.

ACID = Atomicity, Consistency, Isolation, Durability

While performing concurrent operations, a database must ensure data integrity. Locks ensures this data integrity and consistency. Locks can be at database, table, page or row level. Here is a begineer's guide to database locking in PostgreSQL.

In this article, we will see how Rails provides optimistic locking for ActiveRecord models. But before we proceed, let us first understand basics about optimistic and pessimistic locking.

## What is optimistic locking?

Let's say two admin users, Mohan and Ritesh, are managing the product inventory in their e-commerce portal. Mohan recently added a new laptop product in the inventory with title as "Macbook" and informed Ritesh. Upon noticing the title of this product, Ritesh started improving by editing it to be {title: Macbook Air}. Mohan simultaneously started editing the same product to have {title: Macboook Air 13"}. Mohan saved the product first and then Ritesh saves it. What will be the title of the product now? Yep, you guessed it that the title will be the one Ritesh saved as {title: Macbook Air} which leads to Mohan loosing a better and clearer title.

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

When multiple users are editing the same record, it's necessary to have a mechanism where the data is not lost or over written or at-least the user is notified. Optimistic locking is a mechanism to prevent data overrides by assuming that a database transaction conflict will rarely happen.

This locking mechanism uses "version number" column to track changes in each table that needs to implement concurrent access. Every-time a record in such a table changes, its version number is updated. If two users update a record simultaneously, only one of the user gets their changes accepted and other user will receive an error message because the version number won't match the version in the table. In the example above, Ritesh will get an alert that the title has been changed while he was editing it. So the application feature can now allow to refresh the page so that Ritesh see's the updated title for the product.

(Should I add a diagram here probably?)

### Please note that:
- Optimistic locking is just a mechanism to prevent processes from overwriting changes by another process. Optimistic locking is not a magic wand to manage or auto merge any conflicting changes. It can only allow users to alert or notify about such conflicting changes.
- Optimistic locking works by just comparing the value of the "version" column. Thus, optimistic locking is not a real database lock.


## What is pessimistic locking?
When one user is editing a record and we maintain an exclusive lock on that record, other user is prevented from modifying this record until the lock is released or transaction is completed. This explicit locking is known as a pessimistic lock. In our earlier example discussed above, when Ritesh started editing the title, Mohan will be prevented from editing the same product.

In majority of the cases, we would only need optimistic locking thereby avoiding pessimistic locking at the most. We also gain performance by moving from an explicit (pessimistic) lock to optimistic lock which checks the record at the time of an update instead of locking the record for entire transaction.

## How to implement optimistic locking in Rails?
It's fairly simple and straightforward to implement optimistic locking in Rails as ActiveRecord provides a built in mechanism.

By adding a lock_version column to any ActiveRecord model, Rails will automatically check this column before updating a record. Every update operation on the record increments the lock_version column value. If a record is instantiated twice, it will raise a StaleObjectError for the last saved record if the first record is already updated.

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

References and further reading:
https://api.rubyonrails.org/classes/ActiveRecord/Locking/Optimistic.html
https://sipsandbits.com/2018/05/30/optimistic-locking-of-activerecord-models/
https://gist.github.com/ryanermita/464bf88e2fc292e75c9353820c2f0475
