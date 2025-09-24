Schemas:

The purpose of schemas is to define the layout of the database in different ways. A schema defines what the tables of the database looks like, and can present the data stored in the database differently based on which schema is used. I can be imagined as different designs and ways to organize the same data. I can also be used to block access to data, like the "common_user" role might only be able to see "public" schema, which doesnt have all the tables of the "admin" schema.

Roles:

Roles allow the database to limit access to groups of users. Without roles, you'd need to define schema access and privileges to each user individually, but roles allows you to assign these automatically. 

Privileges:

Privileges is what a user can do in the database. The schema might determine that "common_user" has access to the "cars" table, but privileges define what the user can do with the table, like using SELECT, INSERT, UPDATE or DELETE, to mention a few. 

SERIAL:

Serial is an older way of managing identity of rows in a table. It creates a persistant schema object that manages giving out unique numbers for each row created. It's PostGreSQL specific.
The issue with using serial is that it doesn't get removed automatically when you drop the associated table, meaning you could potentially have lots of these staying around of creating/deleting is something that was done often. This might not be a huge problem, but even so, it's messy. 

IDENTITY:

Identity is part of SQL standard, and keeps the sequence within the table, meaning you avoid the issue of left behind objects when dropping a table. The sequence exists only in the table itself, and so is deleted whenever the table is dropped. This is a much cleaner way.

Arrays:

Enums:

JSON/JSONB:

