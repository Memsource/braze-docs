---
nav_title: "POST : Mettre à jour les messages planifiés"
article_title: "POST : Mettre à jour les messages planifiés"
search_tag: Endpoint
page_order: 4
layout: api_page
page_type: reference
description: "Cet article présente en détail l’endpoint Braze Mettre à jour les messages planifiés."

---
{% api %}
# Mettre à jour les messages planifiés
{% apimethod post core_endpoint|https://www.braze.com/docs/core_endpoints %} 
/messages/schedule/update
{% endapimethod %}

Utilisez cet endpoint pour mettre à jour les messages planifiés. Cet endpoint accepte les mises à jour du paramètre `schedule` ou du paramètre `messages` ou des deux. Votre demande doit contenir au moins une des deux clés.

{% apiref postman %}https://documenter.getpostman.com/view/4689407/SVYrsdsG?version=latest#f61edf74-4467-4551-b9c4-a4b8d188cd7a {% endapiref %}

## Limites de débit

{% multi_lang_include rate_limits.md endpoint='default' %}

## Corps de la demande

```
Content-Type: application/json
Authorization: Bearer YOUR-REST-API-KEY
```

```json
{
  "schedule_id": (required, string) Le `schedule_id` à mettre à jour (obtenu à partir de la réponse pour créer une planification).,
  "schedule": {
    // optionnel, voir la documentation de planification
  },
  "messages": {
    // optionnel, voir la documentation d’objets d’envoi de messages disponible
  }
}
```
## Paramètres de demande

| Paramètre | Requis | Type de données | Description |
| --------- | ---------| --------- | ----------- |
| `schedule_id` | Requis | String | Le `schedule_id` à mettre à jour (obtenu à partir de la réponse pour créer une planification). |
|`schedule` | Facultatif | Objet | Voir [Objet de planification]({{site.baseurl}}/api/objects_filters/schedule_object/). |
|`messages` | Facultatif | Objet | Voir [Objets de messagerie disponibles]({{site.baseurl}}/api/objects_filters/#messaging-objects). |
{: .reset-td-br-1 .reset-td-br-2 .reset-td-br-3  .reset-td-br-4}

## Exemple de demande
```
curl --location --request POST 'https://rest.iad-01.braze.com/messages/schedule/update' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer YOUR-REST-API-KEY' \
--data-raw '{
  "schedule_id": "schedule_identifier",
  "schedule": {
    "time": "2017-05-24T20:30:36Z"
   },
  "messages": {
    "apple_push": {
      "alert": "Updated Message!",
      "badge": 1
    },
    "android_push": {
      "title": "Titre mis à jour !",
      "alert": "Message mis à jour !"
    },
    "sms": {  
      "subscription_group_id": "subscription_group_identifier",
      "message_variation_id": "message_variation_identifier",
      "body": "Ceci est mon corps de SMS.",
      "app_id": "app_identifier"
    }
  }
}'
```

{% endapi %}
