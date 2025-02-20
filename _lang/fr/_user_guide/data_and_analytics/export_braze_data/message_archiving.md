---
nav_title: Archivage des messages
article_title: Archivage des messages
alias: "/message_archiving/"
page_order: 0
page_type: reference
description: "Cet article de référence couvre l’archivage des messages, une fonctionnalité qui vous permet d’enregistrer une copie du message envoyé aux utilisateurs."

---

# Archivage des messages

> La fonction Archivage des messages vous permet d’enregistrer une copie des messages envoyés aux utilisateurs à des fins d’archivage ou de conformité dans votre compartiment S3.

## Conditions préalables

1. Vous avez acheté cette fonction et votre CSM a activé la fonction.
2. Vous avez connecté un compartiment S3 à votre groupe d’apps via la page **Technology Partners**.

## Overview

Lorsque cette fonction est activée et que vous avez connecté avec succès un compartiment S3 via la page **Technology Partners** de votre groupe d'apps, Braze écrira un fichier JSON compressé dans votre compartiment S3 pour chaque e-mail, SMS ou message de notification push envoyé à un utilisateur. Votre CSM peut activer tous les canaux à enregistrer.

Ce fichier contient les champs définis dans [Références de fichiers](#file-references) et reflète les messages types finaux envoyés à l’utilisateur. Chaque valeur modélisée définie dans votre campagne (par ex., {% raw %}`{{${first_name}}}`{% endraw %}) affichera la valeur finale que l’utilisateur a reçue selon ses informations de profil. Cela vous permettra de conserver une copie du message envoyé pour satisfaire les exigences de conformité, d’audit ou d’assistance aux clients. 

Le fichier sera enregistré dans votre compartiment S3 en utilisant la structure clé suivante :<br>
`sent_messages/Channel/(md5Ofe164PhoneOrEmailOrPushToken)/CampaignId/DispatchId.json.gz`

Un fichier exemple peut ressembler à ceci :<br>
`sent_messages/email/819baa08d8d7e77e19d4666f5fc6050b/wxyz-lmno/651fd10b282850b39e1169c13975234b.json.gz`

{% alert important %}
L’activation de cette fonction aura un impact sur la vitesse d’envoi/de réception de vos messages car le téléchargement du fichier S3 est effectué immédiatement avant que le message ne soit envoyé pour des raisons de vérification. Cela introduit une latence supplémentaire dans le pipeline d'envoi de Braze, ce qui affecte la vitesse d'envoi.
{% endalert %}

## Références de fichier

Vous trouverez ci-dessous des références de charge utile JSON apportées à votre compartiment S3 chaque fois qu’un message est envoyé :

### E-mail
```json
{
  “version”: 1, //version numérique de la structure JSON
  "to": ToAddress, ("customer@example.com")
  "subject": SubjectLine ("Contient un bon de réduction de 20 % !"),
  "from_name": DisplayName ("Braze"),
  "from_address": FromAddress ("no-reply@braze.com"),
  "html_body": HtmlBody,
  "plaintext_body": PlainTextBody,
  "amp_body": AMPEmailBody,
  "extras": HashOfKVP,
  "headers": HashOfHeaders,
  "sent_at": UnixTimestamp,
  "dispatch_id": DispatchIdFromBraze,
  "campaign_id": CampaignApiKey, // peut ne pas être disponible
  "canvas_id": CanvasApiKey, // peut ne pas être disponible
  "message_variation_id": MessagVariationId, // peut ne pas être disponible
}
```

### SMS
```json
{
  “version”: 1 //version numérique de la structure JSON
  "to": PhoneNumber, ("+15555555555"),
  "body": Body ("Salut !"),
  "subscription_group": SubscriptionGroupExternalId,
  "provider": StringOfProviderName,
  "media_urls": ArrayOfString,
  "sent_at": UnixTimestamp,
  "dispatch_id": DispatchIdFromBraze,
  "campaign_id": CampaignApiKey, // peut ne pas être disponible
  "canvas_id": CanvasApiKey, // peut ne pas être disponible
  "message_variation_id": MessagVariationId, // peut ne pas être disponible
}
```

### Notification push
```json
{
  “version”: 1, //version numérique de la structure JSON
  "to": PushToken,
  "payload": JsonOfEntirePushPayload,
  "platform": ios/android/web/kindle,
  "app_id": ApiKeyOfApp,
  "sent_at": UnixTimestamp,
  "dispatch_id": DispatchIdFromBraze,
  "campaign_id": CampaignApiKey, // peut ne pas être disponible
  "canvas_id": CanvasApiKey, // peut ne pas être disponible
  "message_variation_id": MessagVariationId, // peut ne pas être disponible
}
```

## Foire aux questions

### Quelle création de modèle n’est pas incluse dans la charge utile ? 
Les modifications apportées une fois le message expédié de Braze ne seront pas reflétées dans le fichier enregistré dans votre compartiment S3. Seront incluses des modifications apportées par nos partenaires de remise du courrier, comme des liens d’habillage pour le suivi des clics et l’insertion des pixels de suivi. 

### Quels sont les messages sous la valeur « non associé » dans le parcours de campagne ? 
Lorsqu’un message est envoyé en dehors d’une campagne ou de Canvas, l’ID de campagne dans le nom du fichier sera « non associé ». Cela se produit lorsque vous envoyez des messages test à partir du tableau de bord, lorsque Braze envoie des réponses automatiques SMS, ou lorsque les messages envoyés via l’API ne spécifient pas d’ID de campagne.

### Comment faire pour en savoir plus sur cet envoi ? 
À l'aide de l’ID d'envoi, vous pouvez vérifier le message modèle avec nos données Currents pour trouver plus d'informations, notamment l’ID de l'utilisateur externe qui l'a reçu, l'horodatage de livraison, si l'utilisateur a ouvert ou cliqué sur le message, et bien plus encore. 

### Comment les nouvelles tentatives sont-elles traitées ? 
Si votre compartiment S3 n’est pas joignable, Braze réessaiera jusqu’à trois fois avec une [gigue d’interruption](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/#Jitter). Les limites de débit S3 sont automatiquement gérées par Braze.

### Que se passe-t-il si mes identifiants ne sont pas valides ? 
Si vos informations d’identification S3 deviennent invalides à tout moment, Braze ne pourra enregistrer aucun message dans votre compartiment S3, et ces messages seront perdus. Nous vous recommandons de configurer les préférences de [notifications d’erreurs relatives aux informations d’identification AWS]({{site.baseurl}}/user_guide/administrative/company_settings/notification_preferences) pour vous assurer de recevoir des alertes en cas de problème d’identification.

### Pourquoi l’horodatage de mon fichier d’archive "sent_at" diffère légèrement de l’horodatage envoyé dans Currents ? 
La copie S3 obtenue est enregistrée immédiatement avant d’être envoyée à l’utilisateur final. En raison des durées d’envoi S3, il peut y avoir un délai de quelques secondes entre l’horodatage « envoyé_à » à la copie avec rendu et la durée réelle de l’envoi.
