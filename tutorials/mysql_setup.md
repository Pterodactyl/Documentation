# Setting up MySQL
[[toc]]

MySQL is a core component of Pterodactyl Panel but it can be confusing to setup and use if you've never done so before.
This is a very basic tutorial that skims just enough of the surface to set MySQL up and running with the panel.
If you're interested in learning more, there are some great tutorials available on the Internet.

## Logging In
The first step in this process is to login to the MySQL command line where we will be executing some statements to get
things setup. To do so, simply run the command below and provide the Root MySQL account's password that you setup when
installing MySQL. If you do not remember doing this, chances are you can just hit enter as no password is set.

``` bash
mysql -u root -p
```

## Creating a User
For security sake, and due to changes in MySQL 5.7, you'll need to create a new user for the panel. To do so, we want
to first tell MySQL to use the mysql database, which stores such information.

Next, we will create a user called `pterodactyl` and allow logins from localhost which prevents any external connections
to our database. You can also use `%` as a wildcard or enter a numeric IP. We will also set the account password
to `somePassword`.

``` sql
USE mysql;

# Remember to change 'somePassword' below to be a unique password specific to this account.
CREATE USER 'pterodactyl'@'127.0.0.1' IDENTIFIED BY 'somePassword';
```

## Create a Database
Next, we need to create a database for the panel. In this tutorial we will be naming the database `panel`, but you can
substitute that for whatever name you wish.

``` sql
CREATE DATABASE panel;
```

## Assigning Permissions
Finally, we need to tell MySQL that our pterodactyl user should have access to the panel database. To do this, simply
run the command below. If you plan on also using this MySQL instance as a database host on the Panel you'll want to
include the `WITH GRANT OPTION` (which we are doing here). If you won't be using this user as part of the host setup
you can remove that.

``` sql
GRANT ALL PRIVILEGES ON panel.* TO 'pterodactyl'@'127.0.0.1' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
:::tip
The following is to add a database managed by the panel for its game servers
:::
## Adding Database to Pterodactyl Panel
In this guide we assume you have successfully installed the panel using the specified MariaDB as your database and it is on the `0.7.15` version.
## Adding DB user
First of all, you will need to add a user to your database. This is simply done by connecting to your database via CLI and running the following commands. You may need to enter a password if you specified one when connecting to the database. To connect to your database, enter:
```bash
mysql -u root -p
```
Once you are in the database you need to change the db schema by running
```sql
USE mysql;
```
To then create a user run.<span style="color:red"> Change `data` and `password` to keep your database safe! </span>
```sql
CREATE USER 'data'@'127.0.0.1' IDENTIFIED BY 'password';
```
Then add privileges for the user.
:::danger
If your database is on a different host than the one where your panel(daemon is installed, make sure to use `%` instead of `127.0.0.1` below. If your password contains special characters make sure to escape them
:::
```sql
GRANT ALL PRIVILEGES ON *.* TO 'data'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
Close your database connection with
```sql
EXIT;
```
You can now close your ssh session.
:::tip
Note. You might want to open your database to the internet. This is done by setting its bind-address to `0.0.0.0` instead of `127.0.0.1`
:::
## Adding the DB to the Panel
Go to `yourpaneldomain/admin/databases` and click `Create New`. This will open up a window in which you can add a database.
Fill the form, following this example:
- Name - A familiar name to reference the db at a later stage.
- Host - The server this database is hosted on.
- Port - Port of the database server.
- Username - Username of the user we created earlier.
- Password - Password of the user we created earlier.
- Linked node - Node the database is hosted on. (Default value can be retained)
Hit create and you are done!
## Common Errors
### DatabaseController.php:142
```
production.ERROR: ErrorException: Undefined variable: host in /var/www/pterodactyl/app/Http/Controllers/Admin/DatabaseController.php:142
```
The database user you are trying to use doesn't have appropriate grants/has used incorrect password.<br/>
```
[PDOException] SQLSTATE[HY000] [2006] MySQL server has gone away
```
The database server has closed the connection during a query. This would usually be caused by the SQL server being on the same server as the panel. Best way to resolve this issue is to set the IP to `127.0.0.1`
