---
title: Add login to your Flutter app
default: true
description: This tutorial demonstrates how to add user login with Auth0 to an Flutter Web application using the Auth0 Flutter SDK
budicon: 448
topics:
  - quickstarts
  - native
  - flutter
  - dart
  - ios
  - android
contentType: tutorial
useCase: quickstart
github:
  path: sample
---

<!-- markdownlint-disable MD025 MD034 -->

# Add login to your Flutter app

Auth0 allows you to quickly add authentication and access user profile information in your application. This guide demonstrates how to integrate Auth0 with a Flutter Web application using the <a href="https://github.com/auth0/auth0-flutter" target="_blank" rel="noreferrer">Auth0 Flutter SDK</a>.

:::note
The Flutter SDK currently only supports Flutter applications running on Android, iOS, or Web platforms.
:::

## Getting started

This quickstart assumes you already have a <a href="https://flutter.dev/" target="_blank" rel="noreferrer">Flutter</a> application up and running. If not, check out the <a href="https://docs.flutter.dev/get-started/install" target="_blank" rel="noreferrer">Flutter "getting started" guides</a> to get started with a simple app.

You should also be familiar with the <a href="https://docs.flutter.dev/reference/flutter-cli" target="_blank" rel="noreferrer">Flutter command line tool</a>.

<%= include('../_includes/_getting_started', { library: 'Flutter', callback: 'http://localhost:3000', returnTo: 'http://localhost:3000', webOriginUrl: 'http://localhost:3000', showLogoutInfo: true, showWebOriginInfo: true, new_js_sdk: true, show_install_info: false }) %>

<%= include('_install_sdk') %>

## Add login to your app

<a href="https://auth0.com/docs/authenticate/login/auth0-universal-login" target="_blank" rel="noreferrer">Universal Login</a> is the easiest way to set up authentication in your application. We recommend using it for the best experience, best security, and the fullest array of features.

Integrate Auth0 Universal Login in your Flutter app by importing the SDK and instantiating the `Auth0` class using your Auth0 domain and Client ID values. See this example, which instantiates the class inside a widget state object:

```dart
import 'package:auth0_flutter/auth0_flutter.dart';
import 'package:auth0_flutter/auth0_flutter_web.dart';

class MainView extends StatefulWidget {
  const MainView({Key? key}) : super(key: key);

  @override
  State<MainView> createState() => _MainViewState();
}

class _MainViewState extends State<MainView> {
  Credentials? _credentials;

  late Auth0Web auth0;

  @override
  void initState() {
    super.initState();
    auth0 = Auth0Web('${account.namespace}', '${account.clientId}');
  }

  @override
  Widget build(BuildContext context) {
    // ...
  }
}
```

Next, redirect your users to the Auth0 Universal Login page using `loginWithRedirect`. See this example of a `ElevatedButton` widget that logs the user in when clicked:

:::note
You will normally need to specify the `redirectUrl` parameter to `loginWithRedirect`. Omitting this will cause Auth0 to use the <a href="https://auth0.com/docs/authenticate/login/auth0-universal-login/configure-default-login-routes" target="_blank" rel="noreferrer">default login route</a>, which is not configured by default.
:::

```dart
if (_credentials == null) {
  ElevatedButton(
    onPressed: () => auth0.loginWithRedirect(redirectUrl: 'http://localhost:3000'),
    child: const Text("Log in")
  )
}
```

When a user logs in, they are redirected back to your application. You are then able to access the ID and access tokens for this user by calling `onLoad` during startup and handling the credentials that are given to you:

```dart
auth0.onLoad().then((final credentials) => setState(() {
    // Handle or store credentials here
    _credentials = credentials;
  }));
```

::::checkpoint
:::checkpoint-default
Add a button to your app that calls `loginWithRedirect()` and logs the user into your app. Verify that you are redirected to Auth0 for authentication and then back to your application.

Verify that you can access `credentials` as a result of calling `onLoad` and that you're able to access the ID and access tokens.
:::

:::checkpoint-failure
If your application did not launch successfully:

- Ensure the Allowed Callback URLs are set properly
- Verify you saved your changes after entering your URLs
- Make sure the domain and client ID values are imported correctly

Still having issues? Check out our <a href="https://auth0.com/docs" target="_blank" rel="noreferrer">documentation</a> or visit our <a href="https://community.auth0.com" target="_blank" rel="noreferrer">community page</a> to get more help.
:::
::::

## Add logout to your app

To log users out, redirect them to the Auth0 logout endpoint to clear their login session by calling the Auth0 Flutter SDK `logout()`. <a href="https://auth0.com/docs/authenticate/login/logout" target="_blank" rel="noreferrer">Read more about logging out of Auth0</a>.

See this example of an `ElevatedButton` widget that logs the user out of the app:

```dart
ElevatedButton(
  onPressed: () async {
    await auth0.logout();
  },
  child: const Text("Log out"))
```

::::checkpoint
:::checkpoint-default
Add a button to your app that calls `logout()` and logs the user out of your application. When you select it, verify that your Flutter app redirects you to the logout endpoint and back again. You should not be logged in to your application.
:::

:::checkpoint-failure
If your application did not log out successfully:

- Ensure the Allowed Logout URLs are set properly
- Verify you saved your changes after entering your URLs

Still having issues? Check out our <a href="https://auth0.com/docs" target="_blank" rel="noreferrer">documentation</a> or visit our <a href="https://community.auth0.com" target="_blank" rel="noreferrer">community page</a> to get more help.
:::
::::

## Show user profile information

The user profile automatically retrieves user profile properties for you when the page loads, and can be accessed and stored by calling `onLoad` during application startup. The returned object from `onLoad` contains a `user` property with all the user profile properties. This is internally populated by decoding the ID token.

See this example of a `View` component that displays the user profile on the screen:

```dart
import 'package:auth0_flutter/auth0_flutter.dart';
import 'package:flutter/material.dart';

class ProfileView extends StatelessWidget {
  const ProfileView({Key? key, required this.user}) : super(key: key);

  final UserProfile user;

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        if (user.name != null) Text(user.name!),
        if (user.email != null) Text(user.email!)
      ],
    );
  }
}
```

::::checkpoint
:::checkpoint-default
Log in and inspect the `user` property on the result. Verify the current user's profile information, such as `email` or `name`.
:::
:::checkpoint-failure
If your application did not return user profile information:

- Verify the ID token is valid

Still having issues? Check out our <a href="https://auth0.com/docs" target="_blank" rel="noreferrer">documentation</a> or visit our <a href="https://community.auth0.com" target="_blank" rel="noreferrer">community page</a> to get more help.
:::
::::
