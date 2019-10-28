# Admobs (à compléter)

> Mettre de la pub sur notre application

## Prérequis

###### Android X

Afin de pouvoir utiliser les service de google, il faut impérativement une application flutter sous androidX

```dart
    flutter create --androix project_name
```

###### Avoir générer un projet sur firebase

[Firebase](firebase.md)

###### Ajouter le package correspondant dans le pubscec.yaml

```yaml
    firebase_admob: ^0.9.0+7
```

###### Avoir un compte sur admob

Désactiver le bloquer de pub
aller sur [https://apps.admob.com/v2/home](https://apps.admob.com/v2/home)

###### Ajouter dans le fichier android/app/src/main/androidManifest.xml

```xml
 <meta-data
  android:name="com.google.android.gms.ads.APPLICATION_ID"
  android:value="[admob_project_token]"
/>
```

## Utilisation

```dart
    import 'package:firebase_admob/firebase_admob.dart';
import 'package:flutter/material.dart';

class Ads extends StatefulWidget {
  @override
  _AdsState createState() => _AdsState();
}

class _AdsState extends State<Ads> {
  MobileAdTargetingInfo targetingInfo = MobileAdTargetingInfo(
    keywords: <String>['flutterio', 'beautiful apps'],
    contentUrl: 'https://flutter.io',
    birthday: DateTime.now(),
    childDirected: false,
    designedForFamilies: false,
    gender: MobileAdGender.male, // or MobileAdGender.female, MobileAdGender.unknown
    testDevices: <String>[], // Android emulators are considered test devices
  );

  BannerAd _bannerAd;
  BannerAd createBannerAd(){
    return BannerAd(
      adUnitId: BannerAd.testAdUnitId,
      size: AdSize.banner,
      targetingInfo: targetingInfo,
      listener: (MobileAdEvent event){
        print('BannerAd $event'); ///print error
      }
    );
  }

  @override
  void initState() {
    FirebaseAdMob.instance.initialize(appId: BannerAd.testAdUnitId); ///code de test générique
    _bannerAd = createBannerAd()..load()..show(); /// il faut charger la pub 
    // FirebaseAdMob.instance.initialize(appId: '[admob_project_token]');
    super.initState();
  }

  @override
  void dispose(){
    _bannerAd.dispose();
    super.dispose();
  }


  @override
  Widget build(BuildContext context) {
    return Container(
      
    );
  }
}
```