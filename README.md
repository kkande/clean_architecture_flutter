# testing_clean_architecture_register

A new Flutter project.

## Getting Started

This project is a starting point for a Flutter application.

A few resources to get you started if this is your first Flutter project:

- [Lab: Write your first Flutter app](https://docs.flutter.dev/get-started/codelab)
- [Cookbook: Useful Flutter samples](https://docs.flutter.dev/cookbook)

For help getting started with Flutter development, view the
[online documentation](https://docs.flutter.dev/), which offers tutorials,
samples, guidance on mobile development, and a full API reference.


/**
Bien sûr, voici un exemple simplifié d'une page d'inscription en utilisant l'architecture propre (Clean Architecture) avec Flutter. Dans cet exemple, j'utiliserai le package `provider` pour la gestion des états et de l'injection de dépendances.

### Structure du projet :

```
lib/
|-- core/
|   |-- entities/
|       |-- user.dart
|   |-- usecases/
|       |-- register_user_usecase.dart
|   |-- repositories/
|       |-- user_repository.dart
|-- data/
|   |-- repositories_impl/
|       |-- user_repository_impl.dart
|   |-- data_sources/
|       |-- user_data_source.dart
|-- presentation/
|   |-- screens/
|       |-- registration_screen.dart
|   |-- widgets/
|       |-- registration_form.dart
|   |-- blocs/
|       |-- registration_bloc.dart
|-- main.dart
```

### Contenu des fichiers :

1. **`core/entities/user.dart`** :

```dart
// core/entities/user.dart

class User {
  final String username;
  final String email;
  final String password;

  User({required this.username, required this.email, required this.password});
}
```

2. **`core/usecases/register_user_usecase.dart`** :

```dart
// core/usecases/register_user_usecase.dart

import 'package:your_app/core/entities/user.dart';
import 'package:your_app/core/repositories/user_repository.dart';

class RegisterUserUseCase {
  final UserRepository userRepository;

  RegisterUserUseCase(this.userRepository);

  Future<void> execute(User user) {
    return userRepository.registerUser(user);
  }
}
```

3. **`core/repositories/user_repository.dart`** :

```dart
// core/repositories/user_repository.dart

import 'package:your_app/core/entities/user.dart';

abstract class UserRepository {
  Future<void> registerUser(User user);
}
```

4. **`data/repositories_impl/user_repository_impl.dart`** :

```dart
// data/repositories_impl/user_repository_impl.dart

import 'package:your_app/core/entities/user.dart';
import 'package:your_app/core/repositories/user_repository.dart';
import 'package:your_app/data/data_sources/user_data_source.dart';

class UserRepositoryImpl implements UserRepository {
  final UserDataSource userDataSource;

  UserRepositoryImpl(this.userDataSource);

  @override
  Future<void> registerUser(User user) {
    return userDataSource.registerUser(user);
  }
}
```

5. **`data/data_sources/user_data_source.dart`** :

```dart
// data/data_sources/user_data_source.dart

import 'package:your_app/core/entities/user.dart';

abstract class UserDataSource {
  Future<void> registerUser(User user);
}
```

6. **`presentation/screens/registration_screen.dart`** :

```dart
// presentation/screens/registration_screen.dart

import 'package:flutter/material.dart';
import 'package:your_app/presentation/widgets/registration_form.dart';

class RegistrationScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Registration'),
      ),
      body: RegistrationForm(),
    );
  }
}
```

7. **`presentation/widgets/registration_form.dart`** :

```dart
// presentation/widgets/registration_form.dart

import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:your_app/presentation/blocs/registration_bloc.dart';

class RegistrationForm extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final registrationBloc = Provider.of<RegistrationBloc>(context);

    return Padding(
      padding: const EdgeInsets.all(16.0),
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          TextField(
            onChanged: (value) => registrationBloc.updateUsername(value),
            decoration: InputDecoration(labelText: 'Username'),
          ),
          TextField(
            onChanged: (value) => registrationBloc.updateEmail(value),
            decoration: InputDecoration(labelText: 'Email'),
          ),
          TextField(
            onChanged: (value) => registrationBloc.updatePassword(value),
            decoration: InputDecoration(labelText: 'Password'),
            obscureText: true,
          ),
          SizedBox(height: 20),
          ElevatedButton(
            onPressed: () => registrationBloc.registerUser(),
            child: Text('Register'),
          ),
        ],
      ),
    );
  }
}
```

8. **`presentation/blocs/registration_bloc.dart`** :

```dart
// presentation/blocs/registration_bloc.dart

import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:your_app/core/entities/user.dart';
import 'package:your_app/core/usecases/register_user_usecase.dart';

class RegistrationBloc extends ChangeNotifier {
  final RegisterUserUseCase registerUserUseCase;

  RegistrationBloc(this.registerUserUseCase);

  String _username = '';
  String _email = '';
  String _password = '';

  void updateUsername(String value) {
    _username = value;
  }

  void updateEmail(String value) {
    _email = value;
  }

  void updatePassword(String value) {
    _password = value;
  }

  void registerUser() async {
    final user = User(username: _username, email: _email, password: _password);
    await registerUserUseCase.execute(user);
    // Add navigation logic or other actions after registration
  }
}

class RegistrationBlocProvider extends StatelessWidget {
  final Widget child;

  RegistrationBlocProvider({required this.child});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (context) => RegistrationBloc(
        Provider.of<RegisterUserUseCase>(context),
      ),
      child: child,
    );
  }
}
```

9. **`main.dart`** :

```dart
// main.dart

import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:your_app/data/repositories_impl/user_repository_impl.dart';
import 'package:your_app/presentation/screens/registration_screen.dart';
import 'package:your_app/core/usecases/register_user_usecase.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Clean Architecture Example',
      home: MultiProvider(
        providers: [
          Provider<RegisterUserUseCase>(
            create: (context) => RegisterUserUseCase(
              UserRepositoryImpl(
                // Provide the necessary data source here.
              ),
            ),
          ),
        ],
        child: RegistrationBlocProvider(
          child: RegistrationScreen(),
        ),
      ),
    );
  }
}
```

Cet exemple est basé sur des structures propres à Flutter et à l'architecture propre. Assurez-vous de personnaliser les détails en fonction des besoins spécifiques de votre application. Vous pouvez également ajouter des validations, des gestionnaires d'état plus avancés (comme Riverpod ou Bloc), et des fonctionnalités supplémentaires selon vos besoins.
**/