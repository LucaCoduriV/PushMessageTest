# Android message push

## But

Les messages push permettent de notifier une application de la récéption d'un message. Ils permettent par exemple d'envoyer des messages de notification pour stimuler le réengagement des utilisateurs ou alors tout simplement pour l'envoie de donnée.

Les notifications push ont été lancées pour la première fois en 2009 par Apple avec le "Apple Push Notification Service" et ont été introduites par Google en 2010 avec "Google Cloud to Device Messaging". Ce dernier a ensuite été fermé en 2015 pour être remplacé par Firebase cloud messaging.

Il faut quand même noter que les smartphones Android doivent possèder le services de Google installé (Google play service). Sans ce service la communication push n'est pas possible ainsi que tout les autres services proposé par Firebase. Il n'est pas rare d'avoir des smartphones en Asie qui posssède pas ce service et il faut donc trouver une autre manière d'envoyer des messages push, mais souvent celà ne fonctionne pas très bien à cause des restriction pour économiser la batterie.

## les problématiques qu’elle peut résoudre

Un point fort de ce service, est qu'il est entièrement gratuit. Il permet donc d'implémenter une communication d'un serveur aux smartphones sans frais et sans prise de tête.

Il permet de réduire la consommation de la batterie. En effet, si chaque application du smartphone contient une tâche de fond devant maintenir une connection tcp pour vérifier la récéption d'un message, la batterie ne tiendrais pas longtemps. Ce service est donc unique sur le smartphone et va tout simplement réveiller les applications qui doivent réaliser certaines tâches. De plus, réaliser une application souhaitant une fonctionnalité similaire aux messages push mais sans utiliser les services de Firebase devront se battre avec Android pour l'empêcher de les tuer. Les tâches ne sont pas forcément exécutée si l'appareil trouve qu'elle n'est pas utile ou qu'elle est trop gourmande et il est obligatoire d'afficher une notification pour avertir à l'utilisateur qu'une application tourne en fond et consomme donc la batterie.

## Alternatives

Il n'existe pas réellement d'alternative car tous les autres service passeront par Firebase pour afficher les messages push. Ce que les "alternatives" proposent est souvent uniquement une meilleure interface et analyse des données.

## comment est-ce qu’elle s’utilise

Les étapes suivante sont tirées de cette documentation : https://firebase.google.com/docs/android/setup

