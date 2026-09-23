# Audit — technique

## 1. Entraînement, validation et reproductibilité

| Type de problème | Constat observé | Risque / conséquence |
|---|---|---|
| Validation du modèle | Le modèle est entraîné sur l'intégralité du dataset et évalué sur ces mêmes données. | Performance optimiste ; capacité de généralisation non démontrée. |
| Gestion d'erreurs d'entraînement et de persistance | Le script ne contrôle pas explicitement la lecture du dataset, son schéma, l'échec de l'entraînement, la mémoire disponible ou l'écriture du fichier `.joblib`. | Entraînement incomplet, modèle non généré ou échec difficile à diagnostiquer. |
| Architecture du pipeline | Les scripts sont monolithiques ; la préparation des données n'est pas encapsulée dans un pipeline partagé. | Risque de divergence entre les variables d'entraînement et de prédiction ; reproductibilité limitée. |
| Seuil de décision | Le seuil de classification est fixé à `0.5`, sans justification ni configuration visible. | Décisions fondées sur un paramètre non documenté et non traçable. |
|  | Les tests disponibles vérifient uniquement le démarrage (données lisibles, modèle chargeable, script exécutable) ; aucune couverture visible des entrées invalides, des cas limites, des régressions ou du comportement métier. |
| Tests | Les tests disponibles vérifient uniquement le démarrage (données lisibles, modèle chargeable, script exécutable) ; aucune couverture visible des entrées invalides, des cas limites, des régressions ou du comportement métier. | Régressions fonctionnelles et erreurs de validation non détectées avant déploiement ; fiabilité du comportement attendue non démontrée. |

## 2. Sécurité

| Type de problème | Constat observé | Risque / conséquence |
|---|---|---|
| Secret | Un mot de passe de base de données est inscrit en clair dans `legacy/train.py`. | Divulgation possible via le dépôt, une copie du script ou des journaux. |
| Validation des entrées | Le script de prédiction ne contrôle ni les plages de valeurs, ni les types métier, ni un schéma d'entrée. | Prédictions incohérentes ou arrêt du script pour des valeurs invalides. |
| Architecture d'appel | La prédiction est lancée par SSH et dépend d'une exécution depuis la racine du projet pour charger le modèle via un chemin relatif. | Stratégie fragile ; échec possible en cas de connexion SSH indisponible, de répertoire incorrect, de droits insuffisants ou de changement de chemin. |

## 3. Déploiement et exploitation

| Type de problème | Constat observé | Risque / conséquence |
|---|---|---|
| Déploiement | Le déploiement est manuel par SCP ; aucun versionnement ni métadonnée du modèle ne sont visibles. | Difficulté à identifier le modèle effectivement déployé, à tracer un résultat ou à rejouer une prédiction. |
| Chargement du modèle | Le script charge le fichier `.joblib` à chaque exécution. | Latence supplémentaire et accès disque répété à chaque requête ; impact dépendant du volume et du temps de réponse attendu. |
| Évolutivité et supervision | L'exécution est synchrone et aucune supervision, mesure de charge ou alerte en temps réel n'est visible. | Montée en charge difficile à anticiper et détection tardive des indisponibilités, latences anormales ou erreurs répétées. |

## 4. Traçabilité et auditabilité

| Type de problème | Constat observé | Risque / conséquence |
|---|---|---|
| Journalisation | Les requêtes, probabilités, versions de modèle et erreurs ne sont pas journalisées. | Impossible de retracer ou rejouer une prédiction dans son contexte, d'auditer un incident ou de détecter une dérive. |
| Explicabilité | Aucune analyse des variables influençant les prédictions, ni explication locale ou globale du modèle, n'est fournie. | Les professionnels ne peuvent pas comprendre les facteurs ayant conduit au score, contester une prédiction ou vérifier sa cohérence. |

## 5. Continuité de service

| Question ouverte | Risque conditionnel |
|---|---|
| L'infrastructure de production repose-t-elle sur une machine et une copie unique du modèle, ou existe-t-il réplication, sauvegarde et procédure de reprise ? | Risque de point unique de rupture si le système dépend d'une seule machine ou d'une seule copie du modèle. |