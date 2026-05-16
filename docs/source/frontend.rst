Main Entry Point (main.dart)
############################

The **main.dart** file is the starting point of the Flutter application. It contains the minimal logic needed to bootstrap the app, delegating all configuration to dedicated *core* modules.

Overview
********

The ``main()`` function calls ``runApp`` with a ``const CookApp()`` instance. The ``CookApp`` class is a ``StatelessWidget`` that builds the ``MaterialApp`` with:

- A custom light theme from ``AppTheme.lightTheme``.
- The initial route set to the login page via ``AppRoutes.login``.
- A centralized route generator ``AppRoutes.generateRoute`` to handle all named routes dynamically.

This design keeps the entry point clean. All theme and routing definitions live in separate files under `core/`, effectively acting as configuration namespaces.

Widget: CookApp
***************

**Type:** ``StatelessWidget``

**Properties:**

- ``title`` – "Cook App", displayed in the task switcher and other OS integration points.
- ``theme`` – Reference to ``AppTheme.lightTheme`` (defined in ``core/theme.dart``).
- ``initialRoute`` – ``AppRoutes.login`` (defined in ``core/routes.dart``), the first screen shown after launch.
- ``onGenerateRoute`` – ``AppRoutes.generateRoute``, a function that returns a ``Route`` based on the route name and arguments, enabling scalable navigation without hardcoding routes in ``MaterialApp``.

Code
****

.. code-block:: dart

    import 'package:flutter/material.dart';
    import 'package:frontend/core/routes.dart';
    import 'package:frontend/core/theme.dart';


    void main(){
        runApp(const CookApp());
    }

    class CookApp extends StatelessWidget {
        const CookApp({super.key});
    // basically redid main and moved everything out 
    // all core files are basically just name spaces 
        @override
          Widget build(BuildContext context) {
            return MaterialApp(
                title: "Cook App",
                theme: AppTheme.lightTheme, 
                initialRoute: AppRoutes.login,
                onGenerateRoute: AppRoutes.generateRoute
            );
          }
    }

App Theme (core/theme.dart)
###########################

The **theme.dart** file defines the global styling for the application. It sets the main theme used across the app and is applied in ``MaterialApp``.

This keeps all styling in one place instead of being repeated in widgets.

AppTheme Class
**************

The ``AppTheme`` class is used to store theme data.

It is not created as an object and only contains static values.

Light Theme
***********

The app uses a single theme called ``lightTheme``.

It is defined using ``ThemeData``.

Main settings:

- ``colorScheme``  
  Generated using ``ColorScheme.fromSeed`` with a deep purple seed colour  
  This builds a full colour palette automatically

- ``useMaterial3``  
  Set to true to enable Material Design 3 styling

This gives a consistent look across the whole app without manually setting colours for each component.

Design Notes
************

The theme is kept basic on purpose.

- only one theme is defined
- colours are generated from a seed
- no extra overrides are added

This makes it easy to change or expand later if needed.

Code
****

.. code-block:: dart

    import 'package:flutter/material.dart';

    class AppTheme {
        static ThemeData lightTheme = ThemeData(
            colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
            useMaterial3: true,
        );
    }

Session (core/session.dart)
###########################

The **Session** class is used to store simple user session data while the app is running.

It is currently used to keep track of the logged in user id.

Session Class
*************

The class contains a single static field:

- ``userId`` – stores the current user's id, can be null if no user is logged in

This allows the user id to be accessed from anywhere in the app without passing it between pages.

Code
****

.. code-block:: dart

    class Session {
        static int? userId;
    }

App Routes (core/routes.dart)
#############################

Overview
********

The ``routes.dart`` file centralises all navigation within the application.

It defines route names as constants and manages page transitions using a single
route generator function. This prevents navigation logic from being scattered
across multiple features and improves maintainability.

Centralising routes also makes it easier to modify navigation behaviour and
ensures consistency across the application.

Route Constants
***************

The ``AppRoutes`` class defines all named routes used in the application.

.. code-block:: dart

    class AppRoutes {
        static const String login = "/login";
        static const String home = "/home";
        static const String profile = "/profile";
        static const String addRecipe = "/addRecipe";
        static const String createAccount = "/createAccount";
        static const String techniques = "/techniques";
        static const String search = "/search";
        static const String viewRecipe = "/view-recipe";
    }

These constants are used instead of hardcoded strings to reduce errors and improve readability.

Defined Routes
**************

The application supports the following routes:

- ``/login`` - LoginPage
- ``/home`` - HomePage
- ``/profile`` - AccountPage
- ``/addRecipe`` - AddRecipe page
- ``/createAccount`` - CreateAccount page
- ``/techniques`` - CookingTechniquesPage
- ``/search`` - SearchPage
- ``/view-recipe`` - RecipePage

Using named routes ensures consistent navigation throughout the app.

Route Generator
***************

The ``generateRoute`` function controls all navigation logic.

.. code-block:: dart

    static Route<dynamic> generateRoute(RouteSettings settings)

This function reads the requested route name and returns the appropriate page
using a ``MaterialPageRoute``.

Route Matching Flow
*******************

The routing process follows these steps:

1. Read route name from ``settings.name``
2. Match it against defined routes
3. Return the correct page widget
4. Pass any required arguments

This ensures all navigation is handled in a single location.

Debug Checks
************

Basic validation is performed on the route name:

.. code-block:: dart

    if (settings.name == null) {
        print("is null");
    }

    if (settings.name!.isEmpty) {
        print("is empty");
    }

    print("${settings.name}");

These debug statements help identify navigation issues during development.

Routes Without Arguments
************************

