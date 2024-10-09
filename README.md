
## Setting up the app
Following steps from the below:
https://blog.pythonanywhere.com/121/

Those instructions include adding git-based version control. To connect to your own remote GitHub repository, you need to:

1. Generate a new SSH key with `ssh-keygen -t ed25519 -C "your_email_goes_here@example.com"` (in the pythonanywhere server bash)
2. In the same shell, run `cat ~/.ssh/id_ed25519.pub` and copy the output
3. On the GitHub website, navigate to your [Settings > "SSH and GPG keys"](https://github.com/settings/keys), click "Add new SSH Key," and in the provided input, paste your public key.

Now, you are authenticated to your GH account. Next, connect the local repo to remote as follows:

1. Check that you've already run the git setup commands from the above PythonAnywhere tutorial.
2. Rename your branch: `git branch -m master main` - the tutorial uses 'master' which is outdated. Now, 'main' is the default branch main branch name.
3. Connect your local and remote directories: `git push -u origin main`

## Changing table schema

Changing schema will require migration. To keep things simple, you can follow these steps:

1. Add the new table (keeping the old one) to `flask_app.py`
2. Run `db.create_all()` from an interactive Python session on the server.
3. Migrate the data, using either a separate Python script, or the MySQL schell.
4. Remove the old table from `flask_app.py`
5. Drop the table from the MySQL DB

If you are just adding new tables, not replacing, you only need to do steps 1 and 2.

Here is an example. Let's say you are starting with the following in `flask_app.py`:

```python
class Comment(db.Model):

    __tablename__ = "comments"

    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.String(4096))
```

but you want to add a timestamp. So add a class to `flask_app.py`,
update all the usages accordingly, and check that it's workign with new data:

```python
class Comment(db.Model):

    __tablename__ = "comments"

    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.String(4096))
class CommentTS(db.Model):

    __tablename__ = "comments_ts"

    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.String(4096))
    timestamp = db.Column(db.DateTime, server_default=func.now())
```

Next, open an interactive Python session in the `mysite` directory, and execute the following 3 lines:

```sh
10:58 ~/mysite (main)$ python
Python 3.10.5 (main, Jul 22 2022, 17:09:35) [GCC 9.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from flask_app import db
>>> db.create_all()
>>> exit()
```

You can confirm that this worked in your MySQL shell:

```mysql
mysql> SHOW TABLES;
+----------------------------+
| Tables_in_kkubsbi$comments |
+----------------------------+
| comments                   |
| comments_ts                |
+----------------------------+
2 rows in set (0.01 sec)
```

Once you confirm that the app is working well with the new schema, you can migrate the old data. This can be done via Python, or directly via MySQL:

```mysql
INSERT INTO comments_ts (content, timestamp)
SELECT content, NOW()
FROM comments;
```

If that goes well - check that it is okay to avoid data loss! - you can drop the old table:

```mysql

mysql> select * from comments_ts;
+----+---------------------+---------------------+
| id | content             | timestamp           |
+----+---------------------+---------------------+
|  1 | test1               | 2024-10-09 11:13:35 |
|  2 | test2               | 2024-10-09 11:13:38 |
|  3 | test3               | 2024-10-09 11:13:40 |
|  4 | migrated1           | 2024-10-09 11:16:54 |
|  5 | migrated2           | 2024-10-09 11:16:54 |
|  6 | migrated3           | 2024-10-09 11:16:54 |
+----+---------------------+---------------------+
10 rows in set (0.01 sec)
mysql> drop table comments;
Query OK, 0 rows affected (0.06 sec)
mysql> show tables;
+----------------------------+
| Tables_in_kkubsbi$comments |
+----------------------------+
| comments_ts                |
+----------------------------+
1 row in set (0.00 sec)
```
