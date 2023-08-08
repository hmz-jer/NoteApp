Refonte de l'application
1. Introduction
L'objectif de ce document est de décrire le fonctionnement actuel d'une application Java avant sa refonte et son intégration dans un API Gateway. Cette application fournit un service de scoring et de prépaiement des transactions.
2. Détails des fonctionnalités
2.1 Traitement applicatif des messages

2. Détails des fonctionnalités
2.1 Traitement applicatif des messages
2.1.1 Traitement à la réception de la requête jusqu'à l'envoi à ISP
    • Entrée:
        ◦ Paramètres d'entrée:
            ▪ Cust_CNX_ID:
                • Type: Alphanumérique
                • Taille: 12
                • Exemple de valeur: (Si vous pouvez fournir un exemple, ce serait utile.)
            ▪ ORIGINATOR_ID:
                • Type: Alphanumérique
                • Taille: 12
                • Exemple de valeur: (Exemple à fournir)
            ▪ APP_SERVICE:
                • Type: Alphanumérique
                • Taille: 12
                • Exemple de valeur: (Exemple à fournir)
            ▪ APP_SERVICE_VERSION:
                • Type: Alphanumérique
                • Taille: 12
                • Exemple de valeur: (Exemple à fournir)
            ▪ CUST_MSG_ID:
                • Type: Alphanumérique
                • Taille: 12
                • Exemple de valeur: (Exemple à fournir)
            ▪ MSG_CONTENT:
                • Type: Alphanumérique (compressé avec GZIP et encodé en Base64)
                • Taille: Paramétrable
                • Exemple de valeur: (Exemple à fournir)
            ▪ MSG_CONTENT_FORMAT:
                • Type: Alphanumérique
                • Taille: 12
                • Exemple de valeur: (Exemple à fournir)
    • Traitement:
        ◦ Validation de l'existence de chaque paramètre d'entrée.
        ◦ Validation du type de chaque paramètre.
        ◦ Validation de la taille de chaque paramètre, en tenant compte des configurations spécifiques pour MSG_CONTENT.
        ◦ Compression du MSG_CONTENT avec GZIP.
        ◦ Encodage du MSG_CONTENT compressé en Base64.
        ◦ Si toutes les validations sont réussies, décompression du MSG_CONTENT.
        ◦ Récupération du champ TpOfScgReq depuis le MSG_CONTENT décompressé.
        ◦ Contrôle de la cohérence entre le message à scorer (TpOfScgReq) et l'appService sollicité (APP_SERVICE). Si les champs ne sont pas égaux, rejet du message avec le code d'erreur 1401.
        ◦ Récupération de l'IBAN depuis le MSG_CONTENT.
        ◦ Application du HMAC (RFC 2104) avec la fonction de hachage SHA256 pour obtenir un alias pour l'IBAN. La clé secrète utilisée a une taille de 256 bits. L'alias généré a une taille de 32 octets.
        ◦ Remplacement de l'IBAN dans le MSG_CONTENT par l'aliasIBAN.
        ◦ Ajout de l'attribut Message = "ModelRequest" pour indiquer qu'il s'agit d'une requête.
        ◦ Récupération du MSGID depuis le MSG_CONTENT pour l'ID unique de la transaction.
        ◦ Détermination du MESSAGETYPE_ID en fonction de TpOfScgReq selon le tableau de correspondance. (Veuillez fournir le tableau ou quelques exemples de correspondance.)
        ◦ Préparation de l'en-tête pour ISP avec les attributs et valeurs appropriés.
        ◦ 
        ◦ Envoi du MSG_CONTENT modifié à ISP.
    • Sortie:
        ◦ En cas d'incohérence entre TpOfScgReq et APP_SERVICE, le message est rejeté avec le code d'erreur 1401.
        ◦ Si le message est accepté, l'IBAN haché est produit comme sortie et le MSG_CONTENT modifié est envoyé à ISP.
2.1.1 Traitement à la réception de la requête jusqu'à l'envoi à ISP
    • Entrée:
        ◦ Paramètres d'entrée: (Détails des paramètres précédemment mentionnés)
    • Traitement: (Étapes de traitement précédemment mentionnées)
    • Sortie: (Informations de sortie précédemment mentionnées)