Some routes do not require additional data and can be opened directly.

.. code-block:: dart

    case addRecipe:
        return MaterialPageRoute(builder: (_) => AddRecipe());

    case createAccount:
        return MaterialPageRoute(builder: (_) => CreateAccount());

    case search:
        return MaterialPageRoute(builder: (_) => const SearchPage());

    case techniques:
        return MaterialPageRoute(
            builder: (_) => const CookingTechniquesPage(),
        );

These pages are static or self contained.

Routes With Arguments
*********************

Some pages require runtime data passed through ``settings.arguments``.

The arguments are cast to a ``Map<String, dynamic>`` before use.

Home Page
^^^^^^^^^

.. code-block:: dart

    case home:
        final args = settings.arguments as Map<String, dynamic>;
        return MaterialPageRoute(
            builder: (_) => HomePage(
                username: args["username"],
                newAccount: args["newAccount"],
            ),
        );

Profile Page
^^^^^^^^^^^^

.. code-block:: dart

    case profile:
        final args = settings.arguments as Map<String, dynamic>;
        return MaterialPageRoute(
            builder: (_) => AccountPage(
                username: args["username"],
                newAccount: args["newAccount"],
            ),
        );

View Recipe Page
^^^^^^^^^^^^^^^^

.. code-block:: dart

    case viewRecipe:
        final args = settings.arguments as Map<String, dynamic>;
        return MaterialPageRoute(
            builder: (_) => RecipePage(
                recipeId: args["recipeId"],
            ),
        );

Default Route
*************

If an invalid route is requested, a fallback page is shown.

.. code-block:: dart

    default:
        return MaterialPageRoute(
            builder: (_) => const Scaffold(
                body: Center(child: Text("error no route")),
            ),
        )

This prevents the application from crashing due to invalid navigation calls.

Navigation Design
*****************

The routing system follows a centralised architecture:

- All routes defined in one class
- All navigation handled in one function
- Pages remain decoupled from navigation logic

This improves scalability and maintainability.

Summary
*******

The ``routes.dart`` file provides a central navigation system for the application.

It defines route constants, handles page transitions, supports argument passing,
and ensures safe fallback behaviour for invalid routes.

Api Response (core/api_response.dart)
#####################################

The **ApiResponse** class is a simple model used to standardise responses from API calls.

It is used to store the HTTP status code and the response body in a single object.

ApiResponse Class
*****************

The class contains two fields:

- ``statusCode`` – the HTTP status code returned from the server
- ``response`` – the raw response body as a string

This makes it easier to handle API results consistently across the app.

Code
****

.. code-block:: dart

    class ApiResponse {
        final int statusCode;
        final String response;

        ApiResponse({required this.statusCode, required this.response});
    }

Login Page (features/authen/login_page.dart)
############################################

Overview
********

The login page handles user authentication. It collects user credentials, sends them to the backend authentication service, and navigates to the home page if login is successful.

It is implemented as a StatefulWidget because it manages form state and asynchronous API calls.

User Input Handling
*******************

A helper class stores text controllers for:

- first name
- last name
- password

These controllers capture user input from the form fields.

Authentication Flow
*******************

When the user submits the form:

1. Input is validated
2. Credentials are sent to AuthService
3. Response is decoded from JSON
4. Session user ID is stored

.. code-block:: dart

    final data = await AuthService.authenticate(
        userFname,
        userLname,
        password,
    );

Successful Login
****************

If authentication succeeds:

- username is constructed
- user is redirected to home page
- success message is shown

.. code-block:: dart

    Navigator.pushReplacementNamed(
        context,
        AppRoutes.home,
        arguments: {
            "username": username,
            "newAccount": newAccount,
        },
    );

Failed Login
************

If login fails:

- an error message is displayed
- no navigation occurs

Navigation
**********

The page supports navigation to:

- Home page (on success)
- Create account page

Create Account Flow
*******************

The create account button navigates to the create account page and waits for a result.

If account creation succeeds:
- user is automatically redirected to home page

Error Handling
**************

All authentication calls are wrapped in try-catch blocks.

If an error occurs:
- a server error message is shown
- exception is logged to console

Form Validation
***************

Each input field includes validation:

- must not be empty
- password is obscured for security

Summary
*******

The login page provides authentication functionality for the application. It validates user input, communicates with the backend, manages session state, and handles navigation after login.

Create Account Page (features/authen/create_account.dart)
#########################################################

Overview
********

The CreateAccount page is used to register new users in the application. It collects user information, validates the input locally, and sends a request to the backend authentication service to create a new account.

The page is implemented as a StatefulWidget because it manages form state, loading state, and asynchronous API communication.

State Management
****************

This page manages multiple types of state:

- form input values using text controllers
- loading state while the request is in progress
- validation state from the form

Form Controllers
****************

The page uses several TextEditingController instances to capture user input:

- first name
- last name
- email
- password
- confirm password

Each controller stores the current value of its respective field. All controllers are properly disposed in dispose() to prevent memory leaks.

Form Validation
***************

Validation is handled through helper functions attached to each form field.

Rules include:

- first name and last name must not be empty and must be at least 2 characters
- email must contain the @ symbol
- password must be at least 6 characters
- confirm password must match password

Each validator returns a string error message if validation fails, otherwise null.

Submit Function
***************

The _submit function handles the full account creation process.

Flow:

1. Validate the form
2. Read and trim input values
3. Set loading state to true
4. Call AuthService.createAccount
5. Handle response from backend
6. Reset loading state

.. code-block:: dart

    final resp = await AuthService.createAccount(
        first,
        last,
        email,
        password,
    );

