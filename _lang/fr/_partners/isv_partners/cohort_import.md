---
nav_title: Intégration de l’importation de la cohorte
alias: /cohort_import/
hidden: true
---

# Intégration de l’importation de la cohorte de partenaire

> La fonctionnalité d’intégration d’importation de la cohorte de partenaire permet à nos partenaires de s’intégrer à Braze pour envoyer plus de cohortes d’utilisateurs générés dans l’application du partenaire.

## URL de cluster

Braze héberge notre application sur plusieurs clusters aux États-Unis et dans l’UE. L’URL des endpoints d’importation sera différente selon le cluster qui héberge l’instance de la société du client :

| INSTANCE | ENDPOINT REST |
| ----- | ------------------------------- |
| US-01 | `https://rest.iad-01.braze.com` |
| US-02 | `https://rest.iad-02.braze.com` |
| US-03 | `https://rest.iad-03.braze.com` |
| US-04 | `https://rest.iad-04.braze.com` |
| US-05 | `https://rest.iad-05.braze.com` |
| US-06 | `https://rest.iad-06.braze.com` |
| US-08 | `https://rest.iad-08.braze.com` |
| EU-01 | `https://rest.fra-01.braze.eu`  |
| EU-02 | `https://rest.fra-02.braze.eu`  |
{: .reset-td-br-1 .reset-td-br-2}

## URL de l’endpoint

Outre les URL de haut niveau spécifiques au cluster, chaque endpoint est également spécifique à un partenaire. Par exemple, lors de l’importation vers notre cluster US01, l’URL aurait le format `https://rest.iad-01.braze.com/partners/[partner_name]/…`, où `[partner_name]` est généralement le nom de l’entreprise du partenaire. Les spécificités de chaque endpoint sont décrites dans les sections suivantes.

## Authentification

Pour importer des données de cohorte dans Braze, deux clés d’authentification sont requises.

### Clé d’API partenaire

La clé d’API partenaire identifie le partenaire d’intégration et authentifie la demande comme étant valide pour l’importation. La clé doit être incluse dans le corps de la demande dans le champ `partner_api_key`.

Lors de la configuration de l’intégration dans l’application du partenaire, le client doit être invité à préciser son Cluster Braze afin que l’intégration sache quelle URL de Cluster et quelle clé d’API de partenaire utiliser lors de l’importation des données.

Braze fournira la ou les clés d’API partenaires au partenaire avant le début du développement de l’intégration du partenaire. En général, nous fournirons une clé unique valable pour tous les groupements américains, et une autre clé valable pour notre cluster européen.

### Clé d’importation des données client

La clé d’importation des données client identifie le groupe d’apps client dans lequel la cohorte doit être importée. La clé doit être incluse dans le corps de la demande dans le champ `client_secret`.

Cette clé est générée dans le tableau de bord du client dans les paramètres d’intégrations du partenaire. Lors de la configuration de l’intégration dans l’application du partenaire, le client doit être invité à indiquer sa clé d’importation des données afin que l’intégration sache à quel client et à quel groupe d’applications envoyer les données.

## Spécifications de l’endpoint API

### Endpoint du nom de la cohorte
L’Endpoint du nom de la cohorte peut être utilisé pour spécifier le nom d’une cohorte en fonction de son ID. Cet endpoint doit être appelé chaque fois qu’une cohorte est initialement exportée vers Braze, ou lorsque le nom d’une cohorte déjà connu de Braze est modifié.

