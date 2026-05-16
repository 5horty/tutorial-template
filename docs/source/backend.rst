Account Blueprint (account.py)
==============================

The **account.py** module defines a Flask blueprint for user account operations.
It centralises account-related endpoints, keeping route logic separate from the main application factory and making the codebase easier to maintain.

Overview
--------

This module creates a ``Blueprint`` named ``account_bp``. It currently exposes a single ``POST`` route ``/create-account`` that:

* receives form data containing the new user's first name, last name, email and password
* validates that all required fields are present
* inserts a new row into the ``user`` table using a parameterised SQL template
* returns the ID of the newly created user

The route does **not** hash the password. This is a deliberate simplification for the university project. A ``todo`` comment in the code reminds us that dietary preferences should be added in a future iteration.

Blueprint: account_bp
---------------------

**Type:** ``flask.Blueprint``

**Properties:**

* ``name`` - ``account``, the internal name used by Flask for URL generation (e.g. ``url_for('account.create_account')``)
* ``import_name`` - ``__name__``, used so Flask can locate module resources relative to this file

The blueprint is registered on the Flask app in the application factory (not shown here), making all routes available under the configured prefix.

Route: create_account
---------------------

**Method:** ``POST``
**Path:** ``/create-account``
**Decorator:** ``@account_bp.route('/create-account', methods=['POST'])``

**Request:** Expects ``application/x-www-form-urlencoded`` form data. Required fields:

* ``user_fname`` - first name
* ``user_lname`` - last name
* ``user_email`` - email address
* ``user_password`` - password

Validation
----------

1. If ``request.form`` is empty, the request is rejected with ``400 Bad Request``
2. Each field is checked individually
3. If any field is missing or empty, the request is rejected with ``400 Bad Request``

No detailed error messages are returned.

Processing Flow
---------------

1. Extract values from ``request.form``
2. Open a database connection using the ``Database`` context manager
3. Load SQL from ``account/sql/create_user.sql``
4. Execute parameterised insert query
5. Commit transaction
6. Retrieve new user id using ``cur.lastrowid``
7. Return JSON response

Code uses parameterised queries (``?`` placeholders) to prevent SQL injection.

Response
--------

Successful response:

.. code-block:: json

    {
        "user_id": 1
    }

Status code: ``201 Created``

Failure response:
* ``400 Bad Request`` with empty body

Side Effects
------------

A new row is inserted into the ``user`` table.

Passwords are stored in plain text (not secure, but acceptable for this academic project).

Code
----

.. code-block:: python

    from flask import blueprints, current_app, request, abort

    from main.database import Database
    from main.paths import PROJECT_MAIN

    account_bp = blueprints.Blueprint('account', __name__)


    @account_bp.route('/create-account', methods=['POST'])
    def create_account():
        # todo: include dietary preferences

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

        user_fname = request.form['user_fname']
        user_lname = request.form['user_lname']
        user_password = request.form['user_password']
        user_email = request.form['user_email']

        with Database(current_app) as con:
            cur = con.cursor()

            with open(PROJECT_MAIN / "account/sql/create_user.sql", "r") as sql_file:
                sql = sql_file.read()

            cur.execute(sql, (user_fname, user_lname, user_email, user_password))
            con.commit()

            user_id = cur.lastrowid

        return {"user_id": user_id}, 201


SQL Template
------------

The route uses the following parameterised SQL file:

.. code-block:: sql

    INSERT INTO user (user_fname, user_lname, user_email, user_password)
    VALUES (?, ?, ?, ?);

Authentication Blueprint (authentication.py)
============================================

The **authentication.py** module defines a Flask blueprint for user login and
session management. It verifies credentials against the database and establishes
a server-side session that other routes rely on for authorisation.

Overview
--------

This module creates a ``Blueprint`` named ``authentication_bp``. It exposes a
``POST`` endpoint ``/authenticate`` that:

* receives form data containing the user's first name, last name and password,
* looks up the user by first and last name using a case-insensitive ``LIKE``
  comparison,
* compares the supplied password against the stored value,
* returns user data (excluding the password) and sets ``session["user_id"]``
  on success.

The session is then used by other blueprints (e.g. ``add-recipe``,
``search-recipe``, ``view-recipe``) to identify the logged-in user.

.. note::

   Passwords are stored and compared as plain text. This is acceptable only for
   a university project and should be replaced with proper hashing (e.g.
   ``bcrypt``) in any production system.

Blueprint: authentication_bp
----------------------------

**Type:** ``flask.Blueprint``

**Properties:**

- ``name`` - ``authentication``, the internal name used by Flask for URL
  generation (e.g. ``url_for('authentication.authenticate')``)
- ``import_name`` - ``__name__``, passed to the blueprint constructor so Flask
  can locate resources relative to this module

Route: authenticate
-------------------