Successful Response
*******************

If the backend returns a success status code:

- a success message is shown using a snackbar
- the page is closed using Navigator.maybePop

This returns the user to the previous screen, typically the login page.

Failed Response
***************

If the request fails:

- a default error message is shown
- the response body is parsed if possible
- backend error messages are displayed when available

If parsing fails, a generic error message is used instead.

Error Handling
**************

All API calls are wrapped in a try catch block.

If an exception occurs:

- the error is shown in a snackbar
- loading state is reset in finally block

This ensures the UI never remains stuck in a loading state.

Loading State
*************

The _loading boolean controls UI behaviour during submission.

When true:

- the submit button is disabled
- a circular progress indicator is shown inside the button

When false:

- the normal submit button is shown

This prevents duplicate submissions.

UI Layout
*********

The UI is built using a SingleChildScrollView containing a Form widget.

The layout is vertical and includes:

- first name field
- last name field
- email field
- password field
- confirm password field
- submit button

Each field uses an InputDecoration with labels for clarity.

Form Fields
***********

First Name
^^^^^^^^^^

- must not be empty
- minimum length validation applied

Last Name
^^^^^^^^^

- same validation rules as first name

Email
^^^^^

- must contain @ symbol
- basic format validation only

Password
^^^^^^^^

- hidden input
- minimum length requirement

Confirm Password
^^^^^^^^^^^^^^^^

- must match password field exactly
- ensures user input consistency

User Experience Flow
********************

The expected user flow is:

1. User opens create account page
2. Fills in all required fields
3. Validation runs on submit
4. Request is sent to backend
5. Success or error message is displayed
6. User is returned to previous screen on success

Summary
*******

The CreateAccount page provides a complete user registration flow. It handles input validation, backend communication, error handling, and loading state management in a single self-contained screen.

Auth Service (features/authen/services/account_services.dart)
#############################################################

Overview
********

The ``AuthService`` class manages all authentication related network requests
for the application.

It is responsible for:

- user login
- account creation
- basic session handling using cookies

This service provides a lightweight authentication system without requiring
external state management libraries.

Session Handling
****************

Authentication state is stored in memory using a static session cookie.

.. code-block:: dart

    static String? _sessionCookie;

The service provides helper methods to manage this value.

Set Session
^^^^^^^^^^^

.. code-block:: dart

    static void setSessionCookie(String cookie)

Stores the session cookie after a successful login.

Clear Session
^^^^^^^^^^^^^

.. code-block:: dart

    static void clearSession()

Removes the stored session cookie when a user logs out.

Check Authentication
^^^^^^^^^^^^^^^^^^^^

.. code-block:: dart

    static bool isAuthenticated()

Returns true if a session cookie exists.

This is a simple way to track login state across the app.

How Session Cookies Work
************************

When a user logs in successfully:

- the backend returns a ``set-cookie`` header
- this cookie contains the session identifier
- the app extracts and stores it in memory

This cookie is then attached to future requests to authenticate the user.

Authenticate (Login)
********************

The ``authenticate`` function sends login credentials to the backend.

.. code-block:: dart

    static Future<ApiResponse> authenticate(
        String fname,
        String lname,
        String password,
    )

Request Details
^^^^^^^^^^^^^^^

- Method: POST
- Endpoint: ``/authenticate``
- Content type: ``application/x-www-form-urlencoded``

Request Body
^^^^^^^^^^^^

.. code-block:: dart

    {
        "user_fname": fname,
        "user_lname": lname,
        "user_password": password
    }

Session Extraction
^^^^^^^^^^^^^^^^^^

If the login is successful (status code 200), the session cookie is extracted:

.. code-block:: dart

    final cookies = response.headers["set-cookie"];
    if (cookies != null) {
        _sessionCookie = cookies.split(';')[0];
    }

This stores only the first part of the cookie string, which contains the session ID.

Return Value
^^^^^^^^^^^^

The function returns an ``ApiResponse`` containing:

- status code
- response body

Create Account
**************

The ``createAccount`` function registers a new user in the system.

.. code-block:: dart

    static Future<ApiResponse> createAccount(
        String fname,
        String lname,
        String email,
        String password,
    )

Request Details
^^^^^^^^^^^^^^^

- Method: POST
- Endpoint: ``/create-account``
- Content type: ``application/x-www-form-urlencoded``

Request Body
^^^^^^^^^^^^

.. code-block:: dart

    {
        "user_fname": fname,
        "user_lname": lname,
        "user_email": email,
        "user_password": password
    }

Response Handling
^^^^^^^^^^^^^^^^^

Unlike login, account creation does not automatically store a session cookie.

It simply returns an ``ApiResponse`` object with:

- status code
- response body

Error Handling
**************

There is no explicit retry or timeout logic in this service.

If a request fails:

- the HTTP package throws an exception
- it must be handled by the calling UI layer

Limitations
***********

- Session is stored only in memory
- No persistent login across app restarts
- No token refresh or expiry handling
- No request retry logic
- Base URL is hardcoded to localhost

Summary
*******

The AuthService provides basic authentication functionality including login,
account creation, and session tracking using cookies.

It is intentionally lightweight and relies on the backend for security and
session validation.

Home Page (features/recipe/home_page.dart)
##########################################

Overview
********

The home page is the main landing screen after a user logs in. It acts as a central hub for the application, showing available recipes and providing navigation to core features such as searching, adding recipes, viewing techniques, and accessing the user profile.

The page is implemented as a StatefulWidget because it manages dynamic data such as recipe loading state and API responses.

Imports
*******