2.2 Traitement des réponses d'ISP vers l'API Gateway
2.2.1 Réception de la réponse d'ISP
    • Entrée:
        ◦ Réponse d'ISP, qui contient notamment:
            ▪ MSGID_Response: L'identifiant unique de la réponse.
            ▪ ErrorCode: Code d'erreur retourné par ISP.
            ▪ ... (autres champs pertinents de la réponse d'ISP à spécifier)
    • Traitement:
        ◦ Vérification de la présence de l'identifiant de message (MSGID_Response) dans la réponse.
        ◦ Comparaison de l'MSGID_Response avec l'MSGID de la demande initiale pour s'assurer qu'ils sont identiques.
        ◦ Vérification de l'attribut ErrorCode pour s'assurer qu'il est égal à zéro.
        ◦ Suppression de l'en-tête d'ISP de la réponse.
        ◦ Si la réponse dépasse la taille autorisée, suppression des règles de calcul.
        ◦ Compression du MSG_CONTENT avec GZIP.
        ◦ Encodage du MSG_CONTENT compressé en Base64.
    • Sortie:
        ◦ MSG_CONTENT modifié, compressé et encodé en Base64, prêt à être transmis ou traité davantage selon les besoins.
    • 
3. Génération des Tickets d'Usage (TU)
3.1 Description
Les Tickets d'Usage (TU) jouent un rôle crucial dans le suivi des SLA (Service Level Agreements) au sein du système. Chaque TU est généré pour chaque transaction et contient des détails clés sur cette transaction, y compris des informations liées à la requête initiale et à la réponse reçue.
3.2 Attributs d'un TU
    • Détails de la requête:
        ◦ Identifiant de la requête
        ◦ Timestamp de la requête
        ◦ Source de la requête
        ◦ etc.
    • Détails de la réponse:
        ◦ Identifiant de la réponse
        ◦ Timestamp de la réponse
        ◦ Statut de la réponse
        ◦ etc.
3.3 Processus de génération
3.3.1 De l'API Gateway vers ISP

    1. Étapes:
        1. Capture des détails de la requête et de la réponse.
        2. Inclusion de l'en-tête comme décrit dans le DIA.
        3. Suppression du champ MsgContent du TU.
        4. Consolidation des informations restantes dans le format TU.
        5. Stockage ou transmission du TU pour le suivi du SLA.
    2. Sortie:
        1. Un TU généré sans le champ MsgContent.
3.3.2 D'ISP vers l'API Gateway
    1. Déclencheur: (À préciser)
    2. Étapes:
        1. Capture des détails de la requête et de la réponse.
        2. Inclusion du champ MsgContent dans le TU.
        3. Aucune suppression de l'en-tête ISP.
        4. Consolidation des informations dans le format TU.
        5. Stockage ou transmission du TU pour le suivi du SLA.
    3. Sortie:
        1. Un TU complet contenant le champ MsgContent et l'en-tête ISP.


3.4 Attributs détaillés du Ticket d'Usage (TU)
Dans cette section, nous détaillerons chaque attribut du TU, son alimentation et sa condition de présence.
Attribut
Alimentation du champ
Condition de présence
(Espace réservé pour l'attribut)
(Espace réservé pour l'alimentation du champ)
(Espace réservé pour la condition de présence)
...
...
...

3.5 Conditions de gestion des fichiers de TU
Pour assurer une gestion efficace des fichiers de TU, certaines conditions peuvent déclencher la fermeture d'un fichier en cours et l'ouverture d'un nouveau. Ces conditions sont :
    1. Nombre d'enregistrements : Un fichier peut contenir un maximum de 90 000 TU. Lorsque cette limite est atteinte, le fichier en cours est fermé et un nouveau fichier est ouvert pour les TU suivants.
    2. Taille globale du fichier : La taille maximale d'un fichier de TU est de 8 100 octets. Si cette taille est atteinte, le fichier actuel est fermé et un nouveau est ouvert pour continuer à enregistrer les TU.
    3. Temps écoulé : Si une heure s'écoule depuis l'ouverture du fichier actuel, ce fichier est alors fermé, indépendamment du nombre d'enregistrements ou de sa taille, et un nouveau fichier est ouvert pour les TU suivants.