**Method:** ``POST``
**Path:** ``/authenticate``
**Decorator:** ``@authentication_bp.route('/authenticate', methods=['POST'])``

**Request:** Expects ``application/x-www-form-urlencoded`` form data. The
following fields are **required**:

* ``user_fname`` - first name (string)
* ``user_lname`` - last name (string)
* ``user_password`` - password (string)

**Validation:**

1. The function checks for the presence of all three keys directly in
   ``request.form`` using ``in``. If any key is missing, it aborts with
   ``400 Bad Request``.
2. Unlike the account blueprint, empty strings are **not** rejected here;
   only the absence of the key is checked. However, an empty name would
   practically result in a ``401`` because no user would match.

**Processing:**

1. The values are extracted from ``request.form``.
2. A database connection is opened with the ``Database`` context manager, which
   uses ``current_app`` to obtain configuration details.
3. The SQL template ``authentication/sql/get_user.sql`` is read from disk. The
   path is built using ``PROJECT_MAIN``, a ``pathlib.Path`` pointing to the
   project root.
4. The ``SELECT`` query is executed with ``(user_fname, user_lname)`` as
   parameters. Because the SQL uses ``LIKE`` without wildcards, the comparison
   is effectively a case-insensitive equality check (assuming the database uses
   default ``NOCASE`` or similar collation). The ``LIMIT 1`` ensures at most one
   row is returned.
5. If ``cur.fetchone()`` returns ``None``, no user matched the first and last
   name. The function returns ``{"success": False}`` with status
   ``401 Unauthorized`` immediately.
6. The fetched row is converted to a dictionary (``dict(user)``) for easier key
   access. The dictionary contains ``user_id`` and ``user_password`` (aliased
   from the columns in the SQL).
7. The submitted ``user_password`` is compared to ``user["user_password"]``.
   If they do not match, the function returns ``{"success": False}``, ``401``.
8. On success, the password is removed from the dictionary (``del
   user["user_password"]``) so it never reaches the client.
9. ``session["user_id"]`` is set to the user's ID. Flask sends a signed session
   cookie to the client, allowing the server to recognise the user on
   subsequent requests.
10. The function returns ``{"success": True, "user": user}`` with an implicit
    ``200 OK`` status.

**Response:**

- **401 Unauthorized** - ``{"success": false}`` when the user is not found or
  the password does not match. The same generic response is used in both cases
  to avoid revealing which piece of information was incorrect.
- **200 OK** - ``{"success": true, "user": {...}}`` where the ``user`` object
  contains the ``user_id`` and any other fields returned by the query (except
  the password). For the current SQL, only ``user_id`` is included; if more
  columns are added they will appear here.

**Side Effect:** Sets ``session["user_id"]``. All subsequent requests from the
same client can use this value for authentication checks.

Code
----

.. code-block:: python

    from flask import blueprints, current_app, request, abort, session

    from main.database import Database
    from main.paths import PROJECT_MAIN

    authentication_bp = blueprints.Blueprint('authentication', __name__)


    @authentication_bp.route('/authenticate', methods=['POST'])
    def authenticate():
        # Validate form first
        if "user_fname" not in request.form:
            abort(400)

        if "user_lname" not in request.form:
            abort(400)

        if "user_password" not in request.form:
            abort(400)

        # Use first and last names to get the user ID from the database
        user_fname = request.form['user_fname']
        user_lname = request.form['user_lname']
        user_password = request.form['user_password']

        with Database(current_app) as con:
            cur = con.cursor()

            with open(PROJECT_MAIN / "authentication/sql/get_user.sql", "r") as sql_file:
                sql = sql_file.read()

            cur.execute(sql, (user_fname, user_lname))
            user = cur.fetchone()

            if user is None:
                # No user exists or information wrong so return not authenticated
                return {"success": False}, 401

            user = dict(user)

            if user["user_password"] != user_password:
                return {"success": False}, 401

            del user["user_password"]

        session["user_id"] = user["user_id"]

        return {"success": True, "user": user}


SQL Template
------------

The route loads and executes the following parameterised SQL file
(``authentication/sql/get_user.sql``):

.. code-block:: sql

    SELECT
        user_id, user_password
    FROM
        user
    WHERE
        -- Comparison is case-insensitive!
        user_fname LIKE ? AND user_lname LIKE ?
    LIMIT 1;

Recipe Blueprint (recipe.py)
============================

The **recipe.py** module defines a Flask blueprint for recipe management.
It handles listing all recipes, creating new recipes with their associated
ingredients, steps, and images, and serving recipe and step images.

Overview
--------

This module creates a ``Blueprint`` named ``recipe_bp``. It provides four
endpoints:

* ``GET /get-recipes`` – returns a lightweight list of all recipes (ID and title only).
* ``POST /add-recipe`` – creates a full recipe, validates ingredients and steps,
  optionally stores uploaded images for the main recipe and individual steps.
