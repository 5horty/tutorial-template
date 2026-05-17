=============
Installation & Running
=============

Installation
------------

To set up the Cook App backend, follow these steps:

1. Clone the repository and navigate to the backend directory:

.. code-block:: console

   $ git clone <repository-url>
   $ cd cook-app/backend

2. Create and activate a virtual environment:

.. code-block:: console

   $ python -m venv venv
   $ source venv/bin/activate  # On Windows: venv\Scripts\activate

3. Install the required dependencies:

.. code-block:: console

   (.venv) $ pip install -r requirements.txt

The dependencies include:
- Flask >= 3.1.2 (web framework)
- pytest >= 9.0.2 (testing)
- bcrypt >= 5.0.0 (password hashing - for future use)

Running the Server
------------------

To start the development server:

.. code-block:: console

   (.venv) $ flask --app main.app:create_app run --debug

Or using Python:

.. code-block:: console

   (.venv) $ python -m flask --app main.app:create_app run --debug

The server will start at ``http://127.0.0.1:5000``

You should see output like:

.. code-block:: console

   * Serving Flask app 'main.app:create_app'
   * Debug mode: on
   * Running on http://127.0.0.1:5000 (Press CTRL+C to quit)

To verify the server is running, open another terminal and run:

.. code-block:: console

   $ curl http://127.0.0.1:5000/

Response:

.. code-block:: json

   {"message":"Hello World!"}

Database Initialisation
-----------------------

The database is created automatically when the server starts for the first time.
You don't need to do anything manually. The database file will appear at:

- ``main/database.db`` (for normal operation)
- ``main/test_database.db`` (when TESTING mode is enabled)

To reset the database (delete all data), delete the database file:

.. code-block:: console

   (.venv) $ rm main/database.db

The database will be recreated with fresh test data on the next server start.

Stopping the Server
-------------------

Press ``CTRL+C`` in the terminal where the server is running to stop it.