1.  Ajouter Firebase à son application

    -   Dans votre fichier Gradle au niveau de la racine (au niveau du projet) ( \<project>/build.gradle ), ajoutez le plug-in de services Google en tant que dépendance buildscript :

    ```Gradle
    buildscript {

     repositories {
       // Make sure that you have the following two repositories
       google()  // Google's Maven repository
       mavenCentral()  // Maven Central repository
     }

     dependencies {
       ...

       // Add the dependency for the Google services Gradle plugin
       classpath 'com.google.gms:google-services:4.3.14'
     }
    }
    ```

    -   Dans le fichier Gradle de votre module (au niveau de l'application) (généralement \<project>/\<app-module>/build.gradle ), ajoutez le plug-in de services Google :

    ```Gradle
    plugins {
    id 'com.android.application'

    // Add the Google services Gradle plugin
    id 'com.google.gms.google-services'
    ...
    }
    ```

    -   Dans le fichier Gradle de votre module (au niveau de l'application) (généralement \<project>/\<app-module>/build.gradle ), ajoutez les dépendances des produits Firebase que vous souhaitez utiliser dans votre application. Il est recommandé d'utiliser Firebase Android BoM pour contrôler la gestion des versions de la bibliothèque.

    ```Gradle
    dependencies {
    // ...

    // Import the Firebase BoM
    implementation platform('com.google.firebase:firebase-bom:31.1.1')

    // When using the BoM, you don't specify versions in Firebase library dependencies

    // Add the dependency for the Firebase Cloud Messaging
    implementation 'com.google.firebase:firebase-messaging-ktx'

    }
    ```

    -   Il faut ensuite ajouter les fichier de configuration firebase que nous allons ajouter à la prochaine étape.

2.  créer un projet Firebase
    -   Pour pouvoir utiliser Firebase il est nécessaire de vous créer un compte afin d'accéder à cette page: https://console.firebase.google.com/
    -   Maintenant que vous avez accès à la console, créez un nouveau projet:
        ![](Screenshot_1.png)
    -   Saisissez le nom de votre projet puis cliquez continuer 2x et choisissez le compte que vous souhaiter utiliser pour les Google Analytics. Vous aurez maintenant accès à la console de votre projet. Depuis cette console, il est possible d'utiliser les différents services que propose Firebase.
    -   Il faut maintenant importer la configuration dans notre projet Android. Pour cela cliquez sur l'icone android sur la page d'acceuil.![](Screenshot_2.png)
    -   Remplissez à présent les champs obligatoire demandé durant l'assistant de configuration. Veillez à mettre un nom de package qui correspond bien à celui de votre application.
    -   Lorsqu'il vous sera demandé téléchargez le fichier google-services.json et mettez le dans le dossier `./app/` de votre projet. Il permet à votre application de communiquer avec votre projet firebase.
    -   Vous pouvez ensuite sauter les étapes d'après car nous l'avons déjà fais dans la première partie.
3.  Ajouter un service Cloud Messaging à son application
    Nous allons maintenant ajouter le service qui va être utilisé pour réagir aux messages push. Pour cela nous allons ajouter ces lignes à l'intérieur du tag `\<application>` dans notre fichier manifest se trouvant dans `./app/src/main/AndroidManifest.xml`

    ```xml
    <service
        android:name=".MyService"
        android:exported="false">
        <intent-filter>
            <action android:name="com.google.firebase.MESSAGING_EVENT" />
        </intent-filter>
    </service>
    ```

    Veuillez noter que `MyService` correspond à la classe que l'on va utiliser.
    Vous remarquerez aussi que l'on ajoute une intention qui est que lors de la récéption d'un message venant de firebase le service doit être reveillé.

    Maintenant il faut que l'on ajoute la classe MyService, pour ce faire ajoutez les lignes suivante dans un fichier `app\src\main\java\com.domaine.nom\MyService.kt`.

    ```kotlin
    class MyService : FirebaseMessagingService() {
        override fun onNewToken(token: String) {
            super.onNewToken(token)
            Log.i("MYTAG", token)
        }


        override fun onMessageReceived(message: RemoteMessage) {

            if(message.data["test"] != null){
                Log.i("MYTAG", message.data["test"].toString())
                return super.onMessageReceived(message);
            }


            super.onMessageReceived(message)
        }
    }
    ```

    Dans le code ci-dessus la fonction onNewToken est utilisez pour récupérer le token utilisé pour identifier votre appareil. Ce token peut être renouvelé dans certains cas. Ce token est utile lorsque vous souhaitez cibler un appareil en particulier.

    La fonction onMessageReceived est appelée lors de la récéption d'un nouveau message. Que votre application soit ouverte, en arrière plan ou alors fermée cette fonction sera toujours appelée. Je dois avouez que en vous disant ça je ne vous dis pas totalement la vérité. Il existe deux type de message push, les data messages et les notifications push. Ce que j'ai dis est vrai dans le premier cas. En revanche lors du deuxième cas la fonction `onMessageReceived` ne sera appelé que lorsque l'application est ouverte. Cela vous permet de décider comment vous souhaitez afficher cette notification. Les notifications push ne sont pas géré par cette fonction lorsque votre application est en arrière plan ou fermée, une notification est tout simplement affiché sur le téléphone.

    Vous trouverez plus de détails à propos de cette classe sur cette page: https://firebase.google.com/docs/reference/android/com/google/firebase/messaging/FirebaseMessagingService

4.  Envoyer les messages push

Pour envoyer un data message par une requete post:

header:

-   Content-Type: application/json
-   Authorization: key=<your-server-key>

body:

```json
{
    "data": {
        "test": "coucou"
    },
    "to": "target-token"
}
```

## limitations

-   Certain constructeur n'hésite pas à restreindre les applications dans le but de faire durer la batterie plus longtemps, par conséquent il est possible que les notifications/messages ne soient pas reçu par l'application.
-   La taille des messages est limité

## points d’attention