* ``GET /recipe-image/<id>`` – serves the main image blob of a recipe as a
  JPEG with long-term caching.
* ``GET /step-image/<id>`` – serves the image blob of a specific recipe step as
  a JPEG with the same caching policy.

File uploads are validated for allowed extensions (``png``, ``jpg``, ``jpeg``,
``webp``) and a maximum size of 5 MB. The ``add-recipe`` route requires an
active user session.  Images are stored as binary blobs in the database,
simplifying deployment and avoiding a separate file-storage service.

.. note::

   The ``re`` module is imported at the top of the file but is not currently
   used. It may be a leftover from earlier experimentation or intended for
   future input validation.

Blueprint: recipe_bp
--------------------

**Type:** ``flask.Blueprint``

**Properties:**

- ``name`` – ``'recipes'``, the internal name for URL generation (e.g.
  ``url_for('recipes.get_recipes')``).
- ``import_name`` – ``__name__``, passed to the blueprint constructor.

The blueprint is registered on the Flask app with a prefix (not shown here),
which makes all its routes available under that prefix (e.g. ``/api/recipes``).

Helpers
-------

**allowed_file(filename)**

Returns ``True`` if the ``filename`` contains a dot and the characters after the
last dot (lowercased) are in the ``ALLOWED_EXTENSIONS`` set. This prevents
arbitrary file types from being stored.

**Constants**

- ``ALLOWED_EXTENSIONS`` = ``{"png", "jpg", "jpeg", "webp"}`` – only these image
  formats are accepted for upload.
- ``MAX_FILE_SIZE`` = 5 * 1024 * 1024 (5 MB) – the largest acceptable image
  file. Uploads exceeding this size are rejected with a ``400`` error.

Route: get_recipes
------------------

**Method:** ``GET``
**Path:** ``/get-recipes``
**Decorator:** ``@recipe_bp.route('/get-recipes')``

**Authentication:** None required. This endpoint is public so unauthenticated
users can browse recipe titles.

**Processing:**

1. Opens a database connection via the ``Database`` context manager, which
   reads configuration from ``current_app``.
2. Loads the SQL template ``recipe/sql/get_recipes.sql`` – a simple ``SELECT
   recipe_id, recipe_title FROM recipe``.
3. Executes the query and fetches all rows.
4. Iterates over each row and converts it to a dictionary using ``dict(row)``.
   The resulting list is wrapped in ``{"recipes": ...}`` and returned.

**Response:** JSON object containing the key ``recipes``, which is a list of
objects each with ``recipe_id`` and ``recipe_title``.

.. code-block:: json

    {
      "recipes": [
        { "recipe_id": 1, "recipe_title": "Pancakes" }
      ]
    }

Route: add_recipe
-----------------

**Method:** ``POST``
**Path:** ``/add-recipe``
**Decorator:** ``@recipe_bp.route('/add-recipe', methods=['POST'])``

**Authentication:** Requires ``user_id`` in the session. If the user is not
logged in, the server returns ``401 Unauthorized``. This prevents anonymous
users from creating recipes.

**Request:** The endpoint expects ``multipart/form-data`` (not
``application/x-www-form-urlencoded`` or JSON) because it must support file
uploads alongside text fields. The following form fields are **required**:

* ``recipe-title`` (string) – the title of the recipe.
* ``recipe-time`` (string) – preparation/cooking time (format free-text, e.g.
  "30 min").
* ``recipe-difficulty`` (string) – difficulty level (e.g. "Easy", "Medium",
  "Hard").
* ``recipe-ingredients`` (string) – a **JSON-encoded** array of ingredient
  objects.
* ``recipe-steps`` (string) – a **JSON-encoded** array of step objects.

**Optional File Fields:**

* ``recipe-main-image`` – the main image representing the finished dish.
* ``step-image-{index}`` – an image for a particular step. The key prefix
  ``step-image-`` is stripped to obtain the step's ``step-index`` (e.g.
  ``step-image-0`` relates to the step with ``step-index`` 0). Multiple such
  keys can be present.

**Validation (high-level):**

1. The user session is checked first.
2. The JSON strings for ingredients and steps are parsed. If either is invalid
   (not valid JSON) or not of the expected type, a ``400`` response with an
   explanatory error message is returned.
3. Every top-level field (title, steps, difficulty, time, ingredients) is
   checked for presence and non-emptiness.
4. Each **ingredient** in the ``recipe-ingredients`` array is checked for:
   * ``ingredient-amount`` (must be present and an ``int``)
   * ``ingredient-calories`` (must be present and an ``int``)
   * ``ingredient-name`` (must be present)
   * ``ingredient-unit`` (must be present)
