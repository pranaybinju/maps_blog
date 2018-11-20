### How to store emoji characters with Ruby on Rails & MySQL

Emojis has become an essential part of communication in our digital lives. Hence, as developers, our applications should provide first-class support to them. In this blog post, we will explore how to store them in our database.

What if you are getting an error when you try to insert emoji in your text? Isn't is irritating? Same was happening a few days back in one of our internal app we were facing the error, Whenever any user tries to insert emoji in the post, save was running infinitely and we were getting an error

> ActiveRecord::StatementInvalid (Mysql2::Error: Incorrect string value: '\xF0\x9F\x98\x8A ...' for column 'content' at row 1: INSERT INTO posts (content, subject, user_id, created_at, updated_at, url_token) VALUES

The issue was, the database is using encoding `UTF-8` which supports only `3 bytes per character`. But to save the emoji we require `4 bytes per character` and because of that `UTF-8` character set is not enough to store the emoji. The workaround to support 4 bytes character set is `utf8mb4` which was introduced by MySql in 2010. To support emoji we need to follow very simple steps which change the encoding from `utf8` to `utf8mb4`.

1. For safety purpose very important to take backup of your database and add following setting in `my.cnf`

```ruby
[mysqld]
default-storage-engine=InnoDB
innodb_file_format=barracuda
innodb_file_per_table=1
innodb_large_prefix=1
init_connect='SET collation_connection = utf8mb4_unicode_ci'
init_connect='SET NAMES utf8mb4'
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
```

2. Change the database.yml settings:
```ruby
encoding: utf8mb4
collate: utf8mb4_unicode_ci
```

`utf8mb4` allows the string to support 4 bytes per character,
`utf8mb4_unicode_ci` uses the Unicode Collation Algorithm as defined in the Unicode standards

3. Then add a migration to alter the database and table: Add this code in migration
```ruby
def db
  ActiveRecord::Base.connection
end

def alter_column_to_utf8mb(table, column)
  default = column.default.blank? ? '' : "DEFAULT '#{column.default}'"
  null = column.null ? '' : 'NOT NULL'
  execute "ALTER TABLE `#{table}` MODIFY `#{column.name}` #{column.sql_type.upcase}
    CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci #{default} #{null};"
end

def up
  execute "ALTER DATABASE `#{db.current_database}` CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;"
  db.tables.each do |table|
    execute "ALTER TABLE `#{table}` ROW_FORMAT=DYNAMIC CHARACTER SET
      utf8mb4 COLLATE utf8mb4_unicode_ci;"

    db.columns(table).each do |column|
      case column.sql_type
      when /([a-z]*)text/i
        alter_column_to_utf8mb(table, column)
      when /varchar\(([0-9]+)\)/i
        alter_column_to_utf8mb(table, column)
      end
    end
  end
end
```

First `ALTER` command changes the encoding and the `COLLATE` for the database for each table, we need to change the `CHARACTER SET` and `COLLATE` which we are doing in second `ALTER` command. After that, we need to `ALTER` the text and varchar columns to set `CHARACTER SET` and `COLLATE` with default and null attributes of the column.

4. Add initializer file config/initializers/ar_innodb_row_format.rb
```ruby
 require 'active_record/connection_adapters/abstract_mysql_adapter'

 module ActiveRecord
   module ConnectionAdapters
     class AbstractMysqlAdapter
       NATIVE_DATABASE_TYPES[:string] = { name: 'varchar', limit: 191 }
     end
   end
 end
```

This initializer insures to set column limit to `191` for the column type varchar at the time of executing `ALTER` or `CREATE` command on the table. Why length `191`? MySQL indexes are limited to `768 bytes` because the `InnoDB` storage engine has a maximum index length of `767 bytes`. This means that if we increase `VARCHAR(255)` from `3 bytes per character` to `4 bytes per character`, the index key is smaller in terms of characters. That's why we need to change the length from `255` to `191`.

😊 Hurray! we are ready to use emoji. Hope this will help! Feel free to share your thoughts, I'd love to hear those!

Reference link: https://mathiasbynens.be/notes/mysql-utf8mb4#utf8-to-utf8mb4