.. code-block:: dart

    import 'package:flutter/material.dart';
    import 'package:frontend/core/routes.dart';
    import 'dart:convert';
    import 'package:http/http.dart' as http;

The file uses:
- Flutter widgets for UI
- HTTP for API requests
- JSON decoding for backend responses
- App routes for navigation

State Variables
***************

The page maintains two main state variables:

- recipes: stores the list of recipes from the backend
- isLoading: tracks whether data is currently being fetched

.. code-block:: dart

    List<dynamic> recipes = [];
    bool isLoading = true;

Page Initialisation
*******************

When the page loads, it immediately fetches recipes from the backend.

.. code-block:: dart

    @override
    void initState() {
        super.initState();
        _loadRecipes();
    }

This ensures content is displayed as soon as the user enters the page.

Loading Recipes
***************

Recipes are retrieved using a GET request to the backend.

.. code-block:: dart

    final response = await http.get(
        Uri.parse("http://localhost:5000/get-recipes"),
    );

If successful, the response is decoded and stored locally.

.. code-block:: dart

    final data = jsonDecode(response.body);
    recipes = data["recipes"] ?? [];

Error Handling
**************

The request is wrapped in a try-catch block.

If the request fails:
- the error is printed to the console
- loading state is disabled to prevent infinite loading UI

.. code-block:: dart

    catch (e) {
        debugPrint("error: $e");
    }

User Interface
**************

The page layout includes:

- App bar with user greeting
- Quick action buttons
- Recipe list section
- Loading indicator when needed

App Bar
*******

The app bar displays a welcome message using the username passed into the page.

Navigation buttons allow access to:
- Profile page
- Settings (debug placeholder)

Quick Actions
*************

Quick actions provide shortcuts to key features:

- Search recipes
- Add recipe
- Cooking techniques

Each action uses a reusable UI widget to maintain consistency.

Recipe List
***********

Recipes are displayed using a ListView builder.

Each recipe card includes:
- Recipe image
- Title
- Difficulty level
- Navigation arrow

.. code-block:: dart

    ListView.builder(
        itemCount: recipes.length,
    )

Recipe Images
*************

Images are loaded from the backend using the recipe ID.

.. code-block:: dart

    Image.network(
        "http://localhost:5000/recipe-image/${recipe["recipe_id"]}"
    )

If the image fails to load, a placeholder icon is shown instead.

Navigation
**********

Tapping a recipe opens the view recipe page.

The recipe ID is passed through named routes.

.. code-block:: dart

    Navigator.pushNamed(
        context,
        AppRoutes.viewRecipe,
        arguments: {
            "recipeId": recipe["recipe_id"]
        },
    );

Refresh Support
***************

The page supports pull-to-refresh using RefreshIndicator.

.. code-block:: dart

    RefreshIndicator(
        onRefresh: _loadRecipes,
    )

This allows users to manually reload recipe data.

Summary
*******

The home page acts as the main navigation hub of the application. It loads recipe data from the backend, displays it in a structured format, and provides access to all major features.

Add Recipe Page (features/add_recipe/add_recipe.dart)
#####################################################

Overview
********

The AddRecipe page is used to create and submit new recipes to the application. It allows users to enter recipe details such as ingredients, cooking steps, timing, difficulty level, and images.

The page is complex because it supports dynamic form generation, nested step structures, and image uploads.

State Management
****************

This page is implemented as a StatefulWidget because it manages multiple types of mutable state:

- form input values
- dynamic lists of ingredients and steps
- sub steps inside each step
- image files
- loading and submission state

Main Data Structures
********************

The page uses helper classes to organise structured input data.

Ingredient
^^^^^^^^^^

Represents a single ingredient entry:

- name controller
- amount controller
- calories controller
- unit selection

.. code-block:: dart

    class Ingredient {
        final TextEditingController name = TextEditingController();
        final TextEditingController amount = TextEditingController();
        final TextEditingController calories = TextEditingController();
        String? amountUnits;
    }

StepItem
^^^^^^^^

Represents a main cooking step:

- description controller
- duration fields (hours and minutes)
- optional image
- list of sub steps

.. code-block:: dart

    class StepItem {
        final TextEditingController controller = TextEditingController();
        final List<SubStep> subSteps = [];
        final TextEditingController minutes = TextEditingController();
        final TextEditingController hours = TextEditingController();
        File? image;
    }

SubStep
^^^^^^^

Represents a nested step inside a main step:

- description controller
- duration fields
- optional image

.. code-block:: dart

    class SubStep {
        final TextEditingController subStep = TextEditingController();
        final TextEditingController subMinutes = TextEditingController();
        final TextEditingController subHours = TextEditingController();
        File? image;
    }

Form Setup
**********

The page uses a GlobalKey<FormState> for validation.

By default, the form is preloaded with:

- 3 ingredients
- 3 steps

Users can dynamically add more items as needed.

Image Handling
**************

The page supports multiple image inputs:

- main recipe image
- step images
- sub step images

Images are selected using image_picker and stored as File objects.

If image selection fails:

- an error message is displayed using a snackbar
- the process safely returns null

Helper Widgets and Utilities
****************************

Several helper methods are used to simplify the UI:

- buildTextInputField: reusable text input component
- buildImagePicker: image selector with preview and remove option
- buildDurationFields: input fields for hours and minutes

These helpers reduce duplication and keep the UI consistent.

Duration Format
***************

All time values are converted into ISO 8601 duration format.

Example:

- 1 hour 30 minutes becomes PT1H30M

This is handled by the _durationToISO function.