| Champ | Type | Requis | Remarques |
| ----- | ---- | -------- | ----- |
| `partner_api_key` | String | Oui | Clé d’API spécifique au partenaire, utilisée dans toutes les demandes du partenaire à Braze. Cette clé sera spécifique au cluster (voir [Clé d’API du partenaire](#partner-api-key)), de sorte que le partenaire puisse connaître le cluster dans lequel les cohortes seront écrites. |
| `client_secret` | String | Oui | Clé d’importation des données du client propriétaire de la cohorte. |
| `cohort_id` | String | Oui | Identifiant de la cohorte. Cet identifiant doit être unique pour le client spécifié. |
| `nom` | String | Oui | Nom spécifié par le client pour la cohorte |
| `created_at` | String | Oui | Horodatage au format ISO-8601 |
{: .reset-td-br-1 .reset-td-br-2 .reset-td-br-3  .reset-td-br-4}

#### Exemple de demande :

`POST: https://rest.iad-01.braze.com/partners/[partner_name]/cohorts`
```
{
	“partner_api_key” : “123456-1234-1234-12345678”,
	“client_secret” : “234567-2345-2345-23456789”,
	“cohort_id” : “[un identifiant unique généré par le partenaire]”,
	“nom” : "Nom de la cohorte qui apparaîtra dans le tableau de bord de Braze",
	“created_at” : “2021-01-21T19:20:30+05:00”
}
```

### Endpoint de la cohorte d’utilisateurs

L’Endpoint de la cohorte d’utilisateurs permet de spécifier quels utilisateurs ont été ajoutés ou supprimés d’une cohorte particulière. Cet endpoint doit être appelé lorsqu’une cohorte est actualisée. Seuls les utilisateurs qui ont récemment saisi la cohorte ou qui ont quitté la cohorte depuis la dernière actualisation doivent être envoyés à Braze.

| Champ | Type | Requis | Remarques |
| ----- | ---- | -------- | ----- |
| `partner_api_key` | String | Oui | Clé d’API spécifique au partenaire, utilisée dans toutes les demandes du partenaire à Braze. Cette clé sera spécifique au cluster (voir [Clé d’API du partenaire](#partner-api-key)), de sorte que l’intégration puisse connaître le cluster dans lequel les cohortes seront écrites. |
| `client_secret` | String | Oui | Clé d’importation des données du client propriétaire de la cohorte. |
| `cohort_id` | String | Oui | Identifiant de la cohorte. L’identifiant doit être unique pour le client spécifié. |
| `cohort_changes` | Array d’objets | Oui | Les objets peuvent avoir deux champs. Un, `user_ids`, est obligatoire et peut être un tableau de `external_ids`, `device_ids` et d’`alias`. Chaque élément est un ID pour un utilisateur dont le statut dans la cohorte a changé. Le deuxième champ, `should_remove`, est un booléen facultatif indiquant si les utilisateurs de cet objet doivent être supprimés de la cohorte au lieu d’y être ajoutés. La valeur par défaut est fausse. Au début, nous ignorerons tous les ID qui ne correspondent pas à l’ID utilisateur externe pour un utilisateur, ce qui signifie que les utilisateurs anonymes ne peuvent pas être ajoutés ou supprimés d’une cohorte. La longueur combinée max. des ID utilisateur dans une seule demande est 1 000. Par exemple, si vous transmettez un ID d’appareil à un utilisateur identifié, nous ne l’ajouterons ou ne le supprimerons pas. Vous devez utiliser des ID externes pour les utilisateurs identifiés. |
{: .reset-td-br-1 .reset-td-br-2 .reset-td-br-3  .reset-td-br-4}

#### Exemple de demande :

`POST: https://rest.iad-01.braze.com/partners/[partner_name]/cohorts/users`
```
{
	“partner_api_key” : “123456-1234-1234-12345678”,
	“client_secret” : “234567-2345-2345-23456789”,
	“cohort_id” : “[un identifiant unique généré par le partenaire]”,
	"cohort_changes" : "[
	   {"user_ids": ["test_user_1", "test_user_2"]}
	]"
}
```

## Limitation du taux

En dehors du maximum de 1 000 ID utilisateur par demande dans l’Endpoint de la cohorte d’utilisateurs, ces endpoints ne sont pas spécifiquement limités.

## Filtre de cohorte

Braze ajoutera un filtre qui permet à un utilisateur de tableau de bord d’inclure ou d’exclure des utilisateurs d’une audience ciblée s’ils sont dans une cohorte de partenaire. Le filtre fournit une liste déroulante des noms de toutes les cohortes connues de Braze pour ce client. Ce filtre ne sera visible que par les clients avec lesquels le partenaire et Braze ont accepté de collaborer avec cette intégration.

## Résolution des problèmes

Reportez-vous au tableau suivant pour connaître les codes d’erreur spécifiques aux endpoints de l’importation de la cohorte et comment les résoudre.

| Code d’erreur | Description |
| ----- | ---- |
| `401` | Clé d’API partenaire non valide |
|  | Secret client non valide |
|  | Partenaire non activé pour le client avec secret client : **&#60;secret client&#62;** |
| `400` | `cohort_id` doit être une chaîne de caractères valide |
|  | `cohort_changes` doit être un ensemble d’objets avec une clé `user_ids` et/ou un mappage `device_ids` vers un tableau de chaînes de caractères ou un objet d’`alias` |
|  | Seulement 1 000 `user_ids`, `device_ids` et `alias` sont autorisés par demande |
|  | `nom` doit être une chaîne de caractères non vide |
|  | `created_at` doit être une heure valide au format [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) |
{: .reset-td-br-1 .reset-td-br-2}

Pour une résolution des problèmes supplémentaire, consultez [Erreurs et réponses]({{site.baseurl}}/api/errors/), qui couvrent les diverses erreurs et réponses du serveur qui peuvent apparaître lors de l’utilisation de l’API Braze.