5. Each **step** in the ``recipe-steps`` array is checked for:
   * ``step-description``
   * ``step-duration``
   * ``step-index``

If any sub-field is missing or the wrong type, a ``400`` response with a
specific error string is returned.

**Image Validation:**

- For the main image and every step image, the function checks:
  * That the filename exists and the extension is allowed (via ``allowed_file``).
  * That the file size does not exceed ``MAX_FILE_SIZE`` (5 MB).
- If an image fails validation, the request is rejected with a ``400`` status
  and a message indicating which image (or step image) caused the problem.

**Processing (after validation):**

1. A database connection is opened.
2. The main recipe is inserted using ``recipe/sql/add_recipe.sql`` with the
   parameters ``(title, time, difficulty, main_image_blob)``. If no main image
   was uploaded, ``main_image_blob`` is ``None``.
3. ``cur.lastrowid`` gives the ``new_recipe_id``, which is used to link
   ingredients and steps.
4. For each ingredient, ``recipe/sql/add_ingredient.sql`` is executed with
   the ingredient fields and the new recipe ID.
5. For each step, the function looks up any uploaded image in the
   ``step_images`` dictionary (built earlier) using the step's ``step-index`` as
   the key. If found, the binary data is passed; otherwise ``None`` is used.
   The ``recipe/sql/add_step.sql`` template is executed with the recipe ID,
   step index, description, duration, and image blob.
6. The transaction is committed with ``con.commit()``, ensuring all inserts
   succeed or fail together.

**Response:**

- **201 Created** – JSON body containing the ``recipe-id`` of the newly created
  recipe:
  
  .. code-block:: json
  
      { "recipe-id": 42 }
  
- **400 Bad Request** – JSON string with a description of the validation error
  (e.g. ``"invalid json in data : ..."``, ``"recipe title is required"``,
  ``"ingredient amount must be int"``, ``"image too big max is 5mb"``).
- **401 Unauthorized** – when ``user_id`` is not in the session.

**Side Effects:** Inserts one row into ``recipe``, multiple rows into
``recipe_ingredient``, and multiple rows into ``recipe_step``. If images were
uploaded, their binary content is stored in the ``recipe_main_image`` and
``recipe_step_image`` columns.

Route: get_recipe_image
-----------------------

**Method:** ``GET``
**Path:** ``/recipe-image/<int:recipe_id>``
**Decorator:** ``@recipe_bp.route("/recipe-image/<int:recipe_id>")``

**Authentication:** None required. The images are publicly accessible.

**Processing:**

1. Opens a database connection.
2. Loads the SQL template ``recipe/sql/get_main_image.sql``, which fetches
   ``recipe_main_image`` for the given ``recipe_id``.
3. Executes the query. If the result is ``None`` or the returned column is
   ``NULL``, a ``404 Not Found`` is returned with the message ``"image not
   found"``.
4. Otherwise, the binary data is wrapped in a Flask ``Response`` with the
   MIME type ``image/jpeg``. The ``Cache-Control`` header is set to
   ``public, max-age=31536000`` (one year) because recipe images are static
   and can be cached aggressively by browsers and proxies.

**Response:** Binary JPEG data (status ``200``) or ``404``.

Route: get_step_image
---------------------

**Method:** ``GET``
**Path:** ``/step-image/<int:step_id>``
**Decorator:** ``@recipe_bp.route("/step-image/<int:step_id>")``

**Authentication:** None required.

**Processing:**

1. Opens a database connection.
2. Loads ``recipe/sql/get_step_image.sql``, which retrieves
   ``recipe_step_image`` for the given ``step_id``.
3. If the image is missing, returns ``404``.
4. Otherwise returns the blob as ``image/jpeg`` with the same one-year caching
   headers.

**Response:** Binary JPEG data or ``404``.

Code
----