.. code-block:: dart

    String _durationToISO(int hours, int minutes) {
        String result = "PT";
        if (hours > 0) result += "${hours}H";
        if (minutes > 0) result += "${minutes}M";
        if (result == "PT") result += "0M";
        return result;
    }

Recipe Submission Flow
**********************

The _saveRecipe function handles the full submission process.

The flow is:

1. Validate the form
2. Read recipe metadata (name, difficulty, time)
3. Convert ingredients into structured JSON format
4. Convert steps and sub steps into ordered structure
5. Attach image file paths
6. Send data using RecipeService.addRecipe

Steps are indexed in a hierarchical format:

- Main steps: 1, 2, 3
- Sub steps: 1.1, 1.2, 2.1, etc

Success Handling
****************

If the request succeeds:

- a success message is shown
- the page is closed using Navigator.pop or maybePop

Failure Handling
****************

If the request fails:

- an error message is shown
- the user remains on the page to correct input

Error Handling
**************

Image selection and submission are wrapped in try catch blocks.

If an error occurs:

- a snackbar message is shown
- the UI remains stable
- the function safely exits

Dynamic Input Behaviour
***********************

Users can dynamically extend the form:

- add new ingredients
- add new steps
- add sub steps inside steps

Each update uses setState to refresh the UI.

Validation Rules
****************

The form includes basic validation:

- recipe name is required
- ingredient fields must be filled
- step descriptions are required
- difficulty must be selected

UI Structure
************

The page is structured as a scrollable form containing:

- recipe name input
- main image selector
- total duration fields
- difficulty dropdown
- ingredient list section
- step and sub step editor
- save button

This structure allows users to build complex recipes in a guided way.

Summary
*******

The AddRecipe page provides a full recipe creation system with dynamic inputs, nested step support, image uploads, and structured submission to the backend service.

Recipe Service (features/add_recipe/services/add_recipe_services.dart)
#######################################################################

Overview
********

The ``RecipeService`` is responsible for sending fully constructed recipe data
from the Flutter frontend to the backend API.

It uses a multipart form request because the payload contains both structured
JSON data and binary image files.

This service acts as the bridge between the UI layer and the backend
``/add-recipe`` endpoint.

Why Multipart Form Data Is Used
*******************************

A normal JSON request cannot directly send files such as images.

Instead, ``multipart/form-data`` is used because it allows:

- sending text fields (recipe name, difficulty, time)
- sending structured JSON strings (ingredients and steps)
- sending multiple binary files (images)

Each part of the request is sent as a separate "field" inside the multipart body.

Add Recipe Function
*******************

The main function is:

.. code-block:: dart

    static Future<bool> addRecipe(
        String name,
        List ingredients,
        List steps,
        String time,
        String difficulty,
        String? mainImage,
    )

It collects all recipe data and prepares it for transmission.

Request Type
************

- HTTP method: POST
- Format: multipart/form-data
- Endpoint: ``/add-recipe``

Authentication
**************

If a user session exists, authentication is added using cookies.

.. code-block:: dart

    if (AuthService.sessionCookie != null) {
        request.headers["Cookie"] = AuthService.sessionCookie!;
    }

This allows the backend to associate the recipe with the logged-in user.

Multipart Request Structure
***************************

The request is built using:

.. code-block:: dart

    var request = http.MultipartRequest(
        'POST',
        Uri.parse("http://localhost:5000/add-recipe"),
    );

A multipart request is composed of:

1. Text fields (key value pairs)
2. File fields (binary uploads)

Text Fields
***********

These fields are added using:

.. code-block:: dart

    request.fields['recipe-title'] = name;
    request.fields['recipe-ingredients'] = jsonEncode(ingredients);
    request.fields['recipe-steps'] = jsonEncode(steps);
    request.fields['recipe-time'] = time;
    request.fields['recipe-difficulty'] = difficulty;

Important details:

- ingredients and steps are encoded using ``jsonEncode``
- this allows complex lists and objects to be sent as strings
- backend must decode these JSON strings back into usable structures

Image Handling
**************

The service supports two types of images:

1. Main recipe image
2. Step-specific images

Main Recipe Image
^^^^^^^^^^^^^^^^^

The main image is optional.

It is added only if:

- path is not null
- file exists on disk

.. code-block:: dart

    File imageFile = File(mainImage);

    if (await imageFile.exists()) {
        request.files.add(
            await http.MultipartFile.fromPath(
                "recipe-main-image",
                mainImage,
            ),
        );
    }

This attaches the image as a file part in the multipart request.

Step Images
^^^^^^^^^^^

Each step can have its own image.

The system loops through all steps:

.. code-block:: dart

    for (var i = 0; i < steps.length; i++) {
        var step = steps[i];
    }

For each step:

- it checks if a step image exists
- verifies the file exists on disk
- uploads it if valid

.. code-block:: dart

    if (step['step-image'] != null &&
        step['step-image'].isNotEmpty) {

        File stepImageFile = File(step['step-image']);

        if (await stepImageFile.exists()) {
            request.files.add(
                await http.MultipartFile.fromPath(
                    'step-image-${step['step-index']}',
                    step['step-image'],
                ),
            );
        }
    }

Step Image Key Format
^^^^^^^^^^^^^^^^^^^^^

Each image is sent using a dynamic key:

``step-image-{step-index}``

Example:

- step-image-1
- step-image-2
- step-image-3

This is important because:

- backend uses the index to match images to correct steps
- ensures images are not mixed between steps
- supports variable number of steps

Sending the Request
*******************

The request is sent using a streamed HTTP call:

.. code-block:: dart

    var streamedResponse = await request.send();

