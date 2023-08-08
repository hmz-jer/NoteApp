
## Backlog de refonte de l'application

### Épic 1 : Traitement applicatif des messages

1. **Tâche 1.1** : Mise en place de l'API Gateway pour recevoir les requêtes.
2. **Tâche 1.2** : Validation des paramètres d'entrée (vérification de l'existence, du type et de la taille).
3. **Tâche 1.3** : Compression et décompression du `MSG_CONTENT` en utilisant GZIP.
4. **Tâche 1.4** : Encodage et décodage du `MSG_CONTENT` en Base64.
5. **Tâche 1.5** : Comparaison des champs `APP_SERVICE` et `TpOfScgReq` pour assurer la cohérence.
6. **Tâche 1.6** : Hashage de l'IBAN en utilisant HMAC-SHA256.
7. **Tâche 1.7** : Remplacement de l'IBAN par l'aliasIBAN dans `MSG_CONTENT`.
8. **Tâche 1.8** : Ajout des attributs `MESSAGETYPE_ID` et `Message`.
9. **Tâche 1.9** : Préparation de l'en-tête pour l'envoi à ISP et ajout du `MTID` (Message Type Identifier).
10. **Tâche 1.10** : Horodatage du message lors de l'envoi à ISP.

### Épic 2 : Gestion des réponses d'ISP

1. **Tâche 2.1** : Réception de la réponse d'ISP.
2. **Tâche 2.2** : Vérification de l'attribut `ErrorCode` dans la réponse d'ISP.
3. **Tâche 2.3** : Suppression de l'en-tête d'ISP de la réponse pour les réponses allant de l'API Gateway vers ISP.
4. **Tâche 2.4** : Compression et encodage du `MSG_CONTENT` pour les réponses d'ISP à l'API Gateway.
5. **Tâche 2.5** : Horodatage du message lors de la réception de la réponse d'ISP.

### Épic 3 : Génération des Tickets d'Usage (TU)

1. **Tâche 3.1** : Définition du format et des attributs pour les TU.
2. **Tâche 3.2** : Capture des détails de la requête et de la réponse pour la génération des TU.
3. **Tâche 3.3** : Gestion de la fermeture et de l'ouverture des fichiers de TU en fonction des conditions spécifiées.
4. **Tâche 3.4** : Sécurisation des données spécifiques dans le champ `MsgContent` des TU, selon les directives de Sam.
