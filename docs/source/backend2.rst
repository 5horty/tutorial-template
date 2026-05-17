==============================================
Backend Documentation - Cook App Flask Application
==============================================

Overview
========

This is a Flask-based backend API for the Cook App mobile application. The backend provides RESTful endpoints for user authentication, account management, recipe creation, recipe search, recipe viewing, and step-by-step cooking progress tracking.

The backend is organized into several blueprints:
- Core modules (database management, path configuration, application factory)
- Authentication features (login, session management)
- Account features (user registration)
- Recipe features (listing, creation, image serving)
- Search features (recipe search by title)
- View recipe features (detailed recipe with step completion tracking)

Tech Stack
----------
- Framework: Flask (Python)
- Database: SQLite with foreign key constraints
- Image Handling: Binary BLOB storage in database
- Session Management: Flask session cookies
- Testing: pytest

Project Structure
-----------------

::

    backend/
    |-- main/
    |   |-- __init__.py
    |   |-- app.py                    # Application factory
    |   |-- database.py               # Database connection management
    |   |-- paths.py                  # Absolute path configuration
    |   |
    |   |-- account/
    |   |   |-- routes.py             # Account creation endpoints
    |   |   `-- sql/
    |   |       `-- create_user.sql   # SQL template for user creation
    |   |
    |   |-- authentication/
    |   |   |-- routes.py             # Login and session endpoints
    |   |   `-- sql/
    |   |       `-- get_user.sql      # SQL template for user lookup
    |   |
    |   |-- index/
    |   |   `-- routes.py             # Root endpoint (health check)
    |   |
    |   |-- recipe/
    |   |   |-- routes.py             # Recipe CRUD and image serving
    |   |   `-- sql/
    |   |       |-- add_ingredient.sql
    |   |       |-- add_recipe.sql
    |   |       |-- add_step.sql
    |   |       |-- get_main_image.sql
    |   |       |-- get_recipes.sql
    |   |       `-- get_step_image.sql
    |   |
    |   |-- search_recipe/
    |   |   |-- routes.py             # Recipe search endpoints
    |   |   `-- sql/
    |   |       |-- recipe_ingredients.sql
    |   |       `-- search_recipe.sql
    |   |
    |   `-- view_recipe/
    |       |-- routes.py             # Recipe detail and step tracking
    |       `-- sql/
    |           |-- complete_step.sql
    |           |-- get_recipe.sql
    |           |-- get_recipe_ingredients.sql
    |           |-- get_recipe_steps.sql
    |           `-- uncomplete_step.sql
    |
    |-- database/
    |   |-- schema.sql                # Database table definitions
    |   `-- test_data.sql             # Sample data for testing
    |
    `-- requirements.txt              # Python dependencies


Core Modules
============

paths.py - Path Configuration
-----------------------------

The **paths.py** module provides absolute path references to important
directories. It solves the common problem of file paths breaking when the
working directory changes.

**Code:**

::

    from pathlib import Path

    PROJECT_MAIN = Path(__file__).resolve().parent
    PROJECT_ROOT = PROJECT_MAIN.parent

**Explanation:**

This code uses Python's ``pathlib.Path`` to create absolute paths.

- ``__file__`` is a special variable that contains the path to the current
  file (``paths.py``). If you printed it, you might see something like
  ``/home/user/project/backend/main/paths.py``.

- ``.resolve()`` converts relative paths to absolute and resolves any symbolic
  links. This ensures we have the real, full path.

- ``.parent`` gives the directory containing the file. One ``.parent`` removes
  the filename, giving the directory. A second ``.parent`` goes up one more
  level.

Since ``paths.py`` lives inside the ``main/`` directory:

- ``PROJECT_MAIN`` becomes the absolute path to the ``main/`` directory.
  For example: ``/home/user/project/backend/main/``

- ``PROJECT_ROOT`` becomes the absolute path to the ``backend/`` directory
  (the parent of ``main/``). For example: ``/home/user/project/backend/``

**Why This Matters:**

Without these constants, code would use fragile relative paths:

::

    with open("recipe/sql/get_recipes.sql", 'r') as sql_file:  # Breaks if working directory changes!

If you ran the application from a different directory (like your home
directory or a test runner), relative paths would break because the
current working directory would be different.

By using absolute paths built from ``__file__``, the code becomes resilient
to working directory changes. All other modules import these constants to
build paths to SQL files, database files, and other resources.

**Usage Example:**

::

    from main.paths import PROJECT_MAIN

    with open(PROJECT_MAIN / "recipe/sql/get_recipes.sql", 'r') as sql_file:
        sql = sql_file.read()

The ``/`` operator in ``pathlib`` joins paths, so ``PROJECT_MAIN / "recipe"``
creates a path like ``/home/user/project/backend/main/recipe``. This works
the same on Windows, Linux, and macOS.


database.py - Database Connection Management
--------------------------------------------

The **database.py** module provides a context manager for SQLite database
connections and handles automatic database initialisation.

**Class Definition and Constructor:**

::

    class Database:
        def __init__(self, app):
            self.is_testing = app.config.get('TESTING', False)

            if not Path(self.get_database_path()).exists():
                self.create_new_database()

**Explanation:**

The ``Database`` class is designed to work with Flask's ``current_app``.

- The constructor takes a Flask ``app`` object as a parameter. This allows it
  to read configuration values like ``TESTING``.

- ``self.is_testing`` checks whether the app is in testing mode. This is set
  in the Flask config (``app.config['TESTING'] = True``).

- The constructor then checks if the database file exists by calling
  ``get_database_path()``. If the file doesn't exist, it automatically calls
  ``create_new_database()`` to create the database from scratch.

**Context Manager Methods:**

::

    def __enter__(self):
        self.con = connect(self.get_database_path())
        self.con.execute("PRAGMA foreign_keys = ON")
        self.con.row_factory = Row
        return self.con

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.con.close()

**Explanation:**

These methods enable the ``with`` statement pattern. The ``__enter__`` method
runs when entering a ``with`` block, and ``__exit__`` runs when leaving.

Inside ``__enter__``:

- ``connect(self.get_database_path())`` opens a SQLite connection to the
  database file (either ``database.db`` or ``test_database.db``).

- ``self.con.execute("PRAGMA foreign_keys = ON")`` enables foreign key
  constraints. SQLite disables them by default for backwards compatibility,
  so this line turns them on.

- ``self.con.row_factory = Row`` sets the row factory to ``sqlite3.Row``.
  This allows rows to be accessed as dictionaries (e.g., ``row["column_name"]``)
  instead of just tuples.

- The connection object is returned, so inside the ``with`` block you can do
  ``cur = con.cursor()``.

Inside ``__exit__``:

- ``self.con.close()`` closes the database connection. This happens
  automatically even if an exception occurred inside the ``with`` block,
  ensuring no connection leaks.

**Database Creation Method:**

::

    def create_new_database(self):
        con = connect(self.get_database_path())
        con.execute("PRAGMA foreign_keys = ON")
        cur = con.cursor()

        with open(self.get_database_schema()) as schema:
            sql = schema.read()
            cur.executescript(sql)

        with open(self.get_database_test_data()) as test_data:
            sql = test_data.read()
            cur.executescript(sql)

        con.commit()
        con.close()