.. code-block:: python

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


    @recipe_bp.route('/add-recipe', methods=['POST'])
    def add_recipe():
        if "user_id" not in session:
            abort(401)

        try:
            recipe_title = request.form.get("recipe-title")
            recipe_time = request.form.get("recipe-time")
            recipe_difficulty = request.form.get("recipe-difficulty")

            recipe_ingredients = json.loads(request.form.get("recipe-ingredients", "[]"))
            recipe_steps = json.loads(request.form.get("recipe-steps", "[]"))
        except (json.JSONDecodeError, TypeError) as error:
            abort(400, f"invalid json in data : {str(error)}")

        # validate data
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

        # validate ingredients
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

        # validate steps
        for step in recipe_steps:
            if "step-description" not in step:
                abort(400, "step desc is required")
            if "step-duration" not in step:
                abort(400, "step duration is required")
            if "step-index" not in step:
                abort(400, "step index is required")

        # handle main image
        main_image_blob = None
        if "recipe-main-image" in request.files:
            file = request.files["recipe-main-image"]
            if file and file.filename and allowed_file(file.filename):
                image_data = file.read()
                if len(image_data) > MAX_FILE_SIZE:
                    abort(400, "image too big max is 5mb")
                main_image_blob = image_data

        # step images
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

        with Database(current_app) as con:
            cur = con.cursor()
            with open(PROJECT_MAIN / "recipe/sql/add_recipe.sql", 'r') as sql_file:
                sql = sql_file.read()
                cur.execute(sql, (recipe_title, recipe_time, recipe_difficulty, main_image_blob))

            new_recipe_id = cur.lastrowid

            # insert ingredients
            for ingredient in recipe_ingredients:
                ingredient_amount = ingredient["ingredient-amount"]
                ingredient_calories = ingredient["ingredient-calories"]
                ingredient_name = ingredient["ingredient-name"]
                ingredient_unit = ingredient["ingredient-unit"]

                with open(PROJECT_MAIN / "recipe/sql/add_ingredient.sql", 'r') as sql_file:
                    sql = sql_file.read()
                    cur.execute(sql, (new_recipe_id, ingredient_name, ingredient_amount,
                                      ingredient_unit, ingredient_calories))

            # insert steps
            for step in recipe_steps:
                step_description = step["step-description"]
                step_duration = step["step-duration"]
                step_index = step["step-index"]
                step_image_blob = step_images.get(step_index, None)

                with open(PROJECT_MAIN / "recipe/sql/add_step.sql", 'r') as sql_file:
                    sql = sql_file.read()
                    cur.execute(sql, (new_recipe_id, step_index, step_description, step_duration, step_image_blob))
            con.commit()

        return {"recipe-id": new_recipe_id}, 201


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

SQL Templates
-------------

**get_recipes.sql** (``recipe/sql/get_recipes.sql``)

.. code-block:: sql

    SELECT recipe_id, recipe_title FROM recipe;

**add_recipe.sql** (``recipe/sql/add_recipe.sql``)

.. code-block:: sql

    INSERT INTO recipe (recipe_title, recipe_time, recipe_difficulty, recipe_main_image)
    VALUES (?, ?, ?, ?);

**add_ingredient.sql** (``recipe/sql/add_ingredient.sql``)

.. code-block:: sql

    INSERT INTO recipe_ingredient (
        recipe_id, recipe_ingredient_name,
        recipe_ingredient_amount, recipe_ingredient_unit,
        recipe_ingredient_calories
    )
    VALUES (?, ?, ?, ?, ?)

**add_step.sql** (``recipe/sql/add_step.sql``)

.. code-block:: sql

    INSERT INTO recipe_step (recipe_id, recipe_step_index,
                             recipe_step_description, recipe_step_duration, recipe_step_image
    )
    VALUES (?, ?, ?, ?, ?)

**get_main_image.sql** (``recipe/sql/get_main_image.sql``)

.. code-block:: sql

    SELECT recipe_main_image
    FROM recipe
    WHERE recipe_id = ?;

**get_step_image.sql** (``recipe/sql/get_step_image.sql``)

.. code-block:: sql

    SELECT recipe_step_image
    FROM recipe_step
    WHERE recipe_step_id = ?;

Search Recipe Blueprint (search_recipe.py)
==========================================

The **search_recipe.py** module defines a Flask blueprint for searching recipes
by title. It returns matching recipes along with a list of ingredient names for
each, providing enough data for a search results screen without additional
requests.

Overview
--------

This module creates a ``Blueprint`` named ``search_bp``. It exposes a ``GET``
endpoint ``/search-recipe`` that:

* Requires an active user session (authenticated users only).
* Accepts a query string ``q`` and searches the ``recipe`` table using ``LIKE``.
* For each matching recipe, retrieves the associated ingredients from the
  ``recipe_ingredient`` table and attaches them as a nested list.

To minimise file I/O on every request, the SQL templates are read **once** when
the module is imported and stored in the module-level constants ``SEARCH_SQL``
and ``INGREDIENT_SQL``. This avoids opening files repeatedly and improves
response times.

Blueprint: search_bp
--------------------

**Type:** ``flask.Blueprint``

**Properties:**

- ``name`` – ``'search'``, the internal name used by Flask for URL generation
  (e.g. ``url_for('search.search_recipe')``).
- ``import_name`` – ``__name__``, passed to the blueprint constructor so Flask
  can locate resources relative to this module.

The blueprint is registered on the Flask app (typically under a prefix like
``/api``). Its single route enables the frontend to implement a search bar for
recipes.

Route: search_recipe
--------------------

**Method:** ``GET``
**Path:** ``/search-recipe``
**Decorator:** ``@search_bp.route('/search-recipe', methods=['GET'])``

**Authentication:** Requires ``user_id`` in the session. If the user is not
logged in, the server returns ``401 Unauthorized``. This prevents anonymous
users from accessing the search functionality.

