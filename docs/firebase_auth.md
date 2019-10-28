# Firebase Auth

> Utilisation du gestionnaire de state 'provider' et de firebase Auth

## Prérequis

###### Android X

Afin de pouvoir utiliser les service de google, il faut **impérativement** une application flutter sous androidX

```
    flutter create --androix project_name
```

###### Avoir générer un projet sur firebase

[Firebase](firebase.md)

## Initialisation de l'application

Dans le pubspec.yaml rajouter

```yaml
    firebase_auth: ^0.14.0+5
    google_sign_in: ^4.0.7
    provider: ^3.1.0
    cloud_firestore:  ^0.12.9+5
```

Faire un pub get


## Gestion de l'authentification

```dart
    /// Classe pour typer le retour de nos fonctions
class AuthResponse {
  bool success;
  String message;

  AuthResponse(this.success, this.message);
}

/// Un enum pour les difféerents status que nous autorisons
enum Status { Uninitialized, Authenticated, Authenticating, Unauthenticated }

/* Classe qui va gérer l'auth, 
 * elle implémentes la classe ChangeNotifier 
 * pour la gestion des events
*/
class AuthProvider with ChangeNotifier {
  /* On déclare une instance de FirebaseAuth, 
   * elle va servir pour maintenir connecter l'utilisateur 
   * et le déconecter apres
  */
  FirebaseAuth _auth = FirebaseAuth.instance;
  
  
  FirebaseUser _user;
  
  ///Par défatut notre status est non initialisé
  Status _status = Status.Uninitialized;

  /// On déclarer une nouvelle instance de google signin
  GoogleSignIn _googleSignIn = new GoogleSignIn();

  /*On declare ici qu'a chaque changement de state de l'instance _auth, 
  * on utilise la fonction _onAuthStateChanged
  */
  AuthProvider() {
    this._auth.onAuthStateChanged.listen(_onAuthStateChanged);
  }
  //#region getter
  
  Status get status => _status;
  FirebaseUser get user => _user;
  
  //#endregion getter
  
  Future signOut() async {
    this._user = null;
    this._auth.signOut();
    this._status = Status.Unauthenticated;
    await this._googleSignIn.signOut();
    
    ///permet de notifier dans l'application les changment des variables
    notifyListeners(); 
    
    return Future.delayed(Duration.zero);
  }
  
  //#region sign with password/email

  Future resetPassword(String email) async {
    await this._auth.sendPasswordResetEmail(email: email);
  }

  Future<AuthResponse> signInWithEmailAndPassword(
      String email, String password) async {
    /// On change notre status  
    this._status = Status.Authenticating;
    notifyListeners();
    
    try {
      await this._auth
          .signInWithEmailAndPassword(email: email, password: password);
      this._status = Status.Authenticated;
      notifyListeners();
      return new AuthResponse(true, 'success');
    } catch (e) {
      this._status = Status.Unauthenticated;
      notifyListeners();
      return new AuthResponse(false, e.message);
    }
  }

  /// dès que le state de fireauth est changé, cette fonction est appelée
  Future<void> _onAuthStateChanged(FirebaseUser firebaseUser) async {
    if (firebaseUser == null) {
      this._status = Status.Unauthenticated;
    } else {
      this._user = firebaseUser;
      this._status = Status.Authenticated;
    }
    notifyListeners();
  }

  Future<AuthResponse> signUpWithEmailAndPassword(
      String firstname, String lastname, String email, String password) async {
    this._status = Status.Authenticating;
    try {
      await this._auth
          .createUserWithEmailAndPassword(email: email, password: password);
      
      ///une fois notre utilisateur conecté, on l'enregistre dans notre bdd
      Firestore.instance.collection('users').document(this._user.uid).setData({
        'uid': this._user.uid,
        'firstname': firstname,
        'lastname': lastname,
        'email': email,
        'photo': ''
      });

      return new AuthResponse(true, user.uid);
    } catch (e) {
      this._status = Status.Unauthenticated;
      notifyListeners();
      return new AuthResponse(false, e.message);
    }
  }

  //#endregion sign with password / email

  //#region google sign
  // https://medium.com/flutter-community/flutter-implementing-google-sign-in-71888bca24ed
  Future<AuthResponse> signInWithGoogle() async {
    try {
      final GoogleSignInAccount googleUser = await this._googleSignIn.signIn();
      final GoogleSignInAuthentication googleAuth = await googleUser.authentication;
      final AuthCredential credential = GoogleAuthProvider.getCredential(
        accessToken: googleAuth.accessToken,
        idToken: googleAuth.idToken,
      );

      AuthResult userRes = await this._auth.signInWithCredential(credential);
      FirebaseUser user = userRes.user;

      assert(!user.isAnonymous);
      assert(await user.getIdToken() != null);

      Firestore.instance.collection('users').document(user.uid).setData({
        'uid': user.uid,
        'firstname': user.displayName,
        'lastname': '',
        'photo': user.photoUrl,
        'email': user.email,
      });
      return new AuthResponse(true, 'signInWithGoogle succeeded: $user');
    } catch (e) {
      this._status = Status.Unauthenticated;
      notifyListeners();
      String message = 'User not found';
      if (e.code)
        switch (e) {
          case 'ERROR_INVALID_CREDENTIAL':
          case 'ERROR_ACCOUNT_EXISTS_WITH_DIFFERENT_CREDENTIAL':
          case 'ERROR_OPERATION_NOT_ALLOWED':
          case 'ERROR_INVALID_ACTION_CODE':
          case 'ERROR_USER_DISABLED':
            message = e.message;
            break;
          default:
            message = e.message;
            break;
        }

      return new AuthResponse(false, message);
    }
  }

  void signOutGoogle() async {
    await _googleSignIn.signOut();

    print('User Sign Out');
  }

  //#endregion google sign
}
```

Grace a ce provider, ont peut facilement gérer notre auth dans l'application et même améliorer notre UI grace auchangement de status
Nous enregistrons nos utilisateur dans notre de base de données car nous aurons besoin de les utiliser plus tard.

Firebase auth dispose d'une base de données distincte pour la gestion des connections d'utilisateurs

## Afficher une page en fonction de si l'utilisateur est authentifier ou non 

Dans notre fichier main, ou notre fichier splashscreen on a plusieurs façon de savoir si l'utilisateur actuel est auth ou pas

###### Utilisation de provider

Ayant initialisé des status, nous pouvons faire appel a eux pour déterminer notre page

```dart
///Le initState nous permet de lancé des fonction à l'initialisation de l'application
void initState() {
    super.initState();
    Future.delayed(Duration(seconds: 3), () {
      final _auth = Provider.of<AuthProvider>(context);

      Navigator.pushReplacementNamed(context,
          (_auth.status != Status.Authenticated) ? '/sign-in' : '/home');
    });
  }
```

###### Utilisation de l'instance de auth

Nous pouvons déterminé via l'instence s'il y a un utilisateur connecté

```dart
void initState() {
    super.initState();
    Future.delayed(Duration(seconds: 3), () {
      final _isLogged = AuthProvider.auth.currentUser();

      Navigator.pushReplacementNamed(context,
          (_isLogged != null) ? '/sign-in' : '/home');
    });
  }
```