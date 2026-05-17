=====================================================
Frontend Documentation - Cook App Flutter Application
=====================================================

Overview
========

This is a Flutter-based mobile application for managing cooking recipes. The app allows users to create accounts, log in, browse recipes, search for recipes, add new recipes with images and step-by-step instructions, and follow recipes with a step-by-step guided cooking mode.

The frontend is organized into several core modules:
- Core files (routing, themes, sessions, API responses)
- Authentication features (login, account creation)
- Recipe features (home page, adding recipes, viewing recipes, searching)
- Static reference pages (cooking techniques, account profile)
- Service layers for API communication

Tech Stack
----------
- Framework: Flutter (Dart)
- HTTP Client: http package
- Image Handling: image_picker package
- State Management: StatefulWidget pattern

Project Structure
-----------------

::

    frontend/
    |-- core/
    |   |-- main.dart              # Application entry point
    |   |-- routes.dart            # Navigation routing configuration
    |   |-- theme.dart             # App theme and styling
    |   |-- session.dart           # Global session state
    |   |-- api_response.dart      # API response wrapper
    |
    |-- features/
        |-- authen/
        |   |-- login_page.dart           # Login screen
        |   |-- create_account.dart       # Account creation screen
        |   |-- services/
        |       |-- account_services.dart # Authentication API calls
        |
        |-- recipe/
        |   |-- home_page.dart            # Main recipe listing page
        |
        |-- add_recipe/
        |   |-- add_recipe.dart           # Add new recipe form
        |   |-- services/
        |       |-- recipe_service.dart   # Recipe creation API calls
        |
        |-- search_recipe/
        |   |-- search_page.dart          # Recipe search interface
        |   |-- services/
        |       |-- search_recipe_service.dart # Search API calls
        |
        |-- view_recipe/
        |   |-- view_recipe.dart          # Step-by-step recipe viewer
        |   |-- services/
        |       |-- service_view_recipe.dart # Recipe viewing API calls
        |
        |-- techniques/
        |   |-- cooking_techniques_page.dart  # Static cooking techniques reference
        |
        |-- profile/
            |-- profile_page.dart         # Static user account profile page


Core Files
==========

main.dart - Application Entry Point
------------------------------------

The story of the application begins here. When the app launches, the main function runs first.

::

    void main(){
        runApp(const CookApp());
    }

This simple function starts the entire Flutter application by calling runApp with the CookApp widget.

The CookApp class is a StatelessWidget, which means it does not change once it's built. It extends StatelessWidget and has a constructor that accepts a super key for widget identification.

