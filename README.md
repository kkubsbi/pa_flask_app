
## Setting up the app
Following steps from the below:
https://blog.pythonanywhere.com/121/

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

```
10:58 ~/mysite (main)$ python
Python 3.10.5 (main, Jul 22 2022, 17:09:35) [GCC 9.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from flask_app import db
>>> db.create_all()
>>> exit()
```

You can confirm that this worked in your MySQL shell:

```
mysql> SHOW TABLES;
+----------------------------+
| Tables_in_kkubsbi$comments |
+----------------------------+
| comments                   |
| comments_ts                |
+----------------------------+
2 rows in set (0.01 sec)
```