A streamed response is used because multipart uploads may be large.

It is then converted into a normal HTTP response:

.. code-block:: dart

    var response = await http.Response.fromStream(streamedResponse);

Response Handling
*****************

The backend response is evaluated using status codes:

Success cases:

- 200 OK
- 201 Created

.. code-block:: dart

    if (response.statusCode == 200 || response.statusCode == 201) {
        return true;
    }

Failure case:

- any other status code

.. code-block:: dart

    throw Exception(
        "Failed to add recipe: ${response.statusCode} - ${response.body}",
    );

Error Handling
**************

The entire function is wrapped in a try catch block.

This ensures:

- network errors are caught
- file errors are handled
- invalid request construction does not crash the app

If an error occurs:

- it is rethrown as an exception with a message

Notes
*****

- Base URL is currently hardcoded to localhost
- Ingredients and steps must be JSON serialisable
- Images are uploaded as file streams
- Backend must parse multipart fields and decode JSON strings

Summary
*******

The RecipeService uses a multipart form request to send both structured data
and images in a single API call.

It supports:

- JSON encoded recipe metadata
- multiple image uploads
- dynamic step indexing
- authenticated requests using cookies

Search Page (features/search_recipe/search_page.dart)
#####################################################

Overview
********

The search page allows users to search for recipes using a keyword query. It sends requests to the backend and displays matching results in real time.

The page is implemented as a StatefulWidget because it manages search input, loading state, and dynamic results.

Imports
*******

.. code-block:: dart

    import 'package:flutter/material.dart';
    import 'package:frontend/core/routes.dart';
    import 'dart:convert';
    import 'services/search_recipe_service.dart';

State Variables
***************

The page maintains:

- query: current search input
- results: list of recipes returned from the backend
- isLoading: indicates whether a search request is in progress
- error: stores error messages if needed

.. code-block:: dart

    String query = "";
    List<dynamic> results = [];
    bool isLoading = false;

Search Function
***************

The search function is triggered whenever the user types.

It sends the query to the backend service.

.. code-block:: dart

    final apiResponse = await SearchService.searchRecipe(name);

If successful, the response is decoded and stored.

.. code-block:: dart

    final data = jsonDecode(apiResponse.response);
    results = data["recipes"];

Loading State
*************

While waiting for results:
- isLoading is set to true
- a progress indicator is shown

If the search completes:
- loading state is disabled
- results are updated

User Interface
**************

The UI consists of:

- Search input field
- Loading indicator
- Results list

Search Input
************

The text field triggers a search on every input change.

.. code-block:: dart

    onChanged: (value) {
        query = value;
        _search(value);
    }

Results Display
***************

Results are displayed using a ListView builder.

Each result shows:
- recipe image
- title
- ingredients
- time
- difficulty

.. code-block:: dart

    ListView.builder(
        itemCount: results.length,
    )

Recipe Card
***********

Each recipe is displayed inside a Card widget.

The card is tappable and navigates to the recipe view page.

.. code-block:: dart

    Navigator.pushNamed(
        context,
        AppRoutes.viewRecipe,
        arguments: {
            "recipeId": recipe["recipe_id"]
        },
    );

Recipe Images
*************

Images are loaded from the backend using recipe ID.

If the image fails to load, a placeholder icon is shown.

.. code-block:: dart

    Image.network(
        "http://localhost:5000/recipe-image/$recipeId"
    )

Empty States
************

The page handles multiple states:

- empty query: prompt user to type
- loading: show spinner
- no results: show "no results found"

Summary
*******

The search page provides real time recipe lookup functionality. It connects to the backend search API, displays structured results, and allows navigation to detailed recipe views.

Search Service (features/search_recipe/services/search_recipe_service.dart)
###########################################################################

Overview
********

The search service handles communication between the Flutter application and the backend search API.

It is responsible for sending search queries and returning structured responses to the UI layer.

Imports
*******

.. code-block:: dart

    import 'package:frontend/features/authen/services/account_services.dart';
    import 'package:http/http.dart' as http;
    import 'package:frontend/core/api_response.dart';

Search Function
***************

The main function performs a GET request to the backend search endpoint.

.. code-block:: dart

    static Future<ApiResponse> searchRecipe(String name)

Request Construction
********************

The request is built using the search query as a URL parameter.

.. code-block:: dart

    Uri.parse("http://localhost:5000/search-recipe")
        .replace(queryParameters: {"q": name});

Session Handling
****************

If the user is authenticated, a session cookie is included in the request headers.

.. code-block:: dart

    if (AuthService.sessionCookie != null)
        'Cookie': AuthService.sessionCookie!,

Response Handling
*****************

The response is wrapped in a custom ApiResponse object containing:

- statusCode
- response body

.. code-block:: dart

    return ApiResponse(
        statusCode: response.statusCode,
        response: response.body,
    );

Error Handling
**************

The function does not directly handle UI errors.

Instead, it returns raw response data so the UI layer can decide how to display errors.

Summary
*******

The search service acts as a simple API wrapper that sends search queries to the backend and returns structured responses to the application.

View Recipe Page (features/view_recipe/view_recipe.dart)
########################################################

Overview
********

The ``view_recipe.dart`` file implements the recipe viewing page of the application. 
This page allows users to load a recipe, follow each cooking step, track progress, 
and mark steps as completed.

The page communicates with the backend using the ``ViewService`` class and updates 
the interface dynamically as the user progresses through the recipe.

Imports
*******

.. code-block:: dart

   import 'dart:convert';
   import 'package:flutter/material.dart';
   import 'package:frontend/features/view_recipe/services/service_view_recipe.dart';