::

    class CookApp extends StatelessWidget {
        const CookApp({super.key});

Inside the build method, we create a MaterialApp widget. This is the root widget that sets up all the core app configurations:

::

    Widget build(BuildContext context) {
        return MaterialApp(
            title: "Cook App",
            theme: AppTheme.lightTheme, 
            initialRoute: AppRoutes.login,
            onGenerateRoute: AppRoutes.generateRoute
        );
    }

Let's break down each part:
- title: Sets the app name to "Cook App"
- theme: Uses a custom theme defined in AppTheme.lightTheme (we'll see this later)
- initialRoute: When the app first opens, it shows the login page
- onGenerateRoute: This tells Flutter to use AppRoutes.generateRoute to handle all navigation

The comment in the code mentions "basically redid main and moved everything out" - this means the developer organized the code by separating concerns. Instead of having everything in main.dart, they created namespaces (separate files) for themes and routes.


api_response.dart - API Response Wrapper
-----------------------------------------

This is a simple data class that standardizes how API responses are handled throughout the app.

::

    class ApiResponse {
      final int statusCode;
      final String response;

      ApiResponse({required this.statusCode, required this.response});
    }

The class has two properties:
- statusCode: The HTTP status code (like 200 for success, 404 for not found, 500 for server error)
- response: The actual response body as a string (usually JSON)

Both fields are marked as final, meaning they cannot be changed after the object is created. The required keyword means you must provide both values when creating an ApiResponse object.

This wrapper makes it easy to pass both the status code and response data together from service functions back to the UI layer.


session.dart - Global Session State
------------------------------------

This file manages the global user session state across the entire app.

::

    class Session {
      static int? userId;
    }

The Session class has one static property: userId. Let's understand what this means:

- static: This means userId belongs to the class itself, not to instances of the class. You access it as Session.userId from anywhere in the app.
- int?: The question mark means this is a nullable integer. It can hold a number or be null.
- When a user logs in, the app stores their user ID here
- When a user logs out, this gets set back to null

This is a simple global state management pattern. The static variable acts as a shared memory location that all parts of the app can read from and write to.


theme.dart - Application Theme
-------------------------------

This file defines the visual styling for the entire application.

::

    import 'package:flutter/material.dart';

    class AppTheme { 
        static ThemeData lightTheme = ThemeData(
            colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
            useMaterial3: true,
        );
    }

The AppTheme class is just a namespace (as the comment says, like in Rust where you'd have a module). It contains one static ThemeData object called lightTheme.

The theme configuration:
- colorScheme: Flutter generates a complete color palette from a single seed color (deepPurple). This creates a cohesive color scheme with primary colors, secondary colors, backgrounds, etc., all harmonizing with the seed color.
- useMaterial3: Enables Material Design 3, which is Google's latest design system with updated components and styling.

Because this is static, you can access it from anywhere as AppTheme.lightTheme, which is exactly what main.dart does.


routes.dart - Navigation Configuration
---------------------------------------

This file is the navigation hub of the entire application. It defines all the routes (screens) and handles how to navigate between them.

First, we import all the page widgets we'll need:

::

    import 'package:flutter/material.dart';
    import 'package:frontend/features/authen/create_account.dart';
    import 'package:frontend/features/recipe/home_page.dart';
    import 'package:frontend/features/authen/login_page.dart';
    import 'package:frontend/features/profile/profile_page.dart';
    import 'package:frontend/features/add_recipe/add_recipe.dart';
    import 'package:frontend/features/search_recipe/search_page.dart';
    import 'package:frontend/features/techniques/cooking_techniques_page.dart';
    import 'package:frontend/features/view_recipe/view_recipe.dart';

Next, the AppRoutes class defines all route names as constants. These are like named addresses for each screen:

::

    class AppRoutes {
      static const String login = "/login";
      static const String home = "/home";
      static const String profile = "/profile";
      static const String addRecipe = "/addRecipe";
      static const String createAccount = "/createAccount";
      static const String techniques = "/techniques";
      static const String search = "/search";
      static const String viewRecipe = "/view-recipe";

Using constants instead of string literals prevents typos and makes the code easier to maintain.

Now comes the most important part - the generateRoute function. This is called every time you navigate to a new screen:

::

    static Route<dynamic> generateRoute(RouteSettings settings) {

The function receives RouteSettings which contains information about the requested route, including the route name and any arguments passed to it.

The debugging section checks if the route name exists:

::

    if (settings.name == null) {
      print("is null");
    }
    if (settings.name!.isEmpty) {
      print("is empty");
    }
    print("${settings.name}");

The exclamation mark (!) after settings.name is the null assertion operator - it tells Dart "I know this isn't null, trust me."

Now we use a switch statement to match the route name and create the appropriate page:

**Login Route:**

::

    case login:
        return MaterialPageRoute(builder: (_) => LoginPage());

Simple - no arguments needed, just create and return the LoginPage.

**Home Route:**

::

    case home:
        final args = settings.arguments as Map<String, dynamic>;
        return MaterialPageRoute(
          builder: (_) => HomePage(
            username: args["username"],
            newAccount: args["newAccount"],
          ),
        );

This route is more complex because it needs arguments. The arguments are cast as a Map, then we extract the username and newAccount flag to pass to the HomePage constructor.

**Profile Route:**

::

    case profile:
        final args = settings.arguments as Map<String, dynamic>;
        return MaterialPageRoute(
          builder: (_) => AccountPage(
            username: args["username"],
            newAccount: args["newAccount"],
          ),
        );

Similar pattern to home - extracts user information from arguments.

**Simple Routes:**

::

    case addRecipe:
        return MaterialPageRoute(builder: (_) => AddRecipe());

    case createAccount:
        return MaterialPageRoute(builder: (_) => CreateAccount());

    case techniques:
        return MaterialPageRoute(builder: (_) => const CookingTechniquesPage());

    case search:
        return MaterialPageRoute(builder: (_) => const SearchPage());

These routes don't need any arguments, so they simply instantiate and return their respective pages.

**View Recipe Route:**

::

    case viewRecipe:
        final args = settings.arguments as Map<String, dynamic>;
        return MaterialPageRoute(
          builder: (_) => RecipePage(recipeId: args["recipeId"]),
        );

This route needs to know which recipe to display, so it extracts the recipeId from the arguments.

**Default Case (Error Handling):**

::

    default:
        return MaterialPageRoute(
          builder: (_) =>
              const Scaffold(body: Center(child: Text("error no route"))),
        );

If someone tries to navigate to a route that doesn't exist, show an error message instead of crashing.

The MaterialPageRoute wrapper for each case handles the actual page transition animations and the navigation stack management.


Authentication Features
=======================

account_services.dart - Authentication Service Layer
-----------------------------------------------------

This file handles all communication with the backend authentication API. It manages session cookies and provides methods for login and account creation.

The class starts with private static session management:

::

    class AuthService {
      static String? _sessionCookie;

      static String? get sessionCookie => _sessionCookie;

The _sessionCookie variable stores the authentication cookie received from the server. The underscore prefix makes it private. The getter allows other parts of the app to read the session cookie.

**Setting the Session:**

::

    static void setSessionCookie(String cookie) {
        _sessionCookie = cookie;
    }

This method stores the session cookie when the user logs in.

**Clearing the Session:**

::

    static void clearSession() {
        _sessionCookie = null;
    }

This method removes the session cookie when the user logs out.

**Checking Authentication Status:**

::

    static bool isAuthenticated() {
        return _sessionCookie != null;
    }

This returns true if there's an active session (cookie exists), false otherwise.

**Authenticate Method (Login):**

::

    static Future<ApiResponse> authenticate(
        String fname,
        String lname,
        String password,
    ) async {

This method sends login credentials to the server. It's async because it performs network operations.

::

    final response = await http.post(
      Uri.parse("http://localhost:5000/authenticate"),
      headers: {"Content-Type": "application/x-www-form-urlencoded"},
      body: {
        "user_fname": fname,
        "user_lname": lname,
        "user_password": password,
      },
    );

The http.post call sends a POST request to the /authenticate endpoint with the user's first name, last name, and password as form data.

After receiving the response, we check for success and extract the session cookie:

::

    if (response.statusCode == 200) {
      final cookies = response.headers["set-cookie"];
      if (cookies != null) {
        _sessionCookie = cookies.split(';')[0];
      }
    }

HTTP cookies often contain multiple parts separated by semicolons. We only want the first part (the actual cookie value), so we split on ';' and take index [0].

Finally, wrap the response in our ApiResponse object:

::

    return ApiResponse(
      statusCode: response.statusCode,
      response: response.body,
    );

**Create Account Method:**

::

    static Future<ApiResponse> createAccount(
        String fname,
        String lname,
        String email,
        String password,
    ) async {

This method is similar to authenticate but sends data to a different endpoint and includes an email field:

::

    final response = await http.post(
      Uri.parse("http://localhost:5000/create-account"),
      headers: {"Content-Type": "application/x-www-form-urlencoded"},
      body: {
        "user_fname": fname,
        "user_lname": lname,
        "user_email": email,
        "user_password": password,
      },
    );

It returns the status code and response body wrapped in an ApiResponse object.


login_page.dart - Login Screen
-------------------------------

This file implements the login screen where users enter their credentials.

First, we have a simple User class to hold the text controllers:

::

    class User {
      final TextEditingController _userFnameController = TextEditingController();
      final TextEditingController _userLnameController = TextEditingController();
      final TextEditingController _passwordController = TextEditingController();
    }

TextEditingController objects are Flutter's way of managing text input. They let you read the current value, set values programmatically, and listen for changes.

The LoginPage is a StatefulWidget because it needs to change (show loading states, validation errors, etc.):

::

    class LoginPage extends StatefulWidget {
      const LoginPage({super.key});

      @override
      _LoginPageState createState() => _LoginPageState();
    }

The state class holds the actual logic:

::

    class _LoginPageState extends State<LoginPage> {
      final _formKey = GlobalKey<FormState>();
      final User user = User();

The _formKey is used to validate all form fields at once and trigger validation messages.

**Showing Messages to Users:**

::

    void showMsg(BuildContext context, String msg, int time) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text("${msg}"),
            duration: Duration(seconds: time),
          ),
        );
    }

This helper function displays a temporary message at the bottom of the screen using a SnackBar.

**Sending Login Data to Server:**

::

    Future<void> _sendUser(userFname, userLname, password, newAccount) async {

This async function handles the entire login process. Let's walk through it:

::

    try {
      final data = await AuthService.authenticate(
        userFname,
        userLname,
        password,
      );

Call the authenticate method from AuthService and wait for the response.

::

      final jsonResp = jsonDecode(data.response);

Parse the JSON response body into a Dart object.

::

      if (!mounted) return;

This is a critical safety check. If the widget was removed from the screen while waiting for the network response, we don't want to try updating it. The mounted property tells us if the widget is still active.

::

      if (data.statusCode == 200) {
        String username = "$userFname $userLname";
        Session.userId = jsonResp["user"]["user_id"];

If login was successful (status 200), create the username string and save the user ID to the global Session.

::

        Navigator.pushReplacementNamed(
          context,
          AppRoutes.home,
          arguments: {"username": username, "newAccount": newAccount},
        );

Navigate to the home page, replacing the login page in the navigation stack (so the back button won't go back to login). Pass the username and newAccount flag as arguments.

::

        showMsg(context, "logged in", 2);

Show a success message for 2 seconds.

If login fails:

::

      } else {
        showMsg(context, "invalid credientials", 2);
      }

Show an error message.

The catch block handles any network errors or exceptions:

::

    } catch (e) {
      print(e);
      showMsg(context, "server error", 2);
    }

**Handling Authentication Submission:**

::

    void _handleAuth(bool newAccount) async {
        String userFname = user._userFnameController.text.trim();
        String userLname = user._userLnameController.text.trim();
        String userPassword = user._passwordController.text.trim();

        await _sendUser(userFname, userLname, userPassword, newAccount);
    }

This method extracts the text from the controllers (trimming whitespace) and calls _sendUser.

**Cleanup:**

::

    @override
    void dispose() {
        user._userFnameController.dispose();
        user._userLnameController.dispose();
        user._passwordController.dispose();
        super.dispose();
    }

When the widget is destroyed, we must dispose of the controllers to free up memory. This prevents memory leaks.

**Building the UI:**

::

    @override
    Widget build(BuildContext context) {
        return Scaffold(
          appBar: AppBar(
            title: Text("Login"),
            automaticallyImplyLeading: false,
            centerTitle: true,
          ),

The Scaffold provides the basic page structure with an AppBar. The automaticallyImplyLeading: false removes the back button since login is typically the first screen.

The body contains a centered, padded Form:

::

    body: Center(
        child: Padding(
          padding: EdgeInsets.all(20),
          child: Form(
            key: _formKey,

**First Name Field:**

::

    TextFormField(
      controller: user._userFnameController,
      decoration: InputDecoration(
        labelText: "User first name",
        border: OutlineInputBorder(),
      ),
      validator: (value) {
        if (value == null || value.trim().isEmpty) {
          return "please enter a valid name";
        }
        return null;
      },
    ),

The TextFormField is connected to the controller and has a validator. If the validator returns a string, that string is shown as an error message. If it returns null, the field is valid.

**Last Name and Password Fields:**

The last name field follows the same pattern as the first name. The password field is identical except it has obscureText: true to hide the password as dots.

**Login Button:**

::

    ElevatedButton(
      onPressed: () {
        if (_formKey.currentState!.validate()) {
          _handleAuth(false);
        }
      },
      child: Text("Login"),
    ),

When pressed, validate all fields. If all validators return null (all fields are valid), call _handleAuth with newAccount = false.

**Create Account Button:**

::

    ElevatedButton(
      onPressed: () async {
        final result = await Navigator.pushNamed(
          context,
          AppRoutes.createAccount,
        );

Navigate to the create account page and wait for it to return a result.

::

        if (result is Map && result['username'] != null) {
          if (result['message'] != null) {
            if (context.mounted) {
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(
                  content: Text(result['message'].toString()),
                ),
              );
            }
            await Future.delayed(const Duration(milliseconds: 300));
          }

If the create account page returned data, show a message briefly (300 milliseconds delay ensures the message is visible).

::

          if (context.mounted) {
            Navigator.pushReplacementNamed(
              context,
              AppRoutes.home,
              arguments: {
                'username': result['username'],
                'newAccount': result['newAccount'] ?? true,
              },
            );
          }
        }
      },
      child: const Text("Create Account"),
    ),

Navigate to the home page with the new user's information.


create_account.dart - Account Creation Screen
----------------------------------------------

This screen allows new users to create an account.

The CreateAccount widget is stateful:

::

    class CreateAccount extends StatefulWidget {
      const CreateAccount({super.key});

      @override
      State<CreateAccount> createState() => _CreateAccountState();
    }

The state class manages all the form controllers and loading state:

::

    class _CreateAccountState extends State<CreateAccount> {
      final _formKey = GlobalKey<FormState>();
      final TextEditingController _firstNameController = TextEditingController();
      final TextEditingController _lastNameController = TextEditingController();
      final TextEditingController _emailController = TextEditingController();
      final TextEditingController _passwordController = TextEditingController();
      final TextEditingController _confirmController = TextEditingController();

      bool _loading = false;

We have controllers for five fields: first name, last name, email, password, and password confirmation. The _loading flag tracks whether we're currently creating the account.

**Cleanup:**

::

    @override
    void dispose() {
        _firstNameController.dispose();
        _lastNameController.dispose();
        _emailController.dispose();
        _passwordController.dispose();
        _confirmController.dispose();
        super.dispose();
    }

All five controllers must be disposed when the widget is destroyed.

**Submit Logic:**

::

    Future<void> _submit() async {
        if (!_formKey.currentState!.validate()) return;

First, validate all fields. If any validator fails, stop here.

::

        final first = _firstNameController.text.trim();
        final last = _lastNameController.text.trim();
        final email = _emailController.text.trim();
        final password = _passwordController.text;

Extract and trim the values from the controllers.

::

        setState(() => _loading = true);

Set loading to true, which will show a loading spinner on the button.

::

        try {
          final resp = await AuthService.createAccount(
            first,
            last,
            email,
            password,
          );

Call the create account API.

::

          if (resp.statusCode == 200 || resp.statusCode == 201) {
            if (!mounted) return;
            ScaffoldMessenger.of(context).showSnackBar(
              const SnackBar(content: Text('Account created successfully')),
            );
            Navigator.of(context).maybePop();

If successful (200 OK or 201 Created), show a success message and navigate back to the login page.

::

          } else {
            String msg = 'Failed to create account';
            try {
              final body = jsonDecode(resp.response);
              if (body is Map && body['message'] != null) msg = body['message'];
            } catch (_) {}

If it fails, try to extract a more specific error message from the response JSON. If that fails, use the generic message.

::

            if (!mounted) return;
            ScaffoldMessenger.of(
              context,
            ).showSnackBar(SnackBar(content: Text(msg)));
          }

Show the error message to the user.

::

        } catch (e) {
          ScaffoldMessenger.of(
            context,
          ).showSnackBar(SnackBar(content: Text('Error: $e')));
        } finally {
          setState(() => _loading = false);
        }

Catch any exceptions and show them. The finally block ensures we always set loading back to false.

**Validators:**

::

    String? _validateName(String? v) {
        if (v == null || v.trim().isEmpty) return 'Required';
        if (v.trim().length < 2) return 'Too short';
        return null;
    }

Names must exist and be at least 2 characters.

::

    String? _validateEmail(String? v) {
        if (v == null || v.trim().isEmpty) return 'Email required';
        if (!v.contains('@')) return 'Enter a valid email';
        return null;
    }

Email must exist and contain an @ symbol (basic validation).

::

    String? _validatePassword(String? v) {
        if (v == null || v.isEmpty) return 'Password required';
        if (v.length < 6) return 'Password must be 6+ characters';
        return null;
    }

Password must be at least 6 characters.

**Building the UI:**

::

    @override
    Widget build(BuildContext context) {
        return Scaffold(
          appBar: AppBar(title: const Text('Create Account')),
          body: SingleChildScrollView(
            padding: const EdgeInsets.all(16),

SingleChildScrollView allows the form to scroll if the keyboard covers part of the screen.

The form contains five TextFormFields, each with appropriate decorations and validators:

::

    TextFormField(
      controller: _firstNameController,
      decoration: const InputDecoration(labelText: 'First name'),
      validator: _validateName,
    ),

The password confirmation field has a special validator:

::

    TextFormField(
      controller: _confirmController,
      decoration: const InputDecoration(
        labelText: 'Confirm password',
      ),
      obscureText: true,
      validator: (v) {
        if (v == null || v.isEmpty) return 'Confirm password';
        if (v != _passwordController.text) {
          return 'Passwords do not match';
        }
        return null;
      },
    ),

It checks that the confirmation matches the password field.

**Submit Button:**

::

    ElevatedButton(
      onPressed: _loading ? null : _submit,
      child: _loading
          ? const SizedBox(
              height: 16,
              width: 16,
              child: CircularProgressIndicator(strokeWidth: 2),
            )
          : const Text('Create account'),
    ),

If loading, disable the button (onPressed: null) and show a spinner. Otherwise, enable it and show the text.


Recipe Features
===============

home_page.dart - Main Recipe Listing Page
------------------------------------------

This is the main screen users see after logging in. It displays a list of recipes and provides quick navigation buttons.

The HomePage is a StatefulWidget that accepts username and newAccount parameters:

::

    class HomePage extends StatefulWidget {
      const HomePage({super.key, this.username = "guest", this.newAccount = false});

      final String username;
      final bool newAccount;

      @override
      State<HomePage> createState() => _MyHomePageState();
    }

**State Class:**

::

    class _MyHomePageState extends State<HomePage> {
      List<dynamic> recipes = [];
      bool isLoading = true;

The state tracks a list of recipes and whether we're currently loading data.

**Initialization:**

::

    @override
    void initState() {
        super.initState();
        _loadRecipes();
    }

When the page first loads, immediately fetch the recipes.

**Loading Recipes:**

::

    Future<void> _loadRecipes() async {
        try {
          final response = await http.get(
            Uri.parse("http://localhost:5000/get-recipes"),
          );

Make a GET request to fetch all recipes.

::

          if (response.statusCode == 200) {
            final data = jsonDecode(response.body);
            setState(() {
              recipes = data["recipes"] ?? [];
              isLoading = false;
            });
          }

If successful, parse the JSON and update the recipes list. The ?? [] provides an empty list if "recipes" is null.

::

        } catch (e) {
          setState(() {
            isLoading = false;
          });
          debugPrint("error :$e");
        }

On error, stop loading and print the error.

**Building the UI:**

::

    @override
    Widget build(BuildContext context) {
        return Scaffold(
          appBar: AppBar(
            automaticallyImplyLeading: false,
            backgroundColor: Theme.of(context).colorScheme.inversePrimary,
            title: Text(
              "welcome, ${widget.username}",
            ),

The AppBar welcomes the user by name. The widget.username accesses the parameter passed to the HomePage.

**AppBar Actions:**

::

    actions: [
      IconButton(
        onPressed: () {
          Navigator.pushNamed(
            context,
            AppRoutes.profile,
            arguments: {
              "username": widget.username,
              "newAccount": widget.newAccount,
            },
          );
        },
        icon: Icon(Icons.account_box_rounded),
      ),

Profile button navigates to the profile page, passing username and newAccount.

::

      IconButton(
        onPressed: () {
          Navigator.pushNamed(context, AppRoutes.techniques);
        },
        icon: Icon(Icons.kitchen),
      ),
    ],

Cooking Techniques button navigates to the static cooking techniques reference page.

**Body Content:**

::

    body: isLoading
        ? const Center(child: CircularProgressIndicator())
        : RefreshIndicator(

If loading, show a spinner. Otherwise, wrap the content in a RefreshIndicator which allows pull-to-refresh functionality.

::

    onRefresh: _loadRecipes,

When the user pulls down to refresh, call _loadRecipes again.

::

    child: SingleChildScrollView(
      physics: const AlwaysScrollableScrollPhysics(),

SingleChildScrollView makes the content scrollable. AlwaysScrollableScrollPhysics ensures the scroll behavior works even if the content is shorter than the screen (needed for pull-to-refresh).

**Quick Action Buttons:**

::

    Padding(
      padding: const EdgeInsetsGeometry.all(16),
      child: Row(
        children: [
          Expanded(
            child: _buildQuickAction(
              icon: Icons.search,
              label: "search",
              onTap: () {
                Navigator.pushNamed(context, AppRoutes.search);
              },
            ),
          ),

The Row contains three quick action buttons: Search, Add Recipe, and Techniques. Each is wrapped in Expanded to share the available width equally.

**Recipe List Title:**

::

    const Padding(
      padding: EdgeInsets.symmetric(horizontal: 16),
      child: Text(
        "recipes",
        style: TextStyle(
          fontSize: 20,
          fontWeight: FontWeight.bold,
        ),
      ),
    ),

A simple header for the recipe list section.

**Recipe List or Empty State:**

::

    recipes.isEmpty
        ? const Padding(
            padding: EdgeInsets.all(40),
            child: Center(
              child: Text(
                "no recipes yet add your first :)",
                textAlign: TextAlign.center,
                style: TextStyle(
                  fontSize: 16,
                  color: Colors.grey,
                ),
              ),
            ),
          )

If there are no recipes, show an encouraging message.

::

        : ListView.builder(
            shrinkWrap: true,
            physics: const NeverScrollableScrollPhysics(),
            padding: const EdgeInsets.symmetric(horizontal: 16),
            itemCount: recipes.length,
            itemBuilder: (context, index) {
              final recipe = recipes[index];
              return _buildRecipeCard(recipe);
            },
          ),

Otherwise, use ListView.builder to efficiently create recipe cards. The shrinkWrap: true makes the ListView take only as much space as needed. NeverScrollableScrollPhysics disables scrolling on the ListView itself (the parent SingleChildScrollView handles scrolling).

**Quick Action Widget Builder:**

::

    Widget _buildQuickAction({
        required IconData icon,
        required String label,
        required VoidCallback onTap,
    }) {

This helper method creates the quick action buttons. It uses named required parameters.

::

    return InkWell(
      onTap: onTap,
      borderRadius: BorderRadius.circular(12),

InkWell provides the tap ripple effect. The borderRadius matches the container's border radius.

::

      child: Container(
        padding: const EdgeInsets.all(16),
        decoration: BoxDecoration(
          border: Border.all(color: Colors.grey.shade300),
          borderRadius: BorderRadius.circular(12),
        ),

The Container has padding, a border, and rounded corners.

::

        child: Column(
          children: [
            Icon(icon, size: 32),
            const SizedBox(height: 8),
            Text(
              label,
              textAlign: TextAlign.center,
              style: const TextStyle(fontSize: 12),
            ),
          ],
        ),
      ),
    );

The icon sits above the label text.

**Recipe Card Builder:**

::

    Widget _buildRecipeCard(Map<String, dynamic> recipe) {
        return Card(
          margin: const EdgeInsets.only(bottom: 12),
          child: InkWell(
            onTap: () {
              Navigator.pushNamed(
                context,
                AppRoutes.viewRecipe,
                arguments: {"recipeId": recipe["recipe_id"]},
              );
            },

When tapped, navigate to the view recipe page with the recipe's ID.

::

            child: Padding(
              padding: const EdgeInsets.all(12),
              child: Row(
                children: [
                  ClipRRect(
                    borderRadius: BorderRadius.circular(8),
                    child: Image.network(
                      "http://localhost:5000/recipe-image/${recipe["recipe_id"]}",
                      width: 80,
                      height: 80,
                      fit: BoxFit.cover,

Load the recipe image from the server using the recipe ID.

::

                      errorBuilder: (context, error, stackTrace) {
                        return Container(
                          width: 80,
                          height: 80,
                          color: Colors.grey[200],
                          child: const Icon(Icons.restaurant, size: 40),
                        );
                      },
                    ),
                  ),

If the image fails to load, show a placeholder with a restaurant icon.

::

                  const SizedBox(width: 16),

                  Expanded(
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          recipe["recipe_title"] ?? "untitled",
                          style: TextStyle(
                            fontSize: 16,
                            fontWeight: FontWeight.bold,
                          ),
                        ),

Display the recipe title, or "untitled" if missing.

::

                        const SizedBox(height: 4),
                        if (recipe["recipe_difficulty"] != null)
                          Text(
                            "difficulty: ${recipe["recipe_difficulty"]}",
                            style: TextStyle(fontSize: 14, color: Colors.grey[600]),
                          ),
                      ],
                    ),
                  ),
                  const Icon(Icons.chevron_right),

Show a right chevron to indicate the card is tappable.


add_recipe.dart - Recipe Creation Form
---------------------------------------

This is the most complex screen in the application. It allows users to create recipes with ingredients, steps, sub-steps, images, and time durations.

**Data Classes:**

First, we define classes to organize the complex form data:

::

    class Ingredient {
      final TextEditingController name = TextEditingController();
      final TextEditingController amount = TextEditingController();
      final TextEditingController calories = TextEditingController();
      String? amountUnits;
    }

Each Ingredient holds controllers for name, amount, and calories, plus a nullable string for units.

::

    class SubStep {
      final TextEditingController subStep = TextEditingController();
      final TextEditingController subMinutes = TextEditingController();
      final TextEditingController subHours = TextEditingController();
      File? image;
    }

SubSteps have a description, duration (hours and minutes), and an optional image.

::

    class StepItem {
      final TextEditingController controller = TextEditingController();
      final List<SubStep> subSteps = [];
      final TextEditingController minutes = TextEditingController();
      final TextEditingController hours = TextEditingController();
      File? image;
    }

Each StepItem has a description, duration, optional image, and a list of SubSteps.

**State Class:**

::

    class _AddRecipeState extends State<AddRecipe> {
      final TextEditingController _recipeNameController = TextEditingController();
      final TextEditingController _hoursController = TextEditingController();
      final TextEditingController _minutesController = TextEditingController();
      final _formKey = GlobalKey<FormState>();
      final List<Ingredient> _ingredients = [];
      final List<StepItem> _steps = [];
      String? _difficultySelector;
      File? _recipeMainImage;
      final ImagePicker _picker = ImagePicker();

The state holds controllers for the recipe name and total time, lists for ingredients and steps, a difficulty selector, the main recipe image, and an ImagePicker instance.

**Initialization:**

::

    @override
    void initState() {
        super.initState();
        for (int i = 0; i < 3; i++) {
          _ingredients.add(Ingredient());
          _steps.add(StepItem());
        }
    }

Start with 3 empty ingredients and 3 empty steps. Users can add more later.

**Cleanup:**

::

    @override
    void dispose() {
        _recipeNameController.dispose();
        _hoursController.dispose();
        _minutesController.dispose();

        for (var cont in _ingredients) {
          cont.name.dispose();
          cont.amount.dispose();
          cont.calories.dispose();
        }

        for (var cont in _steps) {
          cont.controller.dispose();
          cont.hours.dispose();
          cont.minutes.dispose();

          for (var subCont in cont.subSteps) {
            subCont.subStep.dispose();
            subCont.subMinutes.dispose();
            subCont.subHours.dispose();
          }
        }

        super.dispose();
    }

We must dispose all controllers, including nested ones in ingredients, steps, and sub-steps.

**Image Picking:**

::

    Future<File?> _pickImage() async {
        try {
          final XFile? pickedFile = await _picker.pickImage(
            source: ImageSource.gallery,
            maxWidth: 1800,
            maxHeight: 1800,
            imageQuality: 85,
          );

Use the image picker to let the user select an image from their gallery. Images are constrained to 1800x1800 pixels and 85% quality to reduce file size.

::

          if (pickedFile != null) {
            return File(pickedFile.path);
          }
          return null;

Convert the XFile to a File if successful.

::

        } catch (e) {
          if (!mounted) return null;
          ScaffoldMessenger.of(
            context,
          ).showSnackBar(SnackBar(content: Text("Error picking image: $e")));
          return null;
        }

Handle errors by showing a message.

**Image Picker Widget Builder:**

::

    Widget buildImagePicker({
        required File? image,
        required VoidCallback onPick,
        required VoidCallback onRemove,
        String label = "Add image",
    }) {

This helper builds the UI for image selection.

::

    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        const SizedBox(height: 10),
        if (image != null)
          Stack(
            children: [
              ClipRRect(
                borderRadius: BorderRadius.circular(8),
                child: Image.file(
                  image,
                  height: 150,
                  width: double.infinity,
                  fit: BoxFit.cover,
                ),
              ),

If an image is selected, display it with rounded corners.

::

              Positioned(
                top: 8,
                right: 8,
                child: IconButton(
                  icon: const Icon(Icons.close, color: Colors.white),
                  style: IconButton.styleFrom(backgroundColor: Colors.black54),
                  onPressed: onRemove,
                ),
              ),
            ],
          )

Add a close button in the top-right corner to remove the image.

::

        else
          OutlinedButton.icon(
            onPressed: onPick,
            icon: const Icon(Icons.add_photo_alternate),
            label: Text(label),
          ),

If no image is selected, show a button to pick one.

**Duration Fields Builder:**

::

    Widget buildDurationFields(
        TextEditingController hours,
        TextEditingController minutes,
    ) {

This creates side-by-side hour and minute input fields.

::

    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        const SizedBox(height: 20),
        Row(
          children: [
            Expanded(
              child: TextFormField(
                controller: hours,
                keyboardType: TextInputType.number,
                inputFormatters: [FilteringTextInputFormatter.digitsOnly],

The inputFormatters restrict input to digits only.

::

                decoration: const InputDecoration(
                  labelText: "hours",
                  border: OutlineInputBorder(),
                ),
              ),
            ),
            const SizedBox(width: 20),
            Expanded(
              child: TextFormField(
                controller: minutes,
                keyboardType: TextInputType.number,
                inputFormatters: [FilteringTextInputFormatter.digitsOnly],
                decoration: const InputDecoration(
                  labelText: "minutes",
                  border: OutlineInputBorder(),
                ),
              ),
            ),
          ],
        ),
      ],
    );

The Row with two Expanded children splits the width evenly.

**Text Input Field Builder:**

::

    Widget buildTextInputField({
        required TextEditingController controller,
        required String label,
        TextInputType keyboardType = TextInputType.text,
        List<TextInputFormatter>? inputFormatters,
        String? Function(String?)? validator,
    }) {

This is a reusable text field builder with customizable keyboard type, input formatters, and validator.

**Duration to ISO 8601 Conversion:**

::

    String _durationToISO(int hours, int minutes) {
        String result = "PT";
        if (hours > 0) {
          result += "${hours}H";
        }
        if (minutes > 0) {
          result += "${minutes}M";
        }
        if (result == "PT") {
          result += "0M";
        }
        return result;
    }

This converts hours and minutes to ISO 8601 duration format. For example:
- 2 hours, 30 minutes becomes "PT2H30M"
- 0 hours, 45 minutes becomes "PT45M"
- 0 hours, 0 minutes becomes "PT0M"

**Save Recipe Logic:**

::

    Future<void> _saveRecipe() async {
        if (_formKey.currentState!.validate()) {

First, validate all form fields.

::

          final name = _recipeNameController.text.trim();
          int hour = int.tryParse(_hoursController.text.trim()) ?? 0;
          int minutes = int.tryParse(_minutesController.text.trim()) ?? 0;
          final difficulty = _difficultySelector;

Extract the basic recipe information.

::

          final ingredients = _ingredients.map((ingredient) {
            return {
              "ingredient-name": ingredient.name.text.trim(),
              "ingredient-amount": ingredient.amount.text.trim(),
              "ingredient-units": ingredient.amountUnits,
              "ingredient-calories": ingredient.calories.text.trim(),
            };
          }).toList();

Transform the list of Ingredient objects into a list of maps (dictionaries) for JSON serialization.

::

          final steps = [];
          int flatStepIndex = 0;

          for (var step in _steps) {
            flatStepIndex++;
            steps.add({
              "step-index": flatStepIndex,
              "step-description": step.controller.text.trim(),
              "step-image": step.image?.path,
              "step-duration": _durationToISO(
                int.tryParse(step.hours.text.trim()) ?? 0,
                int.tryParse(step.minutes.text.trim()) ?? 0,
              ),
            });

For each main step, add it to the steps list with a sequential index, description, optional image path, and duration.

::

            for (var subStep in step.subSteps) {
              flatStepIndex++;
              steps.add({
                "step-index": flatStepIndex,
                "step-description": subStep.subStep.text.trim(),
                "step-image": subStep.image?.path,
                "step-duration": _durationToISO(
                  int.tryParse(subStep.subHours.text.trim()) ?? 0,
                  int.tryParse(subStep.subMinutes.text.trim()) ?? 0,
                ),
              });
            }
          }

For each sub-step, add it to the steps list with its own sequential index. This "flattens" the hierarchical step structure into a linear sequence.

::

          final time = _durationToISO(hour, minutes);

Convert the total recipe time to ISO format.

::

          try {
            final success = await RecipeService.addRecipe(
              name,
              ingredients,
              steps,
              time,
              difficulty!,
              _recipeMainImage?.path,
            );

Call the RecipeService to send the data to the server.

::

            if (!mounted) return;
            if (success) {
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text("recipe added successfully")),
              );
              Navigator.of(context).pop();
            }

If successful, show a success message and go back to the previous page.

::

          } catch (e) {
            if (!mounted) return;
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text("failed to add recipe : $e")),
            );
          }

Handle errors.

**Building the UI:**

The build method creates a very long scrollable form. Let's walk through the key sections:

::

    return Scaffold(
      appBar: AppBar(title: const Text("Add Recipe")),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(20),
        child: Form(
          key: _formKey,
          child: Column(

The form is a single Column with all fields stacked vertically.

**Recipe Name Field:**

::

    buildTextInputField(
      controller: _recipeNameController,
      label: "enter a recipe name:",
      validator: (value) {
        if (value == null || value.trim().isEmpty) {
          return "please enter a valid name";
        }
        return null;
      },
    ),

**Main Recipe Image:**

::

    buildImagePicker(
      image: _recipeMainImage,
      onPick: () async {
        final image = await _pickImage();
        if (image != null) {
          setState(() {
            _recipeMainImage = image;
          });
        }
      },
      onRemove: () {
        setState(() {
          _recipeMainImage = null;
        });
      },
      label: "Add Main Recipe Image",
    ),

**Recipe Time:**

::

    buildDurationFields(_hoursController, _minutesController),

**Difficulty Selector:**

::

    DropdownButtonFormField<String>(
      decoration: const InputDecoration(
        labelText: "Select Difficulty",
        border: OutlineInputBorder(),
      ),
      items: ["easy", "medium", "hard"]
          .map((item) => DropdownMenuItem(
                value: item,
                child: Text(item),
              ))
          .toList(),
      onChanged: (value) {
        setState(() {
          _difficultySelector = value;
        });
      },
      validator: (value) {
        if (value == null || value.isEmpty) {
          return "Please Select a Difficulty";
        }
        return null;
      },
    ),

A dropdown menu with three difficulty options.

**Ingredients Section:**

::

    const Text(
      "Enter ingredients",
      style: TextStyle(fontWeight: FontWeight.bold),
    ),

A section header.

::

    ..._ingredients.asMap().entries.map((entry) {
      int index = entry.key;
      Ingredient controller = entry.value;

The spread operator (...) unpacks the mapped widgets into the parent Column. We use asMap().entries to get both the index and the Ingredient object.

::

      return Padding(
        padding: const EdgeInsets.only(bottom: 20),
        child: Row(
          children: [
            Expanded(
              flex: 3,
              child: buildTextInputField(
                controller: controller.name,
                label: "enter and ingredient:",

The ingredient name field takes 3 parts of the available width.

::

            Expanded(
              flex: 2,
              child: buildTextInputField(
                controller: controller.amount,
                inputFormatters: [FilteringTextInputFormatter.digitsOnly],
                keyboardType: TextInputType.number,
                label: "enter a ingredient quantity",

The amount field takes 2 parts.

::

            Expanded(
              flex: 2,
              child: DropdownButtonFormField<String>(
                initialValue: controller.amountUnits,
                decoration: const InputDecoration(
                  labelText: "select Units",
                  border: OutlineInputBorder(),
                ),
                items: ["g", "KG", "ml", "L"]
                    .map((item) => DropdownMenuItem(
                          value: item,
                          child: Text(item),
                        ))
                    .toList(),
                onChanged: (value) {
                  setState(() {
                    controller.amountUnits = value;
                  });
                },

Units dropdown with common measurement units.

::

            Expanded(
              flex: 2,
              child: buildTextInputField(
                controller: controller.calories,
                inputFormatters: [FilteringTextInputFormatter.digitsOnly],
                label: "ingredient calories",

Calories field.

::

    ElevatedButton(
      onPressed: () {
        setState(() {
          _ingredients.add(Ingredient());
        });
      },
      child: const Text("Add Ingredient"),
    ),

Button to add another ingredient to the list.

**Steps Section:**

::

    const Text(
      "Enter Steps (in order)",
      style: TextStyle(fontWeight: FontWeight.bold),
    ),

Section header for steps.

::

    ..._steps.asMap().entries.map((entry) {
      int stepIndex = entry.key;
      StepItem step = entry.value;

      return Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Padding(
            padding: const EdgeInsets.only(bottom: 20),
            child: buildTextInputField(
              controller: step.controller,
              label: " Step ${stepIndex + 1}:",

Each step shows its number (stepIndex + 1 because arrays are 0-indexed).

::

          buildImagePicker(
            image: step.image,
            onPick: () async {
              final image = await _pickImage();
              if (image != null) {
                setState(() {
                  step.image = image;
                });
              }
            },
            onRemove: () {
              setState(() {
                step.image = null;
              });
            },
            label: "Add step ${stepIndex + 1} image",
          ),

Image picker for the step.

::

          buildDurationFields(step.hours, step.minutes),

Duration fields for the step.

**Sub-Steps:**

::

          ...step.subSteps.asMap().entries.map((subEntry) {
            int subIndex = subEntry.key;
            SubStep subStep = subEntry.value;
            return Padding(
              padding: const EdgeInsets.only(left: 20, bottom: 20),

Sub-steps are indented 20 pixels to show they're nested.

::

              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  buildTextInputField(
                    controller: subStep.subStep,
                    label: "Sub Step ${stepIndex + 1}.${subIndex + 1}:",

Sub-steps are numbered like "1.1", "1.2", "2.1", etc.

::

                  buildImagePicker(
                    image: subStep.image,
                    onPick: () async {
                      final image = await _pickImage();
                      if (image != null) {
                        setState(() {
                          subStep.image = image;
                        });
                      }
                    },

Sub-step image picker.

::

                  buildDurationFields(
                    subStep.subHours,
                    subStep.subMinutes,
                  ),

Sub-step duration fields.

::

          ElevatedButton(
            onPressed: () {
              setState(() {
                step.subSteps.add(SubStep());
              });
            },
            child: const Text("add sub step"),
          ),

Button to add a sub-step to this step.

::

    ElevatedButton(
      onPressed: () {
        setState(() {
          _steps.add(StepItem());
        });
      },
      child: const Text("add Step"),
    ),

Button to add a new main step.

**Save Button:**

::

    Center(
      child: ElevatedButton(
        onPressed: _saveRecipe,
        child: const Text("Save Recipe"),
      ),
    ),

The final save button at the bottom of the form.


recipe_service.dart - Recipe API Service
-----------------------------------------

This service handles uploading recipes to the server, including images.

::

    import 'dart:convert';
    import 'dart:io';
    import 'package:http/http.dart' as http;
    import 'package:frontend/features/authen/services/account_services.dart';

We need dart:io for File handling and dart:convert for JSON encoding.

**Add Recipe Method:**

::

    class RecipeService {
      static Future<bool> addRecipe(
        String name,
        List ingredients,
        List steps,
        String time,
        String difficulty,
        String? mainImage,
      ) async {

This method takes all the recipe data and uploads it to the server.

::

    try {
      var request = http.MultipartRequest(
        'POST',
        Uri.parse("http://localhost:5000/add-recipe"),
      );

We use MultipartRequest instead of a normal POST because we need to upload files (images) along with the text data.

::

      if (AuthService.sessionCookie != null) {
        request.headers["Cookie"] = AuthService.sessionCookie!;
      }

Add the session cookie to authenticate the request.

::

      request.fields['recipe-title'] = name;
      request.fields['recipe-ingredients'] = jsonEncode(ingredients);
      request.fields['recipe-steps'] = jsonEncode(steps);
      request.fields['recipe-time'] = time;
      request.fields['recipe-difficulty'] = difficulty;

Add all the text fields. The ingredients and steps lists are converted to JSON strings.

**Main Recipe Image:**

::

      if (mainImage != null && mainImage.isNotEmpty) {
        File imageFile = File(mainImage);
        if (await imageFile.exists()) {
          request.files.add(
            await http.MultipartFile.fromPath("recipe-main-image", mainImage),
          );
        }
      }

If a main image path was provided, check if the file exists, then add it to the request.

**Step Images:**

::

      for (var i = 0; i < steps.length; i++) {
        var step = steps[i];
        if (step['step-image'] != null && step['step-image'].isNotEmpty) {
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
      }

Loop through all steps and add their images. The field name includes the step index (like "step-image-1", "step-image-2") so the backend can match images to steps.

**Sending the Request:**

::

      var streamedResponse = await request.send();
      var response = await http.Response.fromStream(streamedResponse);

Send the multipart request and convert the streamed response to a regular Response object.

::

      if (response.statusCode == 200 || response.statusCode == 201) {
        return true;
      } else {
        throw Exception(
          "Failed to add recipe: ${response.statusCode} - ${response.body}",
        );
      }

Return true on success, throw an exception on failure.

::

    } catch (e) {
      throw Exception("failed to add recipe : $e");
    }

Catch and re-throw any errors.


Search Features
===============

search_page.dart - Recipe Search Interface
-------------------------------------------

This screen allows users to search for recipes by name.

::

    class SearchPage extends StatefulWidget {
      const SearchPage({super.key});

      @override
      State<SearchPage> createState() => _SearchPageState();
    }

**State Class:**

::

    class _SearchPageState extends State<SearchPage> {
      String query = "";
      List<dynamic> results = [];
      String? error;
      bool isLoading = false;

The state tracks the search query, results, any error message, and loading state.

**Search Function:**

::

    Future<void> _search(String name) async {
        if (name.isEmpty) {
          return;
        }

Don't search if the query is empty.

::

        setState(() {
          isLoading = true;
        });

Show loading indicator.

::

        final apiResponse = await SearchService.searchRecipe(name);

        if (apiResponse.statusCode == 200) {
          final data = jsonDecode(apiResponse.response);

          setState(() {
            results = data["recipes"];
            isLoading = false;
          });
        }

Parse the response and update the results.

**Building the UI:**

::

    @override
    Widget build(BuildContext context) {
        return Scaffold(
          appBar: AppBar(title: const Text("Search Recipes")),
          body: Padding(
            padding: const EdgeInsets.all(16.0),
            child: Column(
              children: [
                TextField(
                  decoration: const InputDecoration(
                    labelText: "Search recipes",
                    border: OutlineInputBorder(),
                    prefixIcon: Icon(Icons.search),
                  ),
                  onChanged: (value) {
                    setState(() {
                      query = value;
                    });
                    _search(value);
                  },
                ),

The TextField calls _search on every change, providing real-time search results.

::

                const SizedBox(height: 20),
                Expanded(child: _buildResults()),

The results fill the remaining screen space.

**Results Builder:**

::

    Widget _buildResults() {
        if (query.isEmpty) {
          return const Text("type to search something");
        }

Show a hint message before the user starts typing.

::

        if (isLoading) {
          return const Center(child: CircularProgressIndicator());
        }

Show spinner while loading.

::

        if (results.isEmpty) {
          return const Center(child: Text("not results found"));
        }

Show message if no results match.

::

        return ListView.builder(
          itemCount: results.length,
          itemBuilder: (context, index) {
            final recipe = results[index];

Build a card for each result.

::

            final ingredients = recipe["ingredients"] as List;
            final ingredientNames = ingredients
                .map((i) => i["recipe_ingredient_name"])
                .join(", ");

Extract ingredient names and join them into a comma-separated string.

::

            return Card(
              margin: const EdgeInsets.symmetric(vertical: 8),
              child: ListTile(
                contentPadding: const EdgeInsets.all(8),
                leading: _buildRecipeImage(recipe["recipe_id"]),
                title: Text(recipe["recipe_title"] ?? "no title"),
                subtitle: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text("ingredients: $ingredientNames"),
                    Text("Time: ${recipe["recipe_time"]}"),
                    Text("Difficulty: ${recipe["recipe_difficulty"]}"),
                  ],
                ),
                onTap: () {
                  Navigator.pushNamed(
                    context,
                    AppRoutes.viewRecipe,
                    arguments: {"recipeId": recipe["recipe_id"]},
                  );
                },
              ),
            );

Each card shows the recipe image, title, ingredients, time, and difficulty. Tapping navigates to the recipe detail page.

**Recipe Image Builder:**

::

    Widget _buildRecipeImage(int? recipeId) {
        if (recipeId == null) {
          return _buildPlaceholder();
        }

If no recipe ID, show placeholder.

::

        return ClipRRect(
          borderRadius: BorderRadiusGeometry.circular(8),
          child: Image.network(
            "http://localhost:5000/recipe-image/$recipeId",
            width: 60,
            height: 60,
            fit: BoxFit.cover,
            loadingBuilder: (context, child, LoadingProgress) {
              if (LoadingProgress == null) return child;
              return SizedBox(
                width: 60,
                height: 60,
                child: Center(
                  child: CircularProgressIndicator(
                    value: LoadingProgress.expectedTotalBytes != null
                        ? LoadingProgress.cumulativeBytesLoaded /
                              LoadingProgress.expectedTotalBytes!
                        : null,
                  ),
                ),
              );
            },

Show a progress indicator while the image loads.

::

            errorBuilder: (context, error, stackTrace) {
              return _buildPlaceholder();
            },
          ),
        );

Show placeholder if image fails to load.

**Placeholder Builder:**

::

    Widget _buildPlaceholder() {
        return Container(
          width: 60,
          height: 60,
          decoration: BoxDecoration(
            color: Colors.grey[300],
            borderRadius: BorderRadiusGeometry.circular(8),
          ),
          child: const Icon(Icons.food_bank, color: Colors.grey, size: 30),
        );
    }

A gray box with a food icon.


search_recipe_service.dart - Search API Service
------------------------------------------------

This service calls the search API.

::

    import 'package:http/http.dart' as http;
    import 'package:frontend/core/api_response.dart';

    class SearchService {
      static Future<ApiResponse> searchRecipe(String name) async {
        final response = await http.get(
          Uri.parse(
            "http://localhost:5000/search-recipe",
          ).replace(queryParameters: {"q": name}),
        );

Make a GET request to the search endpoint with the query as a URL parameter.

::

        return ApiResponse(
          statusCode: response.statusCode,
          response: response.body,
        );
      }
    }

Wrap and return the response.


Static Pages
============

cooking_techniques_page.dart - Cooking Techniques Reference
------------------------------------------------------------

This page provides a static reference guide for common cooking techniques. It displays techniques like Sauté, Braise, Roast, and Poach with their descriptions, skill levels, best uses, time ranges, and quick method steps. This page is purely static - it displays hardcoded data and does not interact with any backend services.

**Data Model:**

::

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

The `CookingTechnique` class is a simple immutable data model that holds all information for a single cooking technique. Each property is `final` and `required`, making the objects constant and unchanging after creation.

**Static Techniques Data:**

The page contains a static list of four pre-defined cooking techniques:

::

    static const List<CookingTechnique> _techniques = [
      CookingTechnique(
        title: 'Saute',
        description: 'Cook quickly over medium-high heat with a small amount of fat.',
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
      CookingTechnique(
        title: 'Braise',
        description: 'Sear first, then cook covered with liquid for deep flavor and tenderness.',
        skillLevel: 'Intermediate',
        bestFor: 'Short ribs, chicken thighs, root vegetables',
        timeRange: '45-180 min',
        steps: [
          'Brown ingredients in batches for better fond.',
          'Add aromatics and deglaze with stock or wine.',
          'Cover and simmer low until fork-tender.',
        ],
        icon: Icons.soup_kitchen,
        accent: Color(0xFF6C584C),
      ),
      CookingTechnique(
        title: 'Roast',
        description: 'Use dry oven heat to build caramelization and concentrated flavor.',
        skillLevel: 'Beginner',
        bestFor: 'Potatoes, squash, chicken, fish',
        timeRange: '20-90 min',
        steps: [
          'Preheat oven fully before loading tray.',
          'Use enough space between pieces for airflow.',
          'Turn once for balanced browning.',
        ],
        icon: Icons.outdoor_grill,
        accent: Color(0xFFBC6C25),
      ),
      CookingTechnique(
        title: 'Poach',
        description: 'Gentle cooking in liquid below a simmer for delicate ingredients.',
        skillLevel: 'Intermediate',
        bestFor: 'Eggs, salmon, pears, chicken breast',
        timeRange: '6-25 min',
        steps: [
          'Keep liquid between 160-180 F (71-82 C).',
          'Flavor the poaching liquid with herbs and citrus.',
          'Rest briefly before serving to retain moisture.',
        ],
        icon: Icons.water_drop,
        accent: Color(0xFF4D908E),
      ),
    ];

**Page Structure:**

The `CookingTechniquesPage` is a `StatelessWidget` since its content never changes. It builds a `Scaffold` with:

- An `AppBar` titled "Cooking Techniques"
- A `Container` with a gradient background (warm cream to light blue-gray)
- A `ListView` containing:
  - A `_HeaderCard` showing a motivational message and technique count
  - A list of `_TechniqueCard` widgets, one for each technique

**_HeaderCard Component:**

::

    class _HeaderCard extends StatelessWidget {
      const _HeaderCard({required this.techniqueCount});
      final int techniqueCount;

      @override
      Widget build(BuildContext context) {
        return Container(
          padding: const EdgeInsets.all(20),
          decoration: BoxDecoration(
            color: const Color(0xFF2F3E46),
            borderRadius: BorderRadius.circular(20),
          ),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Text(
                'Build better instincts in the kitchen',
                style: TextStyle(
                  color: Colors.white,
                  fontSize: 22,
                  height: 1.2,
                  fontWeight: FontWeight.w700,
                ),
              ),
              const SizedBox(height: 10),
              Text(
                '$techniqueCount core methods to help you cook with confidence.',
                style: const TextStyle(
                  color: Color(0xFFD6E2E9),
                  fontSize: 14,
                  height: 1.4,
                ),
              ),
            ],
          ),
        );
      }
    }

The header card is a dark teal container with rounded corners that displays an inspirational message and counts how many techniques are available.

**_TechniqueCard Component:**

::

    class _TechniqueCard extends StatelessWidget {
      const _TechniqueCard({required this.technique});
      final CookingTechnique technique;

      @override
      Widget build(BuildContext context) {
        return Card(
          elevation: 0,
          shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(18)),
          child: Container(
            padding: const EdgeInsets.all(16),
            decoration: BoxDecoration(
              borderRadius: BorderRadius.circular(18),
              border: Border.all(color: technique.accent.withValues(alpha: 0.25)),
              color: Colors.white,
            ),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                // Header row with icon, title, and skill level chip
                Row(
                  children: [
                    Container(
                      height: 42,
                      width: 42,
                      decoration: BoxDecoration(
                        color: technique.accent.withValues(alpha: 0.15),
                        borderRadius: BorderRadius.circular(12),
                      ),
                      child: Icon(technique.icon, color: technique.accent),
                    ),
                    const SizedBox(width: 12),
                    Expanded(
                      child: Text(
                        technique.title,
                        style: const TextStyle(
                          fontSize: 18,
                          fontWeight: FontWeight.w700,
                        ),
                      ),
                    ),
                    Chip(
                      backgroundColor: technique.accent.withValues(alpha: 0.15),
                      side: BorderSide.none,
                      label: Text(
                        technique.skillLevel,
                        style: TextStyle(
                          color: technique.accent,
                          fontWeight: FontWeight.w600,
                        ),
                      ),
                    ),
                  ],
                ),
                const SizedBox(height: 10),
                // Description
                Text(
                  technique.description,
                  style: const TextStyle(
                    fontSize: 14,
                    height: 1.35,
                    color: Color(0xFF30353A),
                  ),
                ),
                const SizedBox(height: 12),
                // Meta information rows
                _MetaRow(label: 'Best for', value: technique.bestFor),
                const SizedBox(height: 6),
                _MetaRow(label: 'Typical time', value: technique.timeRange),
                const SizedBox(height: 12),
                // Steps section
                const Text(
                  'Quick method',
                  style: TextStyle(
                    fontWeight: FontWeight.w600,
                    color: Color(0xFF1F2933),
                  ),
                ),
                const SizedBox(height: 8),
                ...technique.steps.asMap().entries.map(
                  (entry) => Padding(
                    padding: const EdgeInsets.only(bottom: 6),
                    child: Text(
                      '${entry.key + 1}. ${entry.value}',
                      style: const TextStyle(
                        color: Color(0xFF3E4C59),
                        height: 1.35,
                      ),
                    ),
                  ),
                ),
              ],
            ),
          ),
        );
      }
    }

Each technique card displays:
1. A colored icon matching the technique's accent color
2. The technique title
3. A skill level chip (Beginner/Intermediate)
4. A brief description
5. "Best for" and "Typical time" metadata rows
6. A numbered list of quick method steps

**_MetaRow Component:**

::

    class _MetaRow extends StatelessWidget {
      const _MetaRow({required this.label, required this.value});
      final String label;
      final String value;

      @override
      Widget build(BuildContext context) {
        return Row(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            SizedBox(
              width: 90,
              child: Text(
                label,
                style: const TextStyle(
                  color: Color(0xFF7B8794),
                  fontWeight: FontWeight.w600,
                ),
              ),
            ),
            Expanded(
              child: Text(value, style: const TextStyle(color: Color(0xFF323F4B))),
            ),
          ],
        );
      }
    }

This helper widget creates a consistent label-value pair row, with the label taking a fixed width of 90 pixels and the value taking the remaining space.

**Navigation Integration:**

This page is registered in `routes.dart` under the route name `AppRoutes.techniques` (which is "/techniques"). It is accessible from the home page via the techniques quick action button in the app bar.


profile_page.dart - User Account Profile Page
----------------------------------------------

This page provides a static user account management interface. It displays user information, allows setting dietary preferences, stores diet notes, and shows recipe completion tracking. Like the cooking techniques page, this page is static - it does not save preferences to a backend server (though it simulates saving with a snackbar message).

**Page Parameters:**

::

    class AccountPage extends StatefulWidget {
      final String username;
      final bool newAccount;

      const AccountPage({
        super.key,
        required this.username,
        required this.newAccount,
      });

The page requires two parameters:
- `username`: The logged-in user's name to display
- `newAccount`: A flag indicating whether this is a new account (shows a setup reminder banner)

**State Class Properties:**

::

    class _AccountPageState extends State<AccountPage> {
      static const List<String> _dietaryOptions = <String>[
        'Vegetarian',
        'Vegan',
        'Gluten free',
        'Dairy free',
        'Nut free',
        'Low sodium',
        'High protein',
        'Halal',
      ];

      final TextEditingController _notesController = TextEditingController(
        text: 'Keep dishes mild and avoid peanuts.',
      );

      final List<_RecipeCompletion> _recipeCompletion = const <_RecipeCompletion>[
        _RecipeCompletion(
          title: 'Grilled Chicken Bowl',
          subtitle: 'Protein-focused dinner',
          progress: 0.9,
          completed: true,
        ),
        _RecipeCompletion(
          title: 'Vegetable Curry',
          subtitle: 'Lunch prep in progress',
          progress: 0.65,
          completed: false,
        ),
        _RecipeCompletion(
          title: 'Berry Breakfast Parfait',
          subtitle: 'Quick breakfast option',
          progress: 0.4,
          completed: false,
        ),
      ];

      final Set<String> _selectedDietaryOptions = <String>{
        'Low sodium',
        'Nut free',
      };

The state maintains:
- A static list of all available dietary options
- A controller for diet notes (pre-filled with example text)
- A static list of recipe completion tracking items
- A set of selected dietary options (initially 'Low sodium' and 'Nut free')

**_RecipeCompletion Data Class:**

::

    class _RecipeCompletion {
      const _RecipeCompletion({
        required this.title,
        required this.subtitle,
        required this.progress,
        required this.completed,
      });

      final String title;
      final String subtitle;
      final double progress;
      final bool completed;
    }

This private class holds data for tracking recipe completion progress. The `progress` field is a double between 0.0 and 1.0 representing completion percentage.

**Helper Methods:**

::

    void _toggleDietaryRequirement(String option) {
      setState(() {
        if (_selectedDietaryOptions.contains(option)) {
          _selectedDietaryOptions.remove(option);
          return;
        }
        _selectedDietaryOptions.add(option);
      });
    }

Toggles a dietary option on/off by adding or removing it from the selected set.

::

    void _saveAccountPreferences() {
      FocusScope.of(context).unfocus();
      ScaffoldMessenger.of(
        context,
      ).showSnackBar(const SnackBar(content: Text('Account preferences saved.')));
    }

Simulates saving preferences. In a real app, this would send the data to a backend. Here it simply dismisses the keyboard and shows a snackbar message.

::

    void _signOut() {
      AuthService.clearSession();
      Navigator.of(
        context,
      ).pushNamedAndRemoveUntil(AppRoutes.login, (route) => false);
    }

Signs the user out by clearing the session cookie and navigating back to the login screen, removing all previous routes from the navigation stack.

::

    String _initialsFor(String username) {
      final parts = username.trim().split(RegExp(r'\s+'));
      if (parts.isEmpty || parts.first.isEmpty) {
        return '?';
      }
      if (parts.length == 1) {
        return parts.first[0].toUpperCase();
      }
      return '${parts.first[0].toUpperCase()}${parts.last[0].toUpperCase()}';
    }

Generates initials from a username. For example:
- "John Doe" returns "JD"
- "John" returns "J"
- Empty string returns "?"

**UI Structure:**

The page uses a `Scaffold` with an `AppBar` containing a sign-out icon button. The body is a `SafeArea` wrapped around a `ListView` with the following sections:

1. **User Header Card:** Displays the user's initials in a circle avatar along with their username and a status message (different for new vs existing accounts).

2. **New Account Banner:** Only shown if `widget.newAccount` is true - a reminder to complete the diet profile.

3. **Dietary Requirements Section:** A `Wrap` of `FilterChip` widgets for each dietary option. Tapping a chip toggles its selection state.

4. **Diet Notes Section:** A `TextField` for entering allergies, ingredient preferences, or reminders.

5. **Recipe Completion Section:** Shows two metric cards (completed count and average progress) followed by a list of recipe completion items with progress bars.

6. **Action Buttons:** Sign out and Save changes buttons side by side.

**_SectionCard Component:**

::

    class _SectionCard extends StatelessWidget {
      const _SectionCard({
        required this.title,
        required this.subtitle,
        required this.child,
      });

      final String title;
      final String subtitle;
      final Widget child;

      @override
      Widget build(BuildContext context) {
        final theme = Theme.of(context);

        return Container(
          padding: const EdgeInsets.all(18),
          decoration: BoxDecoration(
            color: theme.colorScheme.surface,
            borderRadius: BorderRadius.circular(20),
            border: Border.all(color: theme.colorScheme.outlineVariant),
          ),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                title,
                style: theme.textTheme.titleMedium?.copyWith(
                  fontWeight: FontWeight.w700,
                ),
              ),
              const SizedBox(height: 4),
              Text(subtitle, style: theme.textTheme.bodyMedium),
              const SizedBox(height: 16),
              child,
            ],
          ),
        );
      }
    }

A reusable card component with a title, subtitle, and custom child content. Uses the app's theme colors for consistency.

**_CompletionMetricCard Component:**

::

    class _CompletionMetricCard extends StatelessWidget {
      const _CompletionMetricCard({
        required this.label,
        required this.value,
        required this.icon,
        required this.color,
      });

      final String label;
      final String value;
      final IconData icon;
      final Color color;

      @override
      Widget build(BuildContext context) {
        final theme = Theme.of(context);

        return Container(
          padding: const EdgeInsets.all(14),
          decoration: BoxDecoration(
            color: color.withValues(alpha: 0.12),
            borderRadius: BorderRadius.circular(16),
          ),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Icon(icon, color: color),
              const SizedBox(height: 10),
              Text(
                value,
                style: theme.textTheme.titleLarge?.copyWith(
                  fontWeight: FontWeight.w800,
                ),
              ),
              const SizedBox(height: 4),
              Text(label),
            ],
          ),
        );
      }
    }

A small metric card showing an icon, a value (like "2/3" or "65%"), and a label. Used for displaying completion statistics.

**Recipe Completion Items:**

The recipe completion section maps over `_recipeCompletion` to create a list of items. Each item shows:
- A circle avatar with a check icon (if completed) or hourglass icon (if in progress)
- The recipe title and subtitle
- A linear progress bar
- The completion percentage as text

**Navigation Integration:**

This page is registered in `routes.dart` under the route name `AppRoutes.profile` (which is "/profile"). It is accessible from the home page via the profile icon button in the app bar, with username and newAccount parameters passed from the home page state.


View Recipe Features
====================

view_recipe.dart - Step-by-Step Recipe Viewer
----------------------------------------------

This is the most sophisticated screen in the app. It provides a guided cooking mode where users follow recipes one step at a time.

::

    class RecipePage extends StatefulWidget {
      final int recipeId;
      const RecipePage({super.key, required this.recipeId});

      @override
      State<RecipePage> createState() => _RecipePageState();
    }

The page requires a recipeId to know which recipe to display.

**State Class:**

::

    class _RecipePageState extends State<RecipePage> {
      String recipeTitle = "";
      String? recipeTime;
      String? recipeDifficulty;

      List<Map<String, dynamic>> steps = [];
      Set<int> completedSteps = {};

      int? currentStepId;

      bool isLoading = true;

The state tracks recipe metadata, a list of steps, a set of completed step IDs, the current step ID, and loading state.

**Initialization:**

::

    @override
    void initState() {
        super.initState();
        loadRecipe();
    }

Load the recipe data when the page first appears.

**Load Recipe Function:**

::

    Future<void> loadRecipe() async {
        final response = await ViewService.viewRecipe(widget.recipeId.toString());

        if (response.statusCode != 200) {
          setState(() {
            isLoading = false;
          });
          return;
        }

Fetch the recipe data. If it fails, stop loading and return (the UI will show an error state).

::

        final data = jsonDecode(response.response);

        recipeTitle = data["recipe-title"] ?? "";
        recipeTime = data["recipe-time"]?.toString();
        recipeDifficulty = data["recipe-difficulty"]?.toString();

Extract the recipe metadata.

::

        final fetchedSteps = data["recipe-steps"] ?? [];

        steps = [];

        for (final step in fetchedSteps) {
          steps.add({
            "id": int.parse(step["recipe-step-id"].toString()),
            "text": step["step-description"] ?? "",
            "duration": step["step-duration"],
            "completed": step["step-completion"] ?? false,
          });
        }

Transform the steps into a consistent format.

::

        completedSteps = {};

        for (final step in steps) {
          if (step["completed"] == true) {
            completedSteps.add(step["id"]);
          }
        }

Build a set of already completed step IDs.

::

        setState(() {
          final remaining = steps.where((s) => !completedSteps.contains(s["id"]));
          currentStepId = remaining.isEmpty ? null : remaining.first["id"];
          isLoading = false;
        });

Find the first uncompleted step and set it as current. If all steps are complete, set currentStepId to null.

**Current Step Getter:**

::

    Map<String, dynamic> get currentStep =>
        steps.firstWhere((s) => s["id"] == currentStepId);

This computed property returns the full step data for the current step ID.

**Complete Step Function:**

::

    Future<void> completedStep() async {
        if (currentStepId == null) return;

Don't do anything if there's no current step.

::

        final stepToComplete = currentStepId;

        setState(() {
          completedSteps.add(currentStepId!);
          final next = steps.where((s) => !completedSteps.contains(s["id"]));
          currentStepId = next.isEmpty ? null : next.first["id"];
        });

Add the current step to the completed set and find the next uncompleted step. This updates the UI immediately (optimistic update).

::

        try {
          final response = await ViewService.completeStep(stepToComplete!);
          if (response.statusCode != 200) {
            debugPrint("good");
          }
        } catch (e) {
          debugPrint("error $e");
        }

Send the completion to the server. If it fails, we just log it - the UI has already updated.

**Go Back Function:**

::

    Future<void> goBack() async {
        final completedList = completedSteps.toList();
        if (completedList.isEmpty) {
          return;
        }

Don't go back if no steps are completed.

::

        final stepToUncomplete = completedList.last;
        setState(() {
          currentStepId = completedList.last;
          completedSteps.remove(currentStepId);
        });

Set the current step to the last completed step and remove it from the completed set. This is another optimistic update.

::

        try {
          final response = await ViewService.unCompleteStep(stepToUncomplete);
          if (response.statusCode != 200) {
            debugPrint("good");
          }
        } catch (e) {
          debugPrint("error $e");
        }

Send the uncomplete request to the server.

**Progress Calculation:**

::

    double get progress =>
        steps.isEmpty ? 0 : completedSteps.length / steps.length;

Calculate completion percentage for the progress bar.

**Step Image Builder:**

::

    Widget _buildStepImage(int stepId) {
        return Image.network(
          "http://localhost:5000/step-image/$stepId",
          height: 200,
          width: double.infinity,
          fit: BoxFit.cover,
          errorBuilder: (context, error, StackTrace) {
            return const SizedBox.shrink();
          },
        );
    }

Load the step image from the server. If it fails, return an empty widget (SizedBox.shrink()).

**Building the UI:**

::

    @override
    Widget build(BuildContext context) {
        if (isLoading) {
          return Scaffold(
            appBar: AppBar(title: const Text("loading")),
            body: const Center(child: CircularProgressIndicator()),
          );
        }

Show loading state while fetching data.

::

        if (currentStepId == null) {
          return Scaffold(
            appBar: AppBar(title: Text(recipeTitle)),
            body: const Center(child: Text("completed recipe")),
          );
        }

Show completion message if all steps are done.

**Main Recipe UI:**

::

    return Scaffold(
      appBar: AppBar(
        title: Text(recipeTitle),
        actions: [
          if (recipeTime != null)
            Padding(
              padding: const EdgeInsets.only(right: 16),
              child: Center(child: Text(recipeTime!)),
            ),
        ],
      ),

Show recipe title and total time in the app bar.

**Progress Section:**

::

    Padding(
      padding: const EdgeInsets.all(16),
      child: Column(
        children: [
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Text(
                "step ${steps.indexWhere((s) => s["id"] == currentStepId) + 1}",
                style: const TextStyle(fontWeight: FontWeight.bold),
              ),

Show "step 1", "step 2", etc. by finding the index of the current step.

::

              if (recipeDifficulty != null)
                Text("Difficulty: $recipeDifficulty"),
            ],
          ),
          const SizedBox(height: 10),
          LinearProgressIndicator(value: progress),

Show a progress bar.

**Step Content:**

::

    Expanded(
      child: SingleChildScrollView(
        padding: const EdgeInsets.all(20),
        child: Column(
          children: [
            _buildStepImage(currentStepId!),

Show the step image.

::

            const SizedBox(height: 20),

            Text(
              currentStep["text"],
              textAlign: TextAlign.center,
              style: const TextStyle(fontSize: 18),
            ),

Show the step description.

::

            if (currentStep["duration"] != null) ...[
              const SizedBox(height: 20),
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Icon(Icons.timer, size: 20),
                  const SizedBox(width: 8),
                  Text(
                    currentStep["duration"],
                    style: const TextStyle(fontSize: 16),
                  ),
                ],
              ),
            ],

If the step has a duration, show it with a timer icon.

**Action Buttons:**

::

    Padding(
      padding: const EdgeInsets.all(16),
      child: Column(
        children: [
          SizedBox(
            width: double.infinity,
            child: ElevatedButton(
              onPressed: completedStep,
              child: const Text("complete step"),
            ),
          ),

Full-width button to complete the current step.

::

          if (completedSteps.isNotEmpty)
            TextButton(onPressed: goBack, child: const Text("go back")),

If any steps are completed, show a "go back" button to undo the last completion.


service_view_recipe.dart - Recipe Viewing API Service
------------------------------------------------------

This service handles API calls related to viewing and interacting with recipes.

**View Recipe Method:**

::

    class ViewService {
      static Future<ApiResponse> viewRecipe(String recipeId) async {
        final response = await http.get(
          Uri.parse(
            "http://localhost:5000/view-recipe/$recipeId",
          ).replace(queryParameters: {"user_id": Session.userId.toString()}),
          headers: {
            if (AuthService.sessionCookie != null)
              'Cookie': AuthService.sessionCookie!,
          },
        );

Make a GET request to fetch the recipe, passing the user ID as a query parameter and the session cookie in the headers.

::

        return ApiResponse(
          statusCode: response.statusCode,
          response: response.body,
        );
      }

**Complete Step Method:**

::

      static Future<ApiResponse> completeStep(int recipeStepId) async {
        final response = await http.post(
          Uri.parse("http://localhost:5000/complete-step"),
          headers: {
            "Content-Type": "application/Json",
            if (AuthService.sessionCookie != null)
              'Cookie': AuthService.sessionCookie!,
          },
          body: jsonEncode({
            "recipe_step_id": recipeStepId,
            "user_id": Session.userId,
          }),
        );

Make a POST request to mark a step as complete, sending the step ID and user ID as JSON.

::

        return ApiResponse(
          statusCode: response.statusCode,
          response: response.body,
        );
      }

**Uncomplete Step Method:**

::

      static Future<ApiResponse> unCompleteStep(int recipeStepId) async {
        final response = await http.post(
          Uri.parse("http://localhost:5000/uncomplete-step"),
          headers: {
            "Content-Type": "application/Json",
            if (AuthService.sessionCookie != null)
              'Cookie': AuthService.sessionCookie!,
          },
          body: jsonEncode({
            "recipe_step_id": recipeStepId,
            "user_id": Session.userId,
          }),
        );

Similar to completeStep but calls a different endpoint to undo completion.

::

        return ApiResponse(
          statusCode: response.statusCode,
          response: response.body,
        );
      }
    }


Summary and Key Concepts
=========================

State Management
----------------

The app uses Flutter's built-in StatefulWidget pattern for state management. Each screen that needs to change maintains its own state. Global state (user session) is managed through a simple static class.

Navigation
----------

Named routes are used throughout the app. Routes are defined as constants and handled by a central generateRoute function. This makes navigation consistent and maintainable.

API Communication
-----------------

Service classes handle all API communication. They return standardized ApiResponse objects containing status codes and response bodies. Authentication is managed through session cookies stored in the AuthService.

Form Validation
---------------

TextFormField widgets with validator functions provide inline form validation. The Form widget with a GlobalKey allows validating all fields at once.

Image Handling
--------------

The image_picker package is used to select images from the device gallery. Images are uploaded to the server using multipart/form-data requests.

Async Operations
----------------

All network operations are asynchronous and use Future/async/await patterns. Loading states and error handling ensure a smooth user experience even when network requests are slow or fail.

Widget Composition
------------------

The app makes heavy use of custom widget builders to avoid code duplication. Helper methods like buildTextInputField, buildImagePicker, and buildDurationFields create consistent UI elements.

Memory Management
-----------------

TextEditingController and other resources are properly disposed in the dispose method to prevent memory leaks. The mounted property is checked before updating state after async operations.

Static Pages
------------

The app includes two static reference pages:
- **Cooking Techniques Page:** A reference guide displaying common cooking techniques with their descriptions, skill levels, best uses, and step-by-step methods.
- **Account Profile Page:** A user profile management interface with dietary preferences, notes, and recipe completion tracking (static data only).

These pages do not communicate with backend services and serve as UI demonstrations or reference material.

API Endpoints Used
==================

Authentication
--------------
- POST /authenticate - Login
- POST /create-account - Register new user

Recipes
-------
- GET /get-recipes - Fetch all recipes
- POST /add-recipe - Create new recipe (multipart)
- GET /search-recipe?q=query - Search recipes
- GET /view-recipe/:recipeId?user_id=userId - Get recipe details
- GET /recipe-image/:recipeId - Get recipe image
- GET /step-image/:stepId - Get step image

Step Tracking
-------------
- POST /complete-step - Mark step as complete
- POST /uncomplete-step - Mark step as incomplete

All authenticated endpoints require a session cookie in the Cookie header.
