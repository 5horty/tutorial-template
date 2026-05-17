=====================
Installation & Running
=====================

Backend Setup
-------------

1. Navigate to the backend directory:

.. code-block:: console

   $ cd backend

2. Create and activate a virtual environment:

.. code-block:: console

   $ python -m venv venv
   $ source venv/bin/activate  # On Windows: venv\Scripts\activate

3. Install dependencies:

.. code-block:: console

   (.venv) $ pip install -r requirements.txt

4. Run the Flask server:

.. code-block:: console

   (.venv) $ cd main
   (.venv) $ flask run

The server will start at ``http://127.0.0.1:5000``

To verify the server is running:

.. code-block:: console

   $ curl http://127.0.0.1:5000/

Response:

.. code-block:: json

   {"message":"Hello World!"}

Frontend Setup
--------------

1. Navigate to the frontend directory:

.. code-block:: console

   $ cd frontend

2. Install Flutter dependencies:

.. code-block:: console

   $ flutter pub get

3. Run the Flutter app:

.. code-block:: console

   $ flutter run

Choose your target platform:
- Press ``1`` for Android emulator
- Press ``2`` for iOS simulator (macOS only)
- Press ``3`` for Chrome (web)

The app will launch and connect to the backend at ``http://localhost:5000``

Database Notes
--------------

The database is created automatically when the backend starts for the first time.
To reset the database, delete the file:

.. code-block:: console

   (.venv) $ rm main/database.db

Stop the Servers
----------------

- Backend: Press ``CTRL+C`` in the terminal where Flask is running
- Frontend: Press ``q`` in the terminal where Flutter is running