**Query Parameters:**

- ``q`` (string, required) – the search term to match against recipe titles.
  Any leading or trailing whitespace is stripped before processing. If the
  string is empty or becomes empty after stripping, a ``400 Bad Request`` is
  returned.

**Error Responses:**

- ``400 Bad Request`` – when the ``q`` parameter is missing or empty after
  stripping. No further error detail is returned.
- ``401 Unauthorized`` – when the session does not contain ``user_id``.

**Processing:**

1. The session is checked for ``user_id``; if absent, abort ``401``.
2. The query string ``q`` is extracted from ``request.args``. Using
   ``.get('q', '')`` provides a default empty string, and ``.strip()`` removes
   surrounding whitespace. If the result is falsy (empty string), the route
   aborts with ``400``.
3. A database connection is opened with the ``Database`` context manager, which
   reads connection settings from ``current_app``.
4. The pre-loaded ``SEARCH_SQL`` is executed with the pattern ``%{query}%``.
   This is a simple substring search using the SQL ``LIKE`` operator. It is not
   a full-text search, so it may be slow on very large datasets, but is
   sufficient for a second-year university project.
5. All matching recipe rows are fetched and the function iterates over them.
   For each recipe:
   * The ``recipe`` row (a database row object) is converted to a dictionary.
   * The recipe's ``recipe_id`` is used to execute ``INGREDIENT_SQL``, which
     fetches all ingredient names for that recipe.
   * Each ingredient row is also converted to a dictionary and appended to a
     list.
   * The ingredient list is attached to the recipe dictionary under the key
     ``"ingredients"``.
6. The enriched recipe list is wrapped in ``{"recipes": recipes}`` and returned
   with an implicit ``200 OK``.

.. note::

   This implementation uses an **N+1 query pattern**: one query to fetch the
   matching recipes, then an additional query per recipe to fetch its
   ingredients. For a project with a small number of recipes this is
   acceptable, but in a production system you would likely use a ``JOIN`` or
   two-step batch load to avoid per-recipe database round-trips.

**Response:** A JSON object containing a ``recipes`` array. Each array element
is a recipe object with the following fields:

- ``recipe_id`` (int) – unique identifier.
- ``recipe_title`` (str) – the recipe's name.
- ``recipe_time`` (str) – preparation/cooking time.
- ``recipe_difficulty`` (str) – difficulty level.
- ``ingredients`` (list) – each item is an object containing
  ``recipe_ingredient_name`` (str).

.. code-block:: json

    {
      "recipes": [
        {
          "recipe_id": 1,
          "recipe_title": "Chocolate Cake",
          "recipe_time": "45 min",
          "recipe_difficulty": "Medium",
          "ingredients": [
            { "recipe_ingredient_name": "Flour" },
            { "recipe_ingredient_name": "Sugar" }
          ]
        }
      ]
    }

Code
----

.. code-block:: python

    from flask import blueprints, current_app, request, abort, session

    from main.database import Database
    from main.paths import PROJECT_MAIN

    SEARCH_SQL = (PROJECT_MAIN / "search_recipe/sql/search_recipe.sql").read_text()
    INGREDIENT_SQL = (PROJECT_MAIN / "search_recipe/sql/recipe_ingredients.sql").read_text()

    search_bp = blueprints.Blueprint('search', __name__)


    @search_bp.route('/search-recipe', methods=['GET'])
    def search_recipe():
        if "user_id" not in session:
            abort(401)
        query = request.args.get('q', '').strip()

        if not query:
            abort(400)

        with Database(current_app) as con:
            cur = con.cursor()
            cur.execute(SEARCH_SQL, (f"%{query}%",))

            recipe_rows = cur.fetchall()
            recipes = []

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

        return {"recipes": recipes}

SQL Templates
-------------

**search_recipe.sql** (``search_recipe/sql/search_recipe.sql``)

.. code-block:: sql

    SELECT
        recipe_id,
        recipe_title,
        recipe_time,
        recipe_difficulty
    FROM recipe
    WHERE recipe_title LIKE ?;

**recipe_ingredients.sql** (``search_recipe/sql/recipe_ingredients.sql``)

.. code-block:: sql

    SELECT
        recipe_ingredient_name
    FROM recipe_ingredient
    WHERE recipe_id = ?;

View Recipe Blueprint (view_recipe.py)
======================================

The **view_recipe.py** module defines a Flask blueprint for retrieving a full
recipe detail, marking steps as completed, and undoing completion. It enriches
a recipe with its ingredients and steps, including per-user completion status,
making it suitable for the recipe detail screen and step-tracking features.

Overview
--------

This module creates a ``Blueprint`` named ``view_recipe_bp``. It provides three
endpoints:

* ``GET /view-recipe/<recipe_id>`` – returns a recipe object with ingredients and
  steps, where each step includes a ``step-completion`` boolean for a given user.
* ``POST /complete-step`` – marks a step as completed for a user (idempotent
  insert).
* ``POST /uncomplete-step`` – removes a step completion record for a user
  (allowing the user to un-check a step).

All routes require an active session (``user_id`` in ``session``). The
step-completion check and mutation rely on a ``user_id`` passed explicitly
in the request parameters or body, which is a design choice that decouples
the session user from the step target (e.g. an admin might later complete
steps for another user, though this is not currently enforced).

Blueprint: view_recipe_bp
-------------------------

**Type:** ``flask.Blueprint``

**Properties:**

- ``name`` – ``'view-recipe'``, the internal name for URL generation (e.g.
  ``url_for('view-recipe.view_recipe', recipe_id=1)``).
- ``import_name`` – ``__name__``, passed to the blueprint constructor.

The blueprint is registered on the Flask app to provide the core interaction
for viewing and tracking recipe progress.

Route: view_recipe
------------------

**Method:** ``GET``
**Path:** ``/view-recipe/<int:recipe_id>``
**Decorator:** ``@view_recipe_bp.route('/view-recipe/<int:recipe_id>', methods=['GET'])``

**Authentication:** Requires ``user_id`` in the session. Returns ``401
Unauthorized`` if missing.

**Query Parameters:**

- ``user_id`` (string, required) – the ID of the user for whom step-completion
  status should be resolved. Although the session already identifies the
  logged-in user, this parameter is required explicitly. This makes the
  interface self-documenting and allows for potential future extension where
  a different user's progress is viewed (e.g. by an instructor).

**Processing:**

1. The session is checked for ``user_id``; if absent, the request is aborted
   with ``401``.
2. A database connection is obtained via the ``Database`` context manager using
   ``current_app``.
3. **Recipe data:** The SQL template
   ``view_recipe/sql/get_recipe.sql`` is loaded and executed with
   ``(recipe_id,)``. This query returns the recipe's core fields (ID, title,
   difficulty, time). If no row is returned, the server aborts with
   ``404 Not Found`` and the message ``"recipe not found"``.
4. **Ingredients:** The template ``view_recipe/sql/get_recipe_ingredients.sql``
   is executed with ``(recipe_id,)``. It fetches all ingredients for the recipe
   with columns aliased to the JSON keys (e.g. ``ingredient-name``,
   ``ingredient-amount``, etc.). Each row is converted to a dictionary and
   collected in an ``ingredients`` list.
5. **Steps with completion status:** The template
   ``view_recipe/sql/get_recipe_steps.sql`` is executed with ``(recipe_id,
   user_id)``. This query joins (via a correlated ``EXISTS`` subquery) the
   ``recipe_step`` table with the ``recipe_step_completion`` table to determine
   whether the given user has completed each step. The subquery checks for the
   existence of a row in ``recipe_step_completion`` matching the step ID and
   the user ID. If such a row exists, the ``CASE`` expression returns ``1``
   (true), otherwise ``0`` (false). The result column is named
   ``step-completion``.
6. Each step row is converted to a dictionary, and the ``step-completion``
   field is explicitly cast to a boolean (``bool(...)``) so the JSON response
   contains ``true``/``false`` rather than ``1``/``0``.
7. The ingredients list is attached to the recipe dictionary as
   ``recipe-ingredients``, and the steps list as ``recipe-steps``.
8. The complete recipe dictionary is returned.

**Response:** JSON object representing the full recipe with completion data.

.. code-block:: json

    {
      "recipe-id": 1,
      "recipe-title": "Pancakes",
      "recipe-difficulty": "Easy",
      "recipe-time": "20 min",
      "recipe-ingredients": [
        {
          "ingredient-name": "Flour",
          "ingredient-amount": 200,
          "ingredient-unit": "g",
          "ingredient-calories": 364
        }
      ],
      "recipe-steps": [
        {
          "recipe-step-id": 10,
          "step-description": "Mix ingredients",
          "step-duration": 5,
          "step-index": 1,
          "step-completion": true
        }
      ]
    }

Route: complete_step
--------------------

**Method:** ``POST``
**Path:** ``/complete-step``
**Decorator:** ``@view_recipe_bp.route('/complete-step', methods=["POST"])``

**Authentication:** Requires ``user_id`` in the session. Returns ``401
Unauthorized`` if missing.

**Request:** JSON body (``application/json``) with the fields:

- ``recipe_step_id`` (int, required) – the ID of the step to mark as completed.
- ``user_id`` (int, required) – the user who completed the step.

**Validation:** The route expects a valid JSON body. If ``get_json()`` returns
``None`` (missing or malformed JSON), a ``400 Bad Request`` is returned with
the message ``"invalid or missing JSON body"``. If either ``recipe_step_id`` or
``user_id`` is missing from the JSON, another ``400`` is returned.