The file imports:

* ``dart:convert`` for decoding JSON API responses.
* ``material.dart`` for Flutter UI widgets.
* ``service_view_recipe.dart`` for backend API communication.

RecipePage Widget
*****************

The page is implemented as a ``StatefulWidget`` because the UI changes while the 
user interacts with recipe steps.

.. code-block:: dart

   class RecipePage extends StatefulWidget {
     final int recipeId;
   }

The ``recipeId`` is passed into the page and used to retrieve recipe data 
from the backend.

State Variables
***************

The page stores recipe information using several state variables.

.. code-block:: dart

   String recipeTitle = "";
   String? recipeTime;
   String? recipeDifficulty;

These variables store recipe metadata.

Recipe steps are stored inside a list.

.. code-block:: dart

   List<Map<String, dynamic>> steps = [];

Completed steps are tracked using a ``Set``.

.. code-block:: dart

   Set<int> completedSteps = {};

The current active step is stored using:

.. code-block:: dart

   int? currentStepId;

Loading state is tracked using:

.. code-block:: dart

   bool isLoading = true;

Recipe Loading
**************

Recipe data is retrieved using the ``loadRecipe()`` function.

.. code-block:: dart

   final response = await ViewService.viewRecipe(
       widget.recipeId.toString()
   );

The API response is decoded from JSON.

.. code-block:: dart

   final data = jsonDecode(response.response);

Recipe title, duration, and difficulty are extracted from the response.

Recipe steps are converted into local step objects containing:

* Step ID
* Step description
* Step duration
* Completion status

The first incomplete step becomes the active step.

Progress Tracking
*****************

Recipe completion progress is calculated dynamically.

.. code-block:: dart

   double get progress =>
       steps.isEmpty ? 0 :
       completedSteps.length / steps.length;

This value is displayed using a ``LinearProgressIndicator``.

Completing Steps
****************

The ``completedStep()`` function marks the current recipe step as completed.

The function performs the following actions:

1. Adds the current step to ``completedSteps``.
2. Finds the next incomplete step.
3. Updates the UI.
4. Sends the update to the backend API.

.. code-block:: dart

   await ViewService.completeStep(stepToComplete!);

Going Backwards
***************

Users can return to a previous step using the ``goBack()`` function.

The function:

* Removes the latest completed step.
* Updates the current step.
* Sends the update to the backend.

.. code-block:: dart

   await ViewService.unCompleteStep(stepToUncomplete);

Step Images
***********

Recipe step images are loaded dynamically using ``Image.network``.

.. code-block:: dart

   Image.network(
       "http://localhost:5000/step-image/$stepId"
   )

If the image fails to load, an empty widget is displayed instead.

User Interface
**************

The page interface contains:

* Recipe title
* Recipe duration
* Difficulty level
* Current step number
* Step instructions
* Step image
* Progress bar
* Navigation buttons

The current step number is calculated dynamically.

.. code-block:: dart

   steps.indexWhere((s) => s["id"] == currentStepId) + 1

Error Handling
**************

Network requests are wrapped inside ``try-catch`` blocks.

.. code-block:: dart

   catch (e) {
       debugPrint("error $e");
   }

This prevents application crashes when API requests fail.

Summary
*******

This file manages the complete recipe viewing workflow. It retrieves recipe 
information from the backend, displays cooking steps, tracks user progress, 
and synchronises completion state with the API.

Recipe View Service (features/view_recipe/services/service_view_recipe.dart)
############################################################################

Overview
********

The ``service_view_recipe.dart`` file contains the networking logic used by the 
recipe viewing system.

The service communicates with the backend API to:

* Retrieve recipe information
* Mark recipe steps as completed
* Remove completed step progress

Separating networking logic into a service improves code organisation and keeps 
UI logic cleaner.

Imports
*******

.. code-block:: dart

   import 'dart:convert';

   import 'package:frontend/core/api_response.dart';
   import 'package:frontend/core/session.dart';
   import 'package:frontend/features/authen/services/account_services.dart';
   import 'package:http/http.dart' as http;

The file imports:

* ``dart:convert`` for JSON encoding.
* ``ApiResponse`` for standardised API responses.
* ``Session`` for retrieving the current user ID.
* ``AuthService`` for session authentication cookies.
* ``http`` for performing HTTP requests.

ViewService Class
*****************

The ``ViewService`` class contains static networking methods.

