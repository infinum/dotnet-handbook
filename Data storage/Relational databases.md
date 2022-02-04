This is a most common way of storing data. Relational databases provide a good way to store your entities and relationships between them. So far we have worked with SQL Server and PostgreSQL databases.  

To choose a database server (SQLServer or PostgreSQL) we need to keep some things in mind. Regular operations performance is quite similar and for really small applications the difference is minimal. SQL Server supports more functionalities (excellent data encryption, spatial types etc.) but it is also more expensive in higher tiers. Express version has a free license with memory limitations (1GB RAM, 10GB database size) but PostgreSQL license is completely free. From developer's perspective, it is a bit easier to work with SQL Server due to the Management studio which is a very good tool. 

So, when choosing, answer these questions:

* Is the client using a specific tech stack (or wants one)?
* What will be the expected database size in the lifetime of the application?
* What are the application requirements and required database functionalities?
