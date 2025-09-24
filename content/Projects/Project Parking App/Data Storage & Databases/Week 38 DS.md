### Learnings

Schemas:
The purpose of a schema is to define namespaces for tables, which allows one to group tables. This can be useful when organizing. As an example, you might have a sales department and a development department. They have their own schemas, each with different tables, but a leader from sales might like to have access to certain development data, so by using a certain role, they might have access to both schemas and can query data from each. An alternative to schemas is having separate databases, but this complicates the leaders job, since they can't query across both databases in the same query.

Roles:
Roles allow the database to limit access to groups of users. Without roles, you'd need to define schema access and privileges to each user individually, but roles allow you to assign these automatically. You can also inherit properties from other roles, which makes it easier to make parent-child like privilege structures.

Privileges:
Privileges is what a user can do in the database. The schema might determine that "common_user" has access to the "cars" table, but privileges define what the user can do, like using SELECT, INSERT, UPDATE or DELETE, to mention a few. 

SERIAL:
Serial is an older way of managing identity of rows in a table. It creates a persistent sequence object that manages giving out unique numbers for each row created. It's PostgreSQL-specific.
The issue with using serial is that the sequence doesn't get removed automatically when you drop the associated table, meaning you could potentially have lots of these staying around if creating/deleting is done often. This might not be a huge problem, but even so, it's messy. 

IDENTITY:
Identity is part of SQL standard, and ties the sequence directly to the table, meaning you avoid the issue of left behind objects when dropping a table. The sequence exists only in the table itself, and so is deleted whenever the table is dropped. This is a much cleaner way and much more portable due to it being a standard.

Arrays:
When a column has the array type, it can hold multiple values in a sequence. This can be useful in many scenarios, like when you have data that can contain multiple values, like if a person has multiple phone numbers. 

Enums:
Enums as a type allow you to define what values specifically the column can hold. It's predefined, so the value in the field has to match one of the values written into the enum's type. An example could be a task status column that defines the values as "To do", "In progress" or "Done". If the entry doesn't contain one of these, it's invalid and gets rejected. This is very useful to enforce uniformity.

JSON/JSONB:
This type allows one to store json snippets as fields in a table. This can be useful in many scenarios, especially as a way to reduce translations between sources. If you know the data needs to be used as json, you can reduce errors by keeping it in the same format as it was received and as it needs to be read.

Even more so, it allows for flexible data, and can help keep non-uniformed structure. An example could be that one entry has data that doesn't fit into the table overall, but might be worth keeping. json/jsonb allows for that.

jsonb stores the data in a decomposed format, which also allows indexing and more efficient querying. 
### Tasks

#### Screenshots

![[Screenshot 2025-09-24 142939.png]]
![[Screenshot 2025-09-24 142925.png]]

#### Write-up


#### Reflection
BYTEA vs file paths for images

BYTEA:
pros:
* Allow the data to be stored in the table, removing the need for external storage solution.
* Images are included in the databases backup solution.
* No need for extra setup to send images, they can be pulled straight from the database.
* Easier permissions, can be managed with roles and privileges too.
* Better transactional integrity.
cons:
- inefficient/slow read/write
- Bloats the database
- Limited scalability

file paths:
pros:
- More efficient storage
- scales better
- keeps the database small and light
- More features that help performance, like image previews and range requests
- Can use different access tiers if stored in cloud.
cons:
- risk of broken links if images are deleted/moved
- risk of reference to wrong image
- need for extra backup solution
- more work required to handle permissions
- harder to migrate
