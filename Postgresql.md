# PostgreSQL Basics for GitLab

GitLab heavily relies on PostgreSQL for storing various types of data, including user information, repository metadata, issues, merge requests, and more.

## What is PostgreSQL?

PostgreSQL is a powerful, open-source relational database management system (RDBMS). It's known for its reliability, feature richness, and adherence to SQL standards. Key characteristics include:

* **Relational:** Data is organized into tables with defined relationships.
* **ACID Compliant:** Ensures data integrity through Atomicity, Consistency, Isolation, and Durability.
* **Extensible:** Supports custom functions, data types, and more.
* **Robust:** Proven to handle large datasets and high transaction volumes.

## Why PostgreSQL for GitLab?

GitLab chooses PostgreSQL for its stability, scalability, and advanced features, which are crucial for managing the complex data associated with software development workflows.

## Core Concepts

### 1. Databases

* A database is a container for organizing and managing collections of tables and other database objects.
* GitLab typically uses one main PostgreSQL database to store its application data.
* You can list databases using the `psql` command-line tool:

```bash
psql -l
```

**Output Example:**

```sql
 List of databases
	 Name    | Owner    | Encoding | Collate | Ctype | Access privileges
-----------+----------+----------+---------+-------+-------------------
 gitlabhq  | gitlab   | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
					 |          |          |         |       | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
					 |          |          |         |       | postgres=CTc/postgres
(4 rows)
```

### 2. Tables

* Tables are the fundamental structure for storing data in a relational database.
* They consist of rows (records) and columns (attributes).
* Each column has a specific data type (e.g., integer, text, boolean).

**Example Table Structure (Conceptual - GitLab has many tables):**

| Column     | Data Type | Description           |
| :--------- | :-------- | :-------------------- |
| id         | INTEGER   | Unique identifier     |
| name       | TEXT      | Project name          |
| created_at | TIMESTAMP | Project creation time |

### 3. Schemas

* Schemas are namespaces within a database that allow you to organize database objects (tables, views, functions, etc.).
* They help prevent naming conflicts when multiple users or applications use the same database.
* GitLab often uses the `public` schema by default, but it might also utilize other schemas for specific purposes.

### 4. SQL (Structured Query Language)

* SQL is the standard language for interacting with relational databases.
* Common SQL operations include:
    * **SELECT:** Retrieve data from tables.
        ```sql
        SELECT id, name FROM projects WHERE name LIKE 'gitlab%';
        ```
    * **INSERT:** Add new data into tables.
        ```sql
        INSERT INTO projects (name, created_at) VALUES ('new-project', NOW());
        ```
    * **UPDATE:** Modify existing data in tables.
        ```sql
        UPDATE projects SET name = 'renamed-project' WHERE id = 1;
        ```
    * **DELETE:** Remove data from tables.
        ```sql
        DELETE FROM projects WHERE id = 1;
        ```

### 5. Users and Roles

* PostgreSQL manages access to database objects through users and roles.
* Users are specific database identities that can log in.
* Roles can represent users or groups of users and can be granted specific privileges.
* GitLab typically has a dedicated PostgreSQL user (e.g., `gitlab`) with the necessary permissions to access and modify its database.

### 6. Connections

* Applications like GitLab connect to the PostgreSQL database server to interact with the data.
* Connection parameters usually include the host, port, database name, username, and password.
* GitLab's configuration files (e.g., `gitlab.rb` or environment variables) contain these connection details.

## Interacting with PostgreSQL (Basics)

You can interact with PostgreSQL using various tools:

* **`psql`:** A command-line interactive terminal for executing SQL queries.
    ```bash
    psql -U gitlab -h localhost -d gitlabhq
    ```
    (Replace `gitlabhq` with your GitLab database name if different).

* **GUI Tools:** Tools like pgAdmin provide a graphical interface for database management.

## Relevance to GitLab Administration

Understanding PostgreSQL basics is helpful for GitLab administrators for tasks such as:

* **Backup and Restore:** Creating backups of the PostgreSQL database is crucial for disaster recovery.
    ```bash
    # Backup
    pg_dump -U gitlab gitlabhq > gitlab_backup.sql

    # Restore (to a new database or after dropping the existing one)
    psql -U postgres -c "CREATE DATABASE gitlabhq_restored";
    psql -U gitlab -d gitlabhq_restored < gitlab_backup.sql
    ```
* **Performance Monitoring:** Analyzing database performance can help identify bottlenecks. Tools like `pg_stat_statements` can provide insights into query execution.
* **Troubleshooting:** When GitLab encounters issues, examining the PostgreSQL logs and potentially running queries can help diagnose problems.
* **Database Migrations:** GitLab performs database migrations during upgrades. Understanding the underlying database helps in comprehending these processes.

## Further Learning

This is a basic introduction. To delve deeper, consider exploring:

* **SQL Tutorials:** Numerous online resources are available for learning SQL.
* **PostgreSQL Documentation:** The official PostgreSQL documentation is comprehensive.
* **GitLab Documentation:** GitLab's documentation often provides specific information about its database usage.

By grasping these fundamental PostgreSQL concepts, you can better understand how GitLab stores and manages its data, aiding in administration, troubleshooting, and overall system comprehension.