---
nav_title: Badges
article_title: Badges du fil d’actualités pour iOS
platform: iOS
page_order: 3
description: "Cet article explique comment implémenter le nombre de badges du fil d’actualités dans votre application iOS."
channel:
  - Fil d’actualité

---

# Badges

## Demande de décompte de cartes de fil d’actualités non lues

![][45]{: style="float:right;max-width:25%;margin-left:15px;"}

Les badges constituent un excellent moyen d’attirer l’attention sur le nouveau contenu en attente de vos utilisateurs dans le Fil d’actualité. Si vous souhaitez ajouter un badge à votre fil d’actualité, le SDK de Braze fournit des méthodes permettant d’interroger les éléments suivants :

- Les cartes de fil d’actualité non lues pour l’utilisateur actuel
- Total des cartes de fil d’actualité visibles pour l’utilisateur actuel

Les déclarations de méthodes suivantes dans [`ABKFeedController`][44] décrivent cela en détail :

```
- (NSInteger)unreadCardCountForCategories:(ABKCardCategory)categories;
/*
La méthode renvoie le nombre de Cartes de contenu actuellement active qui n’ont pas été visualisées.
Une « vue » survient lorsqu’une carte devient visible dans la vue Cartes de contenu. Ceci permet de faire la différence entre les cartes qui sont hors écran dans la vue de défilement et celles qui sont à l’écran, lorsqu’une carte défile sur l’écran, elle est comptabilisée comme visualisée.
Les cartes sont comptabilisées comme visualisées une seule fois – si une carte défile hors de l’écran et y revient, elle n’est pas re-comptabilisée.
Les cartes ne sont comptabilisées qu’une seule fois, même si elles s’affichent dans plusieurs vues de Cartes de contenu ou sur plusieurs appareils.
*/

- (NSInteger)cardCountForCategories:(ABKCardCategory)categories;
/* 
La méthode renvoie le nombre total de Cartes de contenu actuellement actives. Les cartes ne sont comptabilisées qu’une seule fois, même si elles s’affichent dans plusieurs vues de Cartes de contenu.
 */
 ```

## Afficher le nombre d’articles du fil d’actualités non lus pour le nombre de badges d’application

En plus de servir de rappels de notification push pour une application, les badges peuvent également spécifier des éléments non consultés dans le fil d’actualité de l’utilisateur. La mise à jour du nombre de badges basée sur les mises à jour du fil d’actualité non lu peut constituer un outil précieux pour attirer les utilisateurs vers votre application et augmenter les sessions.

Appelez cette méthode qui enregistre le nombre de badges une fois l’application fermée et la session de l’utilisateur terminée :

{% tabs %}
{% tab OBJECTIVE-C %}

```objc
(void)applicationDidEnterBackground:(UIApplication *)application
```

{% endtab %}
{% tab swift %}

```swift
func applicationDidEnterBackground(_ application: UIApplication)
```

{% endtab %}
{% endtabs %}

Pour cette méthode, implémentez le code suivant, qui actualise activement le nombre de badges alors que l’utilisateur visualise les cartes pendant une session donnée.

{% tabs %}
{% tab OBJECTIVE-C %}

```objc
[UIApplication sharedApplication].applicationIconBadgeNumber = [[Appboy sharedInstance].feedController unreadCardCountForCategories:ABKCardCategoryAll];
```

{% endtab %}
{% tab swift %}

```swift
UIApplication.shared.applicationIconBadgeNumber = Appboy.sharedInstance()?.feedController.unreadCardCount(forCategories: ABKCardCategory.all) ?? 0
```

{% endtab %}
{% endtabs %}

À tout moment, par exemple, avec la méthode `applicationDidBecomeActive`, utilisez le code suivant pour effacer le nombre de badges :

{% tabs %}
{% tab OBJECTIVE-C %}

```objc
[UIApplication sharedApplication].applicationIconBadgeNumber = 0;
```

{% endtab %}
{% tab swift %}

```swift
UIApplication.shared.applicationIconBadgeNumber = 0
```

{% endtab %}
{% endtabs %}

Pour plus d’informations, voir le `Appboy.h` [fichier d’en-tête][15].

[15]: https://github.com/Appboy/appboy-ios-sdk/blob/master/AppboyKit/include/Appboy.h "Appboy.h Header File"
[42]: {% image_buster /assets/img_archive/badge_example.png %} "Badge Example"
[44]: http://appboy.github.io/appboy-ios-sdk/docs/interface_a_b_k_feed_controller.html "abk feed controller"
[45]: {% image_buster /assets/img_archive/newsfeed_badges.png %}