**Explanation:**

This method creates a brand new database from scratch.

- First, it opens a connection to the database file (which may be empty or
  not exist yet).

- ``cur.executescript(sql)`` runs multiple SQL statements at once. This is
  different from ``execute()`` which only runs one statement.

- ``get_database_schema()`` returns the path to ``database/schema.sql``, which
  contains all the ``CREATE TABLE`` statements.

- ``get_database_test_data()`` returns the path to ``database/test_data.sql``,
  which contains sample data (``INSERT`` statements) for development and testing.

- ``con.commit()`` saves all changes to disk. Without this, the changes would
  be lost when the connection closes.

- ``con.close()`` closes the connection.

**Helper Methods:**

::

    def get_database_path(self):
        return PROJECT_MAIN / (
            "test_database.db" if self.is_testing else "database.db"
        )

    @staticmethod
    def delete_test_database():
        test_db = PROJECT_MAIN / "test_database.db"
        if Path(test_db).exists():
            remove(test_db)

    @staticmethod
    def get_database_schema():
        return PROJECT_ROOT / "database/schema.sql"

    @staticmethod
    def get_database_test_data():
        return PROJECT_ROOT / "database/test_data.sql"

**Explanation:**

- ``get_database_path()`` returns either ``main/test_database.db`` (when
  ``is_testing`` is True) or ``main/database.db`` (for normal operation).
  The ``/`` operator joins paths.

- ``delete_test_database()`` is a static method (belongs to the class, not
  instances). It deletes the test database file. This is useful for cleaning
  up between test runs.

- ``get_database_schema()`` returns the path to the schema file in the
  ``database/`` directory at the project root.

- ``get_database_test_data()`` returns the path to the test data file.

**Usage Pattern:**

::

    from flask import current_app
    from main.database import Database

    @some_bp.route('/example')
    def example():
        with Database(current_app) as con:
            cur = con.cursor()
            cur.execute("SELECT * FROM recipe")
            results = cur.fetchall()
        return {"data": [dict(row) for row in results]}

**Explanation of Usage:**

- ``Database(current_app)`` creates a new database instance. The Flask app is
  passed in so the database can read configuration (like ``TESTING``).

- The ``with`` statement calls ``__enter__``, which opens the connection and
  returns it as ``con``.

- Inside the block, you can execute queries using the cursor.

- When the block ends, ``__exit__`` automatically closes the connection.

- ``dict(row)`` converts each Row object to a regular Python dictionary,
  which Flask can automatically convert to JSON.

The context manager ensures the connection is always closed, even if an
exception occurs during query execution.


app.py - Application Factory
----------------------------

The **app.py** module creates and configures the Flask application instance.

**Code:**

::

    from flask import Flask
    from main.database import Database
    from main.account.routes import account_bp
    from main.authentication.routes import authentication_bp
    from main.index.routes import index_bp
    from main.recipe.routes import recipe_bp
    from main.search_recipe.routes import search_bp
    from main.view_recipe.routes import view_recipe_bp


    def create_app() -> Flask:
        app = Flask(__name__)
        app.secret_key = 'SECRET-KEY'
        app.config['MAX_CONTENT_LENGTH'] = 16 * 1000 * 1000

        app.register_blueprint(recipe_bp)
        app.register_blueprint(authentication_bp)
        app.register_blueprint(index_bp)
        app.register_blueprint(account_bp)
        app.register_blueprint(search_bp)
        app.register_blueprint(view_recipe_bp)

        with Database(app) as _:
            pass

        return app

**Explanation:**

The function starts by importing all the necessary blueprints and the Database
class. Notice that each blueprint is imported from its respective module.

**Application Creation:**

::

    app = Flask(__name__)

- ``Flask(__name__)`` creates a new Flask application instance. The
  ``__name__`` argument helps Flask locate resources (templates, static files)
  relative to this module. Since this is the application factory, ``__name__``
  is the name of the module containing this code.

**Configuration:**

::

    app.secret_key = 'SECRET-KEY'
    app.config['MAX_CONTENT_LENGTH'] = 16 * 1000 * 1000

- ``secret_key`` is used by Flask to sign session cookies. This prevents users
  from tampering with their session data. In production, this should be loaded
  from an environment variable rather than hard‑coded.

- ``MAX_CONTENT_LENGTH`` limits the size of incoming request bodies to 16
  megabytes (16 × 1000 × 1000 bytes). This prevents attackers from uploading
  extremely large files that could exhaust server memory.

**Blueprint Registration:**

::

    app.register_blueprint(recipe_bp)
    app.register_blueprint(authentication_bp)
    app.register_blueprint(index_bp)
    app.register_blueprint(account_bp)
    app.register_blueprint(search_bp)
    app.register_blueprint(view_recipe_bp)

Each blueprint is registered with the app. The order doesn't matter because
blueprints are isolated namespaces. When a blueprint is registered, all its
routes become available on the app. For example, if ``recipe_bp`` has a route
``/get-recipes``, that route becomes available at that exact path.

**Database Initialisation:**

::

    with Database(app) as _:
        pass

This line opens a database connection using the ``Database`` context manager.
The ``_`` variable is a convention meaning "I don't need this value".

The important thing happens in the ``Database`` constructor:
- The constructor checks whether the database file exists
- If it doesn't, it calls ``create_new_database()`` to create it with the full
  schema and test data

The connection is opened and then immediately closed (because the ``with``
block does nothing). However, by the time this line finishes, the database
file is guaranteed to exist with all tables created.

**Return:**

::

    return app

The fully configured Flask application is returned. This allows different
environments (development, testing, production) to create the app with
different configurations.

**Why the Application Factory Pattern?**

The application factory pattern allows multiple instances of the app to be
created without global state interfering between them. This is especially
useful for:

- **Testing:** You can create a test app with different configuration
  (like using a test database) for each test.

- **Multiple configurations:** You could have different configurations for
  development, staging, and production.

- **Avoiding global state:** Blueprints can be imported without creating the
  app, which avoids circular import problems.


Database Schema (schema.sql)
============================

The **schema.sql** file defines the complete database structure. Let's examine
each table with its code and explanation.

**Table: user**

::

    CREATE TABLE user (
        user_id INTEGER PRIMARY KEY AUTOINCREMENT,
        user_fname TEXT NOT NULL,
        user_lname TEXT NOT NULL,
        user_password TEXT NOT NUll,
        user_email TEXT NOT Null
    );

**Explanation:**

This table stores registered user accounts.

- ``user_id INTEGER PRIMARY KEY AUTOINCREMENT``: Each user gets a unique number
  that automatically increments when a new row is inserted. This is the primary
  key used to reference users from other tables.

- ``user_fname TEXT NOT NULL``: The user's first name. ``NOT NULL`` means this
  field cannot be empty.

- ``user_lname TEXT NOT NULL``: The user's last name. Also required.

- ``user_password TEXT NOT NULL``: The user's password. Currently stored as
  plain text (acceptable for a university project, but should be hashed in
  production).

- ``user_email TEXT NOT NULL``: The user's email address.

**Table: dietary_requirement**

::

    CREATE TABLE dietary_requirement (
        dietary_requirement_id INTEGER PRIMARY KEY AUTOINCREMENT,
        dietary_requirement_name TEXT NOT NULL
    );

**Explanation:**

A lookup table of possible dietary requirements. This is a reference table
that contains values like "Vegetarian", "Vegan", "Gluten Free", etc.

- ``dietary_requirement_id``: Auto-incrementing unique identifier for each
  dietary requirement.

- ``dietary_requirement_name``: The name of the requirement (e.g., "Vegetarian").

**Table: user_dietary_requirement**

::

    CREATE TABLE user_dietary_requirement (
        user_id INTEGER NOT NULL,
        dietary_requirement_id INTEGER NOT NULL,
        FOREIGN KEY (user_id) REFERENCES user (user_id),
        FOREIGN KEY (dietary_requirement_id) REFERENCES dietary_requirement (dietary_requirement_id),
        PRIMARY KEY (user_id, dietary_requirement_id)
    );

**Explanation:**

This is a junction table (also called an association table or linking table)
that creates a many-to-many relationship between users and dietary requirements.

- A user can have many dietary requirements (vegetarian AND gluten free)
- A dietary requirement can apply to many users (many people can be vegetarian)

The two foreign keys reference the parent tables:
- ``FOREIGN KEY (user_id) REFERENCES user (user_id)`` ensures every user_id
  exists in the user table.
- ``FOREIGN KEY (dietary_requirement_id) REFERENCES dietary_requirement (dietary_requirement_id)``
  ensures every dietary_requirement_id exists in the dietary_requirement table.

The composite primary key ``PRIMARY KEY (user_id, dietary_requirement_id)``
prevents duplicate associations - the same user cannot be linked to the same
dietary requirement twice.

**Table: recipe**

::

    CREATE TABLE recipe (
        recipe_id INTEGER PRIMARY KEY AUTOINCREMENT,
        recipe_title TEXT NOT NULL,
        recipe_time TEXT,
        recipe_difficulty TEXT,
        recipe_main_image BLOB
    );

**Explanation:**

This table stores the core recipe information.

- ``recipe_id``: Auto-incrementing unique identifier for each recipe.

- ``recipe_title TEXT NOT NULL``: The name of the recipe. This is required.

- ``recipe_time TEXT``: The preparation/cooking time (free text like "30 min").
  This can be NULL if not specified.

- ``recipe_difficulty TEXT``: Difficulty level (free text like "Easy", "Medium",
  "Hard"). Can be NULL.

- ``recipe_main_image BLOB``: The main image for the recipe stored as binary
  data. BLOB stands for Binary Large Object. This stores the actual JPEG bytes
  directly in the database.

**Table: completed_recipe**

::

    CREATE TABLE completed_recipe (
        completed_recipe_id INTEGER NOT NULL,
        recipe_id INTEGER NOT NULL,
        user_id INTEGER NOT NULL,
        recipe_completion_date TEXT NOT NULL DEFAULT (current_timestamp),
        completed_recipe_photo BLOB,
        FOREIGN KEY (recipe_id) REFERENCES recipe (recipe_id),
        FOREIGN KEY (user_id) REFERENCES user (user_id)
    );

**Explanation:**

This table tracks which users have completed which recipes.

- ``completed_recipe_id``: Identifier for each completion record. Note this is
  NOT auto-incrementing, so the application must provide a value.

- ``recipe_id``: References the recipe that was completed.

- ``user_id``: References the user who completed it.

- ``recipe_completion_date TEXT NOT NULL DEFAULT (current_timestamp)``: The
  date and time when the recipe was completed. The ``DEFAULT (current_timestamp)``
  means if no value is provided, SQLite automatically uses the current date
  and time.

- ``completed_recipe_photo BLOB``: An optional photo of the finished dish,
  stored as binary data.

**Table: recipe_ingredient**

::

    CREATE TABLE recipe_ingredient (
        recipe_ingredient_id INTEGER PRIMARY KEY AUTOINCREMENT,
        recipe_id INTEGER NOT NULL,
        recipe_ingredient_name TEXT NOT NULL,
        recipe_ingredient_amount INTEGER NOT NULL DEFAULT (1),
        recipe_ingredient_unit TEXT,
        recipe_ingredient_calories INTEGER,
        FOREIGN KEY (recipe_id) REFERENCES recipe (recipe_id)
    );

**Explanation:**

This table stores the ingredients for each recipe.

- ``recipe_ingredient_id``: Auto-incrementing unique identifier for each
  ingredient entry.

- ``recipe_id``: References which recipe this ingredient belongs to.

- ``recipe_ingredient_name TEXT NOT NULL``: The name of the ingredient (e.g.,
  "Flour", "Eggs", "Butter").

- ``recipe_ingredient_amount INTEGER NOT NULL DEFAULT (1)``: The quantity needed.
  Defaults to 1 if not specified.

- ``recipe_ingredient_unit TEXT``: The unit of measurement (e.g., "g", "cup",
  "tbsp", "whole"). Can be NULL for items counted individually.

- ``recipe_ingredient_calories INTEGER``: The calorie count for this ingredient
  (in the specified amount). Can be NULL.

**Table: recipe_step**

::

    CREATE TABLE recipe_step (
        recipe_step_id INTEGER PRIMARY KEY AUTOINCREMENT,
        recipe_id INTEGER NOT NULL,
        recipe_step_index TEXT NOT NULL,
        recipe_step_description TEXT NOT NULL,
        recipe_step_duration TEXT,
        recipe_step_image BLOB,
        FOREIGN KEY (recipe_id) REFERENCES recipe (recipe_id)
    );

**Explanation:**

This table stores the individual cooking steps for each recipe.

- ``recipe_step_id``: Auto-incrementing unique identifier for each step.

- ``recipe_id``: References which recipe this step belongs to.

- ``recipe_step_index TEXT NOT NULL``: The step number or order. Using TEXT
  instead of INTEGER allows substep numbering like "1", "2", "2.1", "2.2", "3".
  This is more flexible than simple integers.

- ``recipe_step_description TEXT NOT NULL``: The instructions for this step.

- ``recipe_step_duration TEXT``: How long this step takes, stored in ISO 8601
  duration format (e.g., "PT5M" for 5 minutes, "PT1H30M" for 1 hour 30 minutes).

- ``recipe_step_image BLOB``: An optional image for this specific step,
  stored as binary data.

**Table: recipe_step_completion**

::

    CREATE TABLE recipe_step_completion (
        recipe_step_id INTEGER NOT NULL,
        user_id INTEGER NOT NULL,
        FOREIGN KEY (recipe_step_id) REFERENCES recipe_step (recipe_step_id),
        FOREIGN KEY (user_id) REFERENCES user (user_id),
        PRIMARY KEY (recipe_step_id, user_id)
    );

**Explanation:**

This is the core table for the guided cooking mode's progress tracking.

- ``recipe_step_id``: References which step was completed.

- ``user_id``: References which user completed the step.

The composite primary key ensures each user can mark a step as completed only
once. The **presence** of a row means the step is completed. The **absence**
of a row means the step is not completed. This is more efficient than storing
a boolean flag because it avoids the need to create rows for every step-user
combination (which would be millions of rows for many users and steps).

**Table: recipe_dietary_requirement**

::

    CREATE TABLE recipe_dietary_requirement (
        recipe_ingredient_id INTEGER NOT NULL,
        dietary_requirement_id INTEGER NOT NULL,
        FOREIGN KEY (recipe_ingredient_id) REFERENCES recipe_ingredient (recipe_ingredient_id),
        FOREIGN KEY (dietary_requirement_id) REFERENCES dietary_requirement (dietary_requirement_id),
        PRIMARY KEY (recipe_ingredient_id, dietary_requirement_id)
    );

**Explanation:**

This junction table links ingredients to dietary requirements. For example,
if an ingredient contains gluten, it would be linked to the "Gluten free"
dietary requirement (meaning this ingredient breaks a gluten-free diet).

- ``recipe_ingredient_id``: References an ingredient from the
  ``recipe_ingredient`` table.

- ``dietary_requirement_id``: References a dietary requirement from the
  ``dietary_requirement`` table.

This allows the application to filter recipes by dietary restrictions:
find all recipes where none of their ingredients are linked to a
restricted dietary requirement.

**Key Design Decisions:**

1. **BLOB Storage for Images:** Images are stored directly in the database as
   binary BLOBs rather than on the filesystem. This simplifies deployment
   (no separate file server needed) and backup (one file contains everything).
   The downside is that database size grows quickly and serving images is
   slightly slower.

2. **TEXT for recipe_step_index:** Using TEXT instead of INTEGER allows
   substep numbering like "1.1", "1.2", "2.1" rather than just sequential
   integers. This gives the frontend more flexibility in displaying steps.

3. **PRAGMA foreign_keys = ON:** The database module enables foreign key
   enforcement on every connection. Without this, SQLite allows deleting a
   recipe even if it has steps, which would leave orphaned rows.

4. **Composite Primary Keys:** Junction tables use composite primary keys
   (both foreign keys together) to prevent duplicate associations. You cannot
   link the same user to the same dietary requirement twice.

5. **Default Timestamp:** The ``completed_recipe`` table uses
   ``DEFAULT (current_timestamp)`` to automatically record when a recipe was
   completed, so the application doesn't need to provide this value.


Authentication Blueprint (authentication.py)
============================================

The **authentication.py** module defines a Flask blueprint for user login and
session management.

**Blueprint Creation:**

::

    from flask import blueprints, current_app, request, abort, session

    from main.database import Database
    from main.paths import PROJECT_MAIN

    authentication_bp = blueprints.Blueprint('authentication', __name__)

**Explanation:**

- The blueprint is created with name ``'authentication'`` and import name
  ``__name__``. This creates a namespace for all authentication-related routes.

- Various Flask imports are needed: ``blueprints`` for creating the blueprint,
  ``current_app`` for accessing the Flask app (to get database configuration),
  ``request`` for accessing form data, ``abort`` for returning error responses,
  and ``session`` for managing user session.

- The ``Database`` class and ``PROJECT_MAIN`` path are imported from other
  modules.

**Route: authenticate**

::

    @authentication_bp.route('/authenticate', methods=['POST'])
    def authenticate():
        # Validate form first
        if "user_fname" not in request.form:
            abort(400)

        if "user_lname" not in request.form:
            abort(400)

        if "user_password" not in request.form:
            abort(400)

**Explanation:**

The decorator ``@authentication_bp.route('/authenticate', methods=['POST'])``
tells Flask that this function handles POST requests to the path
``/authenticate``.

The validation code checks that all required fields exist in the form data.
``request.form`` is a dictionary-like object containing the form fields sent
by the client.

- If ``"user_fname"`` is missing, ``abort(400)`` immediately returns an HTTP
  400 Bad Request response.

- The same check is done for ``"user_lname"`` and ``"user_password"``.

- Unlike the account blueprint, this does NOT check for empty strings. An empty
  name would simply result in no user match later (returning 401).

**Extracting Values and Database Query:**

::

        user_fname = request.form['user_fname']
        user_lname = request.form['user_lname']
        user_password = request.form['user_password']

        with Database(current_app) as con:
            cur = con.cursor()
            with open(PROJECT_MAIN / "authentication/sql/get_user.sql", 'r') as sql_file:
                sql = sql_file.read()
                cur.execute(sql, (user_fname, user_lname))

            user = cur.fetchone()

**Explanation:**

- The form values are extracted into local variables.

- ``with Database(current_app) as con:`` opens a database connection. The
  ``current_app`` global gives access to the Flask app that is handling this
  request. The database uses the app's configuration (like the ``TESTING`` flag).

- ``cur = con.cursor()`` creates a cursor object used to execute SQL.

- The SQL template file is opened, read, and executed. ``PROJECT_MAIN / "authentication/sql/get_user.sql"``
  constructs the absolute path to the SQL file using the path constants.

- ``cur.execute(sql, (user_fname, user_lname))`` executes the query. The second
  argument is a tuple of parameters. The ``?`` placeholders in the SQL are
  replaced with these values in a safe, SQL-injection-preventing way.

- ``user = cur.fetchone()`` retrieves the first row of results, or ``None``
  if no user matches.

**Checking User and Password:**

::

            if user is None:
                return {"success": False}, 401

            user = dict(user)

            if user["user_password"] != user_password:
                return {"success": False}, 401

**Explanation:**

- If no user was found (``user is None``), the function returns a JSON object
  ``{"success": False}`` with HTTP status 401 Unauthorized. The same response
  is used for wrong passwords to avoid revealing whether the username exists.

- ``user = dict(user)`` converts the Row object to a regular Python dictionary.
  This allows accessing fields with bracket notation like ``user["user_password"]``.

- The submitted password is compared to the stored password. Note that this
  compares plain text passwords. In a production application, you would use
  ``bcrypt.checkpw()`` to compare hashed passwords.

**Setting Session and Returning Success:**

::

            del user["user_password"]

        session["user_id"] = user["user_id"]
        return {"success": True, "user": user}

**Explanation:**

- ``del user["user_password"]`` removes the password from the dictionary before
  sending it to the client. The password should never leave the server.

- The database connection is automatically closed when the ``with`` block ends.

- ``session["user_id"] = user["user_id"]`` stores the user's ID in the Flask
  session. Flask signs this data with the secret key and sends it as a cookie
  to the client. On subsequent requests, the client sends this cookie back,
  and Flask restores the session data.

- The function returns a success response with the user object (minus the
  password) and HTTP status 200 OK (default).

**SQL Template (get_user.sql):**

::

    SELECT
        user_id, user_password
    FROM
        user
    WHERE
        user_fname like ? AND user_lname like ?
    limit 1;

**Explanation:**

- This query selects only the ``user_id`` and ``user_password`` columns.
  It does not select the email or other fields because they aren't needed for
  authentication.

- ``LIKE ?`` with no wildcards (like ``%``) performs a case‑insensitive
  comparison in SQLite (depending on collation settings). This means
  "john" matches "JOHN" and "John".

- ``limit 1`` ensures at most one row is returned. Even if there were
  duplicate users (which shouldn't happen), only the first is considered.

**Important Security Note:**

The current implementation stores and compares passwords as plain text. This
is NOT secure and should only be used for a university project where simplicity
is valued over security. In production, you should:

1. Hash passwords when creating accounts using ``bcrypt.hashpw()``
2. Store the hash in the database
3. Compare using ``bcrypt.checkpw()`` during login


Account Blueprint (account.py)
==============================

The **account.py** module defines a Flask blueprint for user account creation.

**Blueprint Creation:**

::

    from flask import blueprints, current_app, request, abort

    from main.database import Database
    from main.paths import PROJECT_MAIN

    account_bp = blueprints.Blueprint('account', __name__)

**Explanation:**

Similar to the authentication blueprint, this creates a blueprint named
``'account'``. The same imports are needed, plus a comment indicating that
dietary preferences should be added in the future.

**Route: create_account**

::

    @account_bp.route('/create-account', methods=['POST'])
    def create_account():
        # todo: include dietary preferences
        # Validate form first
        if not request.form:
            abort(400)

        if not request.form.get("user_fname"):
            abort(400)

        if not request.form.get("user_lname"):
            abort(400)

        if not request.form.get("user_email"):
            abort(400)

        if not request.form.get("user_password"):
            abort(400)

**Explanation:**

The ``todo`` comment indicates that dietary preferences should be added to the
account creation process in the future.

The validation is stricter than the authentication route:

- ``if not request.form:`` checks that the form is not empty. If the client
  sent no data at all, this returns 400.

- ``if not request.form.get("user_fname"):`` checks that the field exists AND
  is not an empty string. The ``.get()`` method returns ``None`` if the key
  doesn't exist, and the empty string ``""`` is falsy, so both cases are caught.

- The same validation is applied to last name, email, and password.

**Extracting Values and Database Insertion:**

::

        user_fname = request.form['user_fname']
        user_lname = request.form['user_lname']
        user_password = request.form['user_password']
        user_email = request.form['user_email']

        with Database(current_app) as con:
            cur = con.cursor()
            with open(PROJECT_MAIN / "account/sql/create_user.sql", 'r') as sql_file:
                sql = sql_file.read()
                cur.execute(sql, (user_fname, user_lname, user_email, user_password))

                con.commit()

            user_id = cur.lastrowid

**Explanation:**

- The validated form values are extracted into variables.

- A database connection is opened with the context manager.

- The SQL template is read from the file system.

- ``cur.execute(sql, (user_fname, user_lname, user_email, user_password))``
  executes the INSERT statement. The four ``?`` placeholders are replaced with
  the user's data.

- ``con.commit()`` saves the transaction. Without this, the INSERT would be
  rolled back when the connection closes.

- ``cur.lastrowid`` retrieves the auto-incremented ``user_id`` that was
  assigned to the new user. This is useful for the frontend to know what ID
  the new user has.

**Return Response:**

::

        return {
            "user_id": user_id
        }, 201

**Explanation:**

The function returns a JSON object containing the new user's ID with HTTP
status 201 Created. The 201 status code indicates that a resource was
successfully created.

**SQL Template (create_user.sql):**

::

    INSERT INTO user (user_fname, user_lname, user_email, user_password)
    VALUES (?, ?, ?, ?);

**Explanation:**

This is a parameterised INSERT statement. The ``?`` placeholders are replaced
with the values passed to ``cur.execute()``. Using parameterised queries
prevents SQL injection attacks because the values are treated as data, not
as part of the SQL command.

**Important Note on Passwords:**

Like the authentication blueprint, this stores the password in plain text.
This should be replaced with bcrypt hashing in production.

**Missing Features:**

- No email validation (doesn't check if email is valid format or already exists)
- No password strength requirements
- No dietary preference collection (noted in the todo comment)


Index Blueprint (index.py)
==========================

The **index.py** module defines the root endpoint for the application.

**Blueprint Creation and Route:**

::

    from flask import blueprints

    index_bp = blueprints.Blueprint('index', __name__)

    @index_bp.route('/')
    def index():
        return {
            "message": "Hello World!"
        }

**Explanation:**

This is the simplest blueprint in the application.

- The blueprint is named ``'index'``.

- The ``@index_bp.route('/')`` decorator binds this function to the root path
  ``/`` with the default HTTP method GET.

- The function returns a JSON object directly. Flask automatically converts
  dictionaries to JSON responses.

- No authentication is required, making this a public health check endpoint.

**Purpose:**

This endpoint serves two purposes:

1. **Health Check:** When deploying the application, you can hit ``/`` to
   verify the server is running.

2. **Welcome Message:** Provides a simple confirmation that the API is
   accessible.

**Example Response:**

::

    {
        "message": "Hello World!"
    }


Recipe Blueprint (recipe.py)
============================

The **recipe.py** module defines a Flask blueprint for recipe management.

**Blueprint Creation and Constants:**

::

    import re
    from flask import blueprints, current_app, request, abort, Response, session
    import json

    from main.database import Database
    from main.paths import PROJECT_MAIN

    recipe_bp = blueprints.Blueprint('recipes', __name__)

    ALLOWED_EXTENSIONS = {"png", "jpg", "jpeg", "webp"}
    MAX_FILE_SIZE = 5 * 1024 * 1024

    def allowed_file(filename):
        return "." in filename and filename.rsplit(".", 1)[1].lower() in ALLOWED_EXTENSIONS

**Explanation:**

- The blueprint is named ``'recipes'``. Note the import of ``re`` (regex) is
  present but not used - it may be leftover from earlier development.

- ``ALLOWED_EXTENSIONS`` is a set of file extensions that are accepted for
  image uploads. Using a set makes membership testing faster (O(1) instead of
  O(n) for a list).

- ``MAX_FILE_SIZE`` is set to 5 MB (5 × 1024 × 1024 bytes). Images larger than
  this are rejected.

- ``allowed_file(filename)`` is a helper function that:
  - Checks if the filename contains a dot (``"." in filename``)
  - Splits the filename at the last dot and takes the extension (``rsplit(".", 1)[1]``)
  - Converts the extension to lowercase (``.lower()``)
  - Checks if the extension is in the allowed extensions set

**Route: get_recipes**

::

    @recipe_bp.route('/get-recipes')
    def get_recipes():
        with Database(current_app) as con:
            cur = con.cursor()
            with open(PROJECT_MAIN / "recipe/sql/get_recipes.sql", 'r') as sql_file:
                sql = sql_file.read()
                cur.execute(sql)
            data = cur.fetchall()

        recipes = []
        for row in data:
            recipes.append(dict(row))

        return {"recipes": recipes}

**Explanation:**

This endpoint returns a lightweight list of all recipes (only ID and title).

- No authentication is required - this is a public endpoint.

- The SQL template is read and executed. ``cur.fetchall()`` returns all rows
  from the query.

- Each row is converted to a dictionary using ``dict(row)`` because the
  database connection has ``row_factory = Row`` set.

- The list is wrapped in a ``{"recipes": ...}`` object and returned.

**SQL Template (get_recipes.sql):**

::

    SELECT recipe_id, recipe_title FROM recipe;

This simple query returns only the ID and title. This is efficient for listing
recipes without loading heavy data like images.

**Route: add_recipe**

This is the most complex route. Let's examine it in sections.

**Authentication Check:**

::

    @recipe_bp.route('/add-recipe', methods=['POST'])
    def add_recipe():
        if "user_id" not in session:
            abort(401)

**Explanation:** The route first checks that the user is logged in. If
``user_id`` is not in the session, it returns 401 Unauthorized. This prevents
anonymous users from creating recipes.

**JSON Parsing:**

::

        try:
            recipe_title = request.form.get("recipe-title")
            recipe_time = request.form.get("recipe-time")
            recipe_difficulty = request.form.get("recipe-difficulty")

            recipe_ingredients = json.loads(request.form.get("recipe-ingredients", "[]"))
            recipe_steps = json.loads(request.form.get("recipe-steps", "[]"))
        except (json.JSONDecodeError, TypeError) as error:
            abort(400, f"invalid json in data : {str(error)}")

**Explanation:** The frontend sends the ingredients and steps as JSON strings
embedded in the form data. This is because multipart form data cannot directly
contain nested structures. The code:
- Extracts the plain text fields using ``.get()``
- Parses the JSON strings into Python objects using ``json.loads()``
- Provides a default of ``"[]"`` (empty JSON array) if the fields are missing
- Catches JSON parsing errors and returns a 400 with the error message

**Field Validation:**

::

        if not recipe_title:
            abort(400, "recipe title is required")
        if not recipe_steps:
            abort(400, "recipe steps is required")
        if not recipe_difficulty:
            abort(400, "recipe difficulty is required")
        if not recipe_time:
            abort(400, "recipe time is required")
        if not recipe_ingredients:
            abort(400, "recipe ingredients is required")

**Explanation:** Each required field is checked. The ``if not`` condition
catches both missing fields (None) and empty strings.

**Ingredient Validation:**

::

        for ingredient in recipe_ingredients:
            if "ingredient-amount" not in ingredient:
                abort(400, "ingredient amount is required")
            if not isinstance(ingredient["ingredient-amount"], int):
                abort(400, "ingredient amount must be int")
            if "ingredient-calories" not in ingredient:
                abort(400, "ingredient amount is required")
            if not isinstance(ingredient["ingredient-calories"], int):
                abort(400, "ingredient calories must be int")
            if "ingredient-name" not in ingredient:
                abort(400, "ingredient name is required")
            if "ingredient-unit" not in ingredient:
                abort(400, "ingredient unit is required")

**Explanation:** Each ingredient in the list must have:
- ``ingredient-amount`` (and it must be an integer)
- ``ingredient-calories`` (and it must be an integer)
- ``ingredient-name`` (string)
- ``ingredient-unit`` (string)

The ``isinstance()`` check ensures the value is the correct type. This prevents
string values like "two" from being inserted into an integer column.

**Step Validation:**

::

        for step in recipe_steps:
            if "step-description" not in step:
                abort(400, "step desc is required")
            if "step-duration" not in step:
                abort(400, "step duration is required")
            if "step-index" not in step:
                abort(400, "step index is required")

**Explanation:** Each step must have description, duration, and index. No type
checking is done on these values (they are stored as TEXT).

**Main Image Handling:**

::

        main_image_blob = None
        if "recipe-main-image" in request.files:
            file = request.files["recipe-main-image"]
            if file and file.filename and allowed_file(file.filename):
                image_data = file.read()
                if len(image_data) > MAX_FILE_SIZE:
                    abort(400, "image too big max is 5mb")
                main_image_blob = image_data

**Explanation:** 
- ``request.files`` contains uploaded files.
- ``if "recipe-main-image" in request.files`` checks if a main image was uploaded.
- The file is validated: it must exist, have a filename, and the extension
  must be allowed.
- ``file.read()`` reads the entire file into memory as bytes.
- The size is checked against the 5 MB limit.
- The binary data is stored in ``main_image_blob`` (will be ``None`` if no
  image was uploaded).

**Step Images Handling:**

::

        step_images = {}
        for key in request.files:
            if key.startswith("step-image-"):
                step_index = key.replace("step-image-", "")
                file = request.files[key]
                if file and file.filename and allowed_file(file.filename):
                    image_data = file.read()
                    if len(image_data) > MAX_FILE_SIZE:
                        abort(400, f"step image {step_index} file too big 5mb max")
                    step_images[step_index] = image_data

**Explanation:** Step images are sent with keys like ``step-image-0``,
``step-image-1``, etc. The code:
- Iterates through all uploaded files
- Checks if the key starts with ``"step-image-"``
- Extracts the index (the number after the prefix)
- Validates the file (same as main image)
- Stores the binary data in a dictionary keyed by the index for easy lookup later

**Recipe Insertion:**

::

        with Database(current_app) as con:
            cur = con.cursor()
            with open(PROJECT_MAIN / "recipe/sql/add_recipe.sql", 'r') as sql_file:
                sql = sql_file.read()
                cur.execute(sql, (recipe_title, recipe_time, recipe_difficulty, main_image_blob))

            new_recipe_id = cur.lastrowid

**Explanation:** The recipe is inserted first because we need its ID to link
ingredients and steps. 
- ``cur.lastrowid`` retrieves the auto-incremented recipe ID.

**Ingredient Insertion:**

::

            for ingredient in recipe_ingredients:
                ingredient_amount = ingredient["ingredient-amount"]
                ingredient_calories = ingredient["ingredient-calories"]
                ingredient_name = ingredient["ingredient-name"]
                ingredient_unit = ingredient["ingredient-unit"]

                with open(PROJECT_MAIN / "recipe/sql/add_ingredient.sql", 'r') as sql_file:
                    sql = sql_file.read()
                    cur.execute(sql, (new_recipe_id, ingredient_name, ingredient_amount,
                                      ingredient_unit, ingredient_calories))

**Explanation:** Each ingredient is inserted separately. The SQL template is
read inside the loop (inefficient but acceptable for small numbers of
ingredients). All ingredients are linked to the new recipe by its ID.

**Step Insertion:**

::

            for step in recipe_steps:
                step_description = step["step-description"]
                step_duration = step["step-duration"]
                step_index = step["step-index"]
                step_image_blob = step_images.get(step_index, None)

                with open(PROJECT_MAIN / "recipe/sql/add_step.sql", 'r') as sql_file:
                    sql = sql_file.read()
                    cur.execute(sql, (new_recipe_id, step_index, step_description,
                                      step_duration, step_image_blob))
            con.commit()

**Explanation:** Each step is inserted. 
- ``step_images.get(step_index, None)`` looks up the image binary data using
  the step index. If no image was uploaded for this step, it uses ``None``.
- After all inserts (recipe, ingredients, steps), the transaction is committed
  with ``con.commit()``. This ensures all inserts succeed or all fail together
  (atomicity).

**Response:**

::

        return {"recipe-id": new_recipe_id}, 201

Returns the new recipe's ID with status 201 Created.

**SQL Templates:**

``add_recipe.sql``:

::

    INSERT INTO recipe (recipe_title, recipe_time, recipe_difficulty, recipe_main_image)
    VALUES (?, ?, ?, ?);

``add_ingredient.sql``:

::

    INSERT INTO recipe_ingredient (
        recipe_id, recipe_ingredient_name,
        recipe_ingredient_amount, recipe_ingredient_unit,
        recipe_ingredient_calories
    )
    VALUES (?, ?, ?, ?, ?)

``add_step.sql``:

::

    INSERT INTO recipe_step (recipe_id, recipe_step_index,
                             recipe_step_description, recipe_step_duration, recipe_step_image
    )
    VALUES (?, ?, ?, ?, ?)

**Route: get_recipe_image**

::

    @recipe_bp.route("/recipe-image/<int:recipe_id>")
    def get_recipe_image(recipe_id):
        with Database(current_app) as con:
            cur = con.cursor()
            with open(PROJECT_MAIN / "recipe/sql/get_main_image.sql", 'r') as sql_file:
                sql = sql_file.read()
                cur.execute(sql, (recipe_id,))
                result = cur.fetchone()

                if result is None or result["recipe_main_image"] is None:
                    abort(404, "image not found")

                response = Response(result["recipe_main_image"], mimetype="image/jpeg")
                response.headers["Cache-Control"] = "public, max-age=31536000"
                return response

**Explanation:** This endpoint serves recipe images.

- The URL contains a parameter ``<int:recipe_id>`` which Flask converts to an
  integer and passes to the function.

- The SQL query fetches the ``recipe_main_image`` BLOB for the given recipe ID.

- If no recipe exists with that ID (``result is None``) or the image column is
  NULL (``result["recipe_main_image"] is None``), return 404.

- A Flask ``Response`` object is created with the binary image data and the
  MIME type ``image/jpeg``.

- The ``Cache-Control`` header is set to ``public, max-age=31536000`` (one year).
  This tells browsers and CDNs to cache the image aggressively because recipe
  images rarely change.

**SQL Template (get_main_image.sql):**

::

    SELECT recipe_main_image
    FROM recipe
    WHERE recipe_id = ?;

**Route: get_step_image**

::

    @recipe_bp.route("/step-image/<int:step_id>")
    def get_step_image(step_id):
        with Database(current_app) as con:
            cur = con.cursor()
            with open(PROJECT_MAIN / "recipe/sql/get_step_image.sql", 'r') as sql_file:
                sql = sql_file.read()
                cur.execute(sql, (step_id,))
                result = cur.fetchone()

                if result is None or result["recipe_step_image"] is None:
                    abort(404, "image not found")

                response = Response(result["recipe_step_image"], mimetype="image/jpeg")
                response.headers["Cache-Control"] = "public, max-age=31536000"
                return response

**Explanation:** Identical to ``get_recipe_image`` but uses a different SQL
template and parameter (step_id instead of recipe_id).

**SQL Template (get_step_image.sql):**

::

    SELECT recipe_step_image
    FROM recipe_step
    WHERE recipe_step_id = ?;


Search Recipe Blueprint (search_recipe.py)
==========================================

The **search_recipe.py** module defines a Flask blueprint for searching recipes
by title.

**SQL Pre-loading and Blueprint Creation:**

::

    from flask import blueprints, current_app, request, abort, session

    from main.database import Database
    from main.paths import PROJECT_MAIN

    SEARCH_SQL = (PROJECT_MAIN / "search_recipe/sql/search_recipe.sql").read_text()
    INGREDIENT_SQL = (PROJECT_MAIN / "search_recipe/sql/recipe_ingredients.sql").read_text()

    search_bp = blueprints.Blueprint('search', __name__)

**Explanation:** 

- Instead of reading SQL templates on every request, this module reads them
  once at import time using ``.read_text()``. This improves performance by
  avoiding repeated file I/O.

- ``.read_text()`` reads the entire file and returns it as a string.

**Route: search_recipe**

::

    @search_bp.route('/search-recipe', methods=['GET'])
    def search_recipe():
        if "user_id" not in session:
            abort(401)

**Explanation:** Authentication is required - only logged-in users can search.

**Query Parameter Extraction:**

::

        query = request.args.get('q', '').strip()

        if not query:
            abort(400)

**Explanation:** 
- ``request.args`` contains URL query parameters (e.g., ``?q=pancakes``).
- ``.get('q', '')`` retrieves the 'q' parameter, defaulting to empty string.
- ``.strip()`` removes leading and trailing whitespace.
- If the result is empty (including if the parameter was missing or only spaces),
  return 400 Bad Request.

**Search Query Execution:**

::

        with Database(current_app) as con:
            cur = con.cursor()
            cur.execute(SEARCH_SQL, (f"%{query}%",))

            recipe_rows = cur.fetchall()
            recipes = []

**Explanation:** 
- The search SQL uses ``LIKE`` with wildcards. ``f"%{query}%"`` wraps the user's
  query with ``%`` on both sides, so searching for "cake" matches "Chocolate Cake"
  and "Cake Pop".
- ``cur.fetchall()`` returns all matching recipe rows.

**Ingredient Fetching (N+1 Query Pattern):**

::

            for recipe in recipe_rows:
                recipe_dict = dict(recipe)
                recipe_id = recipe["recipe_id"]

                cur.execute(INGREDIENT_SQL, (recipe_id,))
                ingredient_rows = cur.fetchall()
                ingredients = []

                for ingredient in ingredient_rows:
                    ingredients.append(dict(ingredient))

                recipe_dict["ingredients"] = ingredients
                recipes.append(recipe_dict)

**Explanation:** 

This is the N+1 query pattern:
- One query fetches N matching recipes
- For each recipe (N times), another query fetches its ingredients

This is acceptable for a small number of recipes but would be inefficient for
large datasets. A production solution might use a JOIN or batch loading.

**Response:**

::

        return {"recipes": recipes}

Returns all recipes with their ingredients.

**SQL Templates:**

``search_recipe.sql``:

::

    SELECT
        recipe_id,
        recipe_title,
        recipe_time,
        recipe_difficulty
    FROM recipe
    WHERE recipe_title LIKE ?;

``recipe_ingredients.sql``:

::

    SELECT
        recipe_ingredient_name
    FROM recipe_ingredient
    WHERE recipe_id = ?;


View Recipe Blueprint (view_recipe.py)
======================================

The **view_recipe.py** module defines a Flask blueprint for retrieving recipe
details and tracking step completion.

**Blueprint Creation:**

::

    from flask import blueprints, current_app, request, abort, session

    from main.database import Database
    from main.paths import PROJECT_MAIN

    view_recipe_bp = blueprints.Blueprint('view-recipe', __name__)

**Route: view_recipe**

::

    @view_recipe_bp.route('/view-recipe/<int:recipe_id>', methods=['GET'])
    def view_recipe(recipe_id):
        if "user_id" not in session:
            abort(401)

**Explanation:** Authentication is required. The recipe ID comes from the URL.

**Recipe Query:**

::

        user_id = request.args.get("user_id")
        with Database(current_app) as con:
            cur = con.cursor()

            with open(PROJECT_MAIN / "view_recipe/sql/get_recipe.sql") as sql_file:
                sql = sql_file.read()
                cur.execute(sql, (recipe_id,))
                result = cur.fetchone()

            if result is None:
                abort(404, "recipe not found")

            recipe = dict(result)

**Explanation:**
- The user_id is extracted from query parameters (e.g., ``?user_id=123``).
- The recipe is fetched using the ID. If not found, return 404.

**Ingredients Query:**

::

            with open(PROJECT_MAIN / "view_recipe/sql/get_recipe_ingredients.sql") as sql_file:
                sql = sql_file.read()
                cur.execute(sql, (recipe_id,))
                ingredient_data = cur.fetchall()

            ingredients = []
            for row in ingredient_data:
                ingredients.append(dict(row))

**Explanation:** Fetches all ingredients for the recipe and converts them to
dictionaries.

**Steps Query with Completion Status:**

::

            with open(PROJECT_MAIN / "view_recipe/sql/get_recipe_steps.sql") as sql_file:
                sql = sql_file.read()
                cur.execute(sql, (recipe_id, user_id))
                step_data = cur.fetchall()

            steps = []
            for row in step_data:
                row_data = dict(row)
                row_data["step-completion"] = bool(row_data["step-completion"])
                steps.append(row_data)

**Explanation:** 
- The SQL query joins the steps table with the completion table to determine
  which steps the current user has completed.
- The query returns 0 or 1 for ``step-completion``. This is converted to a
  Python boolean (``True``/``False``) using ``bool()``.

**Response Assembly:**

::

            recipe["recipe-ingredients"] = ingredients
            recipe["recipe-steps"] = steps

            return recipe

**Explanation:** The ingredients and steps are attached to the recipe
dictionary and returned as JSON.

**SQL Templates:**

``get_recipe.sql``:

::

    SELECT
        r.recipe_id AS "recipe-id",
        r.recipe_title AS "recipe-title",
        r.recipe_difficulty AS "recipe-difficulty",
        r.recipe_time AS "recipe-time"
    FROM
        recipe r
    WHERE
        r.recipe_id = ?

Notice the ``AS`` aliases - they rename the columns to match the frontend's
expected JSON keys (with hyphens instead of underscores).

``get_recipe_ingredients.sql``:

::

    SELECT
        recipe_ingredient_name AS "ingredient-name",
        recipe_ingredient_amount AS "ingredient-amount",
        recipe_ingredient_unit AS "ingredient-unit",
        recipe_ingredient_calories as "ingredient-calories"
    FROM
        recipe_ingredient
    WHERE
        recipe_id = ?

``get_recipe_steps.sql``:

::

    SELECT
        rs.recipe_step_id AS "recipe-step-id",
        rs.recipe_step_description AS "step-description",
        rs.recipe_step_duration AS "step-duration",
        rs.recipe_step_index as "step-index",
        CASE
            WHEN EXISTS
                (
                    SELECT 1
                    FROM
                        recipe_step_completion rsc
                    WHERE
                        rsc.recipe_step_id = rs.recipe_step_id AND rsc.user_id = ?2
                )
                THEN 1
                ELSE 0
            END AS "step-completion"
    FROM
        recipe_step rs
    WHERE
        rs.recipe_id = ?1;

This is the most complex query. The ``CASE`` statement checks if a completion
record exists for this step and the given user. If it exists, it returns 1
(True), otherwise 0 (False).

**Route: complete_step**

::

    @view_recipe_bp.route('/complete-step', methods=["POST"])
    def complete_step():
        if "user_id" not in session:
            abort(401)

        data = request.get_json()
        if data is None:
            abort(400, "invalid or missing JSON body")

        recipe_step_id = data.get("recipe_step_id")
        user_id = data.get("user_id")

        if recipe_step_id is None or user_id is None:
            abort(400)

        with Database(current_app) as con:
            cur = con.cursor()
            with open(PROJECT_MAIN / "view_recipe/sql/complete_step.sql", 'r') as sql_file:
                sql = sql_file.read()
                cur.execute(sql, (recipe_step_id, user_id))
                con.commit()

        return {"success": True}, 200

**Explanation:** 

- The request must contain JSON with ``recipe_step_id`` and ``user_id``.
- ``request.get_json()`` parses the JSON body. If it's invalid, returns None.
- The SQL uses ``INSERT OR IGNORE`` - if the completion already exists, nothing
  happens. This makes the endpoint idempotent (calling it multiple times has
  the same effect as calling it once).

**SQL Template (complete_step.sql):**

::

    INSERT OR IGNORE INTO recipe_step_completion (recipe_step_id, user_id)
    VALUES (?, ?)

**Route: uncomplete_step**

::

    @view_recipe_bp.route('/uncomplete-step', methods=["POST"])
    def uncomplete_step():
        if "user_id" not in session:
            abort(401)

        data = request.get_json()
        if data is None:
            abort(400, "invalid or missing JSON body")

        recipe_step_id = data.get("recipe_step_id")
        user_id = data.get("user_id")

        if recipe_step_id is None or user_id is None:
            abort(400)

        with Database(current_app) as con:
            cur = con.cursor()
            with open(PROJECT_MAIN / "view_recipe/sql/uncomplete_step.sql", 'r') as sql_file:
                sql = sql_file.read()
                cur.execute(sql, (recipe_step_id, user_id))
                con.commit()

        return {"success": True}, 200

**Explanation:** Similar to ``complete_step`` but uses DELETE. If no matching
row exists, the DELETE does nothing (also idempotent).

**SQL Template (uncomplete_step.sql):**

::

    DELETE FROM recipe_step_completion
    WHERE recipe_step_id = ? AND user_id = ?


Requirements (requirements.txt)
===============================

The **requirements.txt** file lists all Python package dependencies.

::

    Flask >= 3.1.2
    pytest >= 9.0.2
    bcrypt >= 5.0.0

**Explanation of each dependency:**

**Flask >= 3.1.2**

Flask is the core web framework. It provides:

- Routing and request handling (mapping URLs to Python functions)
- Request/response abstraction (accessing form data, JSON, query parameters)
- Session management (signed cookies for user state)
- Development server for testing

The minimum version 3.1.2 ensures modern features and security patches.

**pytest >= 9.0.2**

pytest is the testing framework used to write and run unit tests. It provides:

- Fixture system for test setup and teardown (e.g., creating a test database)
- Assert introspection (detailed error messages when assertions fail)
- Test discovery and collection (automatically finds test files)
- Plugin ecosystem (coverage reporting, mocking)

**bcrypt >= 5.0.0**

bcrypt is a password hashing library. While the current implementation stores
passwords in plain text, this dependency suggests future plans to implement
proper password hashing. bcrypt provides:

- Salted password hashing (each password gets a random salt, making rainbow
  table attacks infeasible)
- Configurable work factor (can be increased as hardware improves to slow down
  brute force attacks)
- Constant-time comparison (prevents timing attacks that could leak password
  information)

**Installation:**

To install all dependencies in a virtual environment:

::

    pip install -r requirements.txt

Or with pinned versions (more reproducible builds):

::

    Flask==3.1.2
    pytest==9.0.2
    bcrypt==5.0.0

**Security Note on Plain Text Passwords:**

The current implementation stores passwords as plain text in the ``user``
table. This is **not secure** for production use. The ``bcrypt`` dependency is
included but not yet used. To implement proper password handling:

1. Hash passwords during account creation:
   ``hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt())``

2. Store the hash in the database instead of the plain password

3. During login, compare:
   ``bcrypt.checkpw(password.encode(), stored_hash)``

The plain text approach is acceptable only for this university project where
simplicity and ease of debugging outweigh security concerns.