.. code-block:: dart

   class ViewService {

Static methods allow the service to be used without creating an object instance.

View Recipe Request
*******************

The ``viewRecipe()`` function retrieves recipe information from the backend.

.. code-block:: dart

   static Future<ApiResponse> viewRecipe(
       String recipeId
   ) async {

A GET request is sent to the backend.

.. code-block:: dart

   Uri.parse(
       "http://localhost:5000/view-recipe/$recipeId"
   )

The current user ID is included as a query parameter.

.. code-block:: dart

   queryParameters: {
       "user_id": Session.userId.toString()
   }

Authentication cookies are attached when available.

.. code-block:: dart

   if (AuthService.sessionCookie != null)
       'Cookie': AuthService.sessionCookie!

The response is returned as an ``ApiResponse`` object.

Complete Step Request
*********************

The ``completeStep()`` function marks a recipe step as completed.

.. code-block:: dart

   static Future<ApiResponse> completeStep(
       int recipeStepId
   ) async {

A POST request is sent to:

.. code-block:: dart

   http://localhost:5000/complete-step

The request body contains:

* Recipe step ID
* User ID

The request body is converted into JSON.

.. code-block:: dart

   body: jsonEncode({
       "recipe_step_id": recipeStepId,
       "user_id": Session.userId,
   })

The backend stores the updated progress information.

Uncomplete Step Request
***********************

The ``unCompleteStep()`` function removes progress from a completed recipe step.

.. code-block:: dart

   static Future<ApiResponse> unCompleteStep(
       int recipeStepId
   ) async {

A POST request is sent to:

.. code-block:: dart

   http://localhost:5000/uncomplete-step

This updates the backend database so recipe progress remains synchronised.

Authentication
**************

All requests include the session cookie when available.

.. code-block:: dart

   headers: {
       if (AuthService.sessionCookie != null)
           'Cookie': AuthService.sessionCookie!,
   }

This ensures that only authenticated users can access protected endpoints.

Error Handling
**************

The service returns API responses directly to the UI layer.

The UI is responsible for checking status codes and handling errors.

Summary
*******

This service handles all networking functionality related to viewing recipes 
and updating recipe step progress. Centralising API logic improves 
maintainability and separates backend communication from the user interface.

Cooking Techniques Page (features/techniques/cooking_techniques_page.dart)
##########################################################################

Overview
********

The ``CookingTechniquesPage`` is a static informational page that displays
common cooking techniques for users.

It does not use any backend services or API calls. All data is stored locally
inside the widget.

The purpose of this page is to help users learn basic cooking methods in a
simple and structured format.

Data Model: CookingTechnique
****************************

The ``CookingTechnique`` class is used to store information for each cooking method.

.. code-block:: dart

    class CookingTechnique {
      const CookingTechnique({
        required this.title,
        required this.description,
        required this.skillLevel,
        required this.bestFor,
        required this.timeRange,
        required this.steps,
        required this.icon,
        required this.accent,
      });

      final String title;
      final String description;
      final String skillLevel;
      final String bestFor;
      final String timeRange;
      final List<String> steps;
      final IconData icon;
      final Color accent;
    }

This model is only used for UI display and does not interact with any backend.

Static Data
***********

All cooking techniques are stored in a constant list called ``_techniques``.

.. code-block:: dart

    static const List<CookingTechnique> _techniques = [
      CookingTechnique(
        title: 'Saute',
        description:
            'Cook quickly over medium-high heat with a small amount of fat.',
        skillLevel: 'Beginner',
        bestFor: 'Vegetables, shrimp, sliced chicken',
        timeRange: '5-12 min',
        steps: [
          'Heat pan first, then add oil.',
          'Keep ingredients moving for even browning.',
          'Do not crowd the pan to avoid steaming.',
        ],
        icon: Icons.local_fire_department,
        accent: Color(0xFFE26D5A),
      ),
    ];

UI Structure
************

The page is built using a scrollable ``ListView`` that contains:

- Header card
- Technique cards

.. code-block:: dart

    ListView(
      padding: const EdgeInsets.fromLTRB(16, 16, 16, 24),
      children: [
        _HeaderCard(techniqueCount: _techniques.length),
        const SizedBox(height: 16),
        ..._techniques.map((technique) {
          return _TechniqueCard(technique: technique);
        }),
      ],
    )

Header Card
***********

The header displays an introduction message and total technique count.

.. code-block:: dart

    class _HeaderCard extends StatelessWidget {
      const _HeaderCard({required this.techniqueCount});

      final int techniqueCount;
    }

Technique Card
**************

Each technique is displayed using a card widget.

It includes:

- Title and icon
- Skill level chip
- Description
- Metadata rows
- Step-by-step instructions

.. code-block:: dart

    class _TechniqueCard extends StatelessWidget {
      const _TechniqueCard({required this.technique});

      final CookingTechnique technique;
    }

Helper Widget: Meta Row
***********************

Used to display label-value pairs such as "Best for" or "Time range".

.. code-block:: dart

    class _MetaRow extends StatelessWidget {
      const _MetaRow({required this.label, required this.value});

      final String label;
      final String value;
    }

Behaviour
*********

- Fully static UI
- No API calls
- No state changes
- Purely informational screen

Summary
*******

The ``CookingTechniquesPage`` provides a simple reference screen for common
cooking techniques.

It is static, lightweight, and designed for learning and quick access.

Account Page (features/profile/profile_page.dart)
#################################################

Overview
********

The AccountPage is a static user settings page.

It allows users to:

- view profile information
- select dietary preferences
- enter diet notes
- view mock recipe progress
- sign out

Only sign out communicates with the backend session system.

State Management
****************

This page is a StatefulWidget because it manages:

- selected dietary options
- notes input
- mock progress data

User Header
***********

Displays:

- user initials
- username
- account status

.. code-block:: dart

    String _initialsFor(String username)

Dietary Preferences
*******************

A fixed list of options is shown using FilterChip.

.. code-block:: dart

    final List<String> dietaryOptions = [
        'Vegetarian',
        'Vegan',
        'Gluten free',
        'Dairy free',
    ];

Selections are stored in a Set<String>.

Diet Notes
**********

A TextEditingController stores user notes.

This is purely local state and not persisted.

Recipe Progress
***************

Mock data is used to simulate progress tracking.

.. code-block:: dart

    final List<_RecipeCompletion> _recipeCompletion = [...];

Includes:

- progress bar
- completed count
- summary metrics

Actions
*******

Save Changes:

- shows SnackBar
- no backend request

Sign Out:

.. code-block:: dart

    AuthService.clearSession();

    Navigator.pushNamedAndRemoveUntil(
        AppRoutes.login,
        (route) => false,
    );

Summary
*******

This page provides:

- profile UI
- preference selection
- mock progress tracking
- session logout