**Processing:**

1. The session is checked for ``user_id``; if absent, abort ``401``.
2. The JSON payload is parsed and validated.
3. A database connection is opened.
4. The SQL template ``view_recipe/sql/complete_step.sql`` is read and executed
   with ``(recipe_step_id, user_id)``. This SQL uses ``INSERT OR IGNORE``, so
   if the completion record already exists, the statement silently does
   nothing. This makes the endpoint **idempotent** – calling it multiple times
   will not create duplicates or errors.
5. The transaction is committed.
6. A success response is returned.

**Response:** ``200 OK`` with ``{"success": true}``. There is no body on error
beyond the ``400`` or ``401`` status.

**Side Effect:** Inserts a row into the ``recipe_step_completion`` table unless
it already exists. Other endpoints (like ``view_recipe``) use this table to
show progress.

Route: uncomplete_step
----------------------

**Method:** ``POST``
**Path:** ``/uncomplete-step``
**Decorator:** ``@view_recipe_bp.route('/uncomplete-step', methods=["POST"])``

**Authentication:** Requires ``user_id`` in session. Returns ``401`` if
missing.

**Request:** JSON body with the same fields as ``complete-step``:

- ``recipe_step_id`` (int, required)
- ``user_id`` (int, required)

**Validation:** Identical to ``complete_step`` – the body must be valid JSON
and both fields must be present.

**Processing:**

1. Session check.
2. JSON parsing and field validation.
3. Database connection opened.
4. The SQL template ``view_recipe/sql/uncomplete_step.sql`` is executed with
   ``(recipe_step_id, user_id)``. This ``DELETE`` statement removes the
   matching row from ``recipe_step_completion``. If no such row existed, the
   statement affects zero rows but does not cause an error, making the endpoint
   **idempotent** as well.
5. Commit and return success.

**Response:** ``200 OK`` with ``{"success": true}``.

**Side Effect:** Removes a row from ``recipe_step_completion``, making the
step appear as not completed again.

Code
----

.. code-block:: python

    from flask import blueprints, current_app, request, abort, session

    from main.database import Database
    from main.paths import PROJECT_MAIN

    view_recipe_bp = blueprints.Blueprint('view-recipe', __name__)


    @view_recipe_bp.route('/view-recipe/<int:recipe_id>', methods=['GET'])
    def view_recipe(recipe_id):
        if "user_id" not in session:
            abort(401)

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

            with open(PROJECT_MAIN / "view_recipe/sql/get_recipe_ingredients.sql") as sql_file:
                sql = sql_file.read()
                cur.execute(sql, (recipe_id,))
                ingredient_data = cur.fetchall()

            ingredients = []
            for row in ingredient_data:
                ingredients.append(dict(row))

            with open(PROJECT_MAIN / "view_recipe/sql/get_recipe_steps.sql") as sql_file:
                sql = sql_file.read()
                cur.execute(sql, (recipe_id, user_id))
                step_data = cur.fetchall()

            steps = []
            for row in step_data:
                row_data = dict(row)
                row_data["step-completion"] = bool(row_data["step-completion"])
                steps.append(row_data)

            recipe["recipe-ingredients"] = ingredients
            recipe["recipe-steps"] = steps

            return recipe


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

SQL Templates
-------------

**get_recipe.sql** (``view_recipe/sql/get_recipe.sql``)

.. code-block:: sql

    SELECT
        r.recipe_id AS "recipe-id",
        r.recipe_title AS "recipe-title",
        r.recipe_difficulty AS "recipe-difficulty",
        r.recipe_time AS "recipe-time"
    FROM
        recipe r
    WHERE
        r.recipe_id = ?

**get_recipe_ingredients.sql** (``view_recipe/sql/get_recipe_ingredients.sql``)

.. code-block:: sql

    SELECT
        recipe_ingredient_name AS "ingredient-name",
        recipe_ingredient_amount AS "ingredient-amount",
        recipe_ingredient_unit AS "ingredient-unit",
        recipe_ingredient_calories as "ingredient-calories"
    FROM
        recipe_ingredient
    WHERE
        recipe_id = ?

**get_recipe_steps.sql** (``view_recipe/sql/get_recipe_steps.sql``)

.. code-block:: sql

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

**complete_step.sql** (``view_recipe/sql/complete_step.sql``)

.. code-block:: sql

    INSERT OR IGNORE INTO recipe_step_completion (recipe_step_id, user_id)
    VALUES (?, ?)

**uncomplete_step.sql** (``view_recipe/sql/uncomplete_step.sql``)

.. code-block:: sql

    DELETE FROM recipe_step_completion
    WHERE recipe_step_id = ? AND user_id = ?
