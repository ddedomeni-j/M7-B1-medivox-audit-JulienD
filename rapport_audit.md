# Rapport d'audit — prédicteur de séjour prolongé v1 (MediVox)

## 1. Synthèse exécutive

L'audit du prédicteur MediVox met en évidence des risques prioritaires touchant l'équité, la fiabilité des performances, la sécurité et la traçabilité. 

Les problèmes rencontrés concernent aussi bien le déploiement et la traçabilité du fonctionnement du modèle en production que l'analyse éthique et la méthode qui ont été utilisées pour son entraînement. 

Les manquements les plus graves sont :
- **Équité** : les femmes sont beaucoup moins souvent signalées à risque de séjour prolongé que les hommes, alors que la durée moyenne de séjour observée est comparable. L'écart est plus marqué dans les prédictions du modèle que dans les données historiques.

- **Usage du score** : le rôle concret du score dans le parcours du patient n'est pas documenté. Il faut déterminer qui le consulte, quelles décisions il influence et si un professionnel peut réellement s'en écarter. Si le score joue un rôle déterminant dans une décision produisant un effet significatif sur le patient, les conditions d'application de l'article 22 du RGPD doivent être instruites.

- **Traçabilité** : en production, il n'est pas possible de reconstituer une prédiction, la version du modèle utilisée ou les circonstances d'un incident.

- **Sécurité des données** : un mot de passe de base de données est présent en clair dans le code fourni, ce qui crée un risque d'accès non autorisé à des données de santé protégées.

- **Continuité de service** : les garanties de sauvegarde, de réplication et de reprise en cas de panne ne sont pas documentées.


## 2. Contexte et périmètre

MediVox exploite un prédicteur hérité destiné à signaler les séjours hospitaliers présentant un risque de prolongation. Cet audit évalue, à partir du code, du modèle et du dataset fournis, les risques éthiques, réglementaires, techniques et de ressources associés à son entraînement, son utilisation et son exploitation.

Le tableau suivant décrit les éléments couverts et exclus par la présente étude :

| Statut | Domaine | Éléments couverts / limites |
|---|---|---|
| Audité | Données et personnes | Examiner, à partir du dataset et du code fournis, les données personnelles, les données sensibles et leur usage ; la cohérence de l'encodage des variables ; le statut apparent d'anonymisation ou de pseudonymisation ; les exclusions de variables visibles ; et les risques apparents de fuite de données ou d'absence de séparation entraînement/évaluation. |
| Audité | Équité et biais | Identifier les variables directes ou indirectes de discrimination, notamment le sexe ; mesurer le disparate impact et les erreurs par groupe. Le préjudice potentiel sera interprété selon l'usage réel du score. |
| Audité | Modèle | Examiner l'adéquation entre la famille de modèle, la taille du dataset, les variables explicatives et la variable cible ; l'entraînement, les risques de fuite, l'absence ou la présence d'optimisation et d'évaluation indépendante ; la sérialisation et le seuil de décision. |
| Audité | Pipeline d'entraînement | Examiner la modularité, la réutilisabilité, le couplage aux fichiers locaux, la reproductibilité et le versionnement visible du pipeline. |
| Audité | Usage et exploitation | Examiner la chaîne allant de la donnée au score : mode d'exécution, destinataires du score, décisions potentiellement influencées, supervision humaine visible, journalisation, déploiement et pratiques d'exploitation observables. Les éléments non observables deviennent des questions au client. |
| Audité | Technique et sécurité | Examiner l'architecture, la gestion des secrets, la validation des entrées, la gestion des erreurs, les accès et le transport visibles, la journalisation, les dépendances, ainsi que les points uniques de rupture. |
| Audité | Ressources | Mesurer la taille du modèle, la mémoire utilisée, les durées d'entraînement et d'inférence ; mettre ces mesures en regard du cas d'usage, de l'environnement client connu et d'au moins une alternative légère. |
| Audité | RGPD et AI Act | Qualifier de manière raisonnée les implications observables : données de santé et art. 9 RGPD, conditions de l'art. 22, qualification possible au titre de l'art. 6 de l'AI Act. La conclusion dépend de l'usage réel, de la supervision humaine et d'informations à confirmer avec le client. |
| Audité | Restitution | Documenter et hiérarchiser les constats, leur niveau de sévérité et leurs conséquences pour Hélène (technique) et Marc (DPO). |
| Exclu | Solutions | Proposer une architecture cible, un plan de remédiation détaillé ou des corrections. |
| Exclu | Modification | Modifier le code hérité, réentraîner le modèle en production ou déployer une nouvelle version. |
| Exclu | Données sources | Auditer la provenance, la collecte ou la complétude des données avant leur présence dans le dataset fourni. |
| Exclu | Conformité exhaustive | Réaliser une AIPD complète, rendre un avis juridique définitif ou certifier la conformité RGPD/AI Act. |
| Exclu | Sécurité exhaustive | Réaliser un test d'intrusion ou vérifier directement l'ensemble des mesures de sécurité de l'infrastructure de production. |

## 3. Volet éthique ⚖️

| Constat | Éléments techniques / chiffrés | Lecture opérationnelle |
|---|---|---|
| Utilisation directe de la variable indiquant le sexe | Le modèle utilise directement `sexe_bin`. Taux de prédictions positives « risque de séjour prolongé » : **14,1 %** pour les femmes, **48,6 %** pour les hommes. | L'utilisation du sexe doit être justifiée par une nécessité clinique ou métier documentée. |
| Amplification d'un écart entre femmes et hommes | DI F/M de **0,291** pour les prédictions, contre **0,653** dans les étiquettes historiques. | Signal d'alerte à investiguer, sans conclure à une discrimination. |
| Statut des identifiants inconnu | Le dataset contient `patient_id`, âge, sexe, département, service, type d'admission et durée de séjour. | Le statut d'anonymisation ou de pseudonymisation et le risque de réidentification par recoupement doivent être instruits. |
| Données de santé traitées | `nb_comorbidites`, `imc`, `dms_jours` et la cible concernent la prise en charge de santé. | La base légale et la condition applicable au titre de l'article 9 du RGPD doivent être confirmées avec le DPO. |
| Usage et supervision non documentés | Le script retourne automatiquement une classe et une probabilité ; aucune supervision humaine n'est visible dans les éléments fournis. | Préciser qui consulte le score, ce qu'il déclenche et si un professionnel peut réellement s'en écarter avant d'instruire l'art. 22 RGPD et la qualification AI Act. |
| Durées réelles comparables entre les sexes | Durée moyenne : **5,62 jours** pour les femmes et **5,59 jours** pour les hommes. | L'écart de signalement ne paraît pas expliqué par la seule durée moyenne observée ; il faut investiguer d'éventuels biais historiques ou organisationnels et l'usage réel du score. |
| Amplification d'écarts liés à l'âge | Pour les 10-29 ans, le DI passe de **0,434** dans les étiquettes à **0,129** dans les prédictions, comparé aux 90 ans et plus. | L'âge peut être cliniquement pertinent, mais les écarts observés sont amplifiés par le modèle. Les erreurs par groupe et la finalité du score doivent donc être étudiées. |
| Règle de labellisation non documentée | Les deux valeurs de `sejour_prolonge` se chevauchent dans `dms_jours`, entre **5,6** et **14,9 jours**. Le seuil de 7 jours est une référence exploratoire d'audit. | MediVox doit documenter la règle clinique, métier ou organisationnelle qui définit un séjour prolongé. |

## 4. Volet technique 👩‍💻

| Constat | Éléments techniques / chiffrés | Lecture opérationnelle |
|---|---|---|
| Secret exposé | Un mot de passe de base de données est écrit en clair dans le script d'entraînement. | Risque d'accès non autorisé aux données de santé en cas d'accès au dépôt ou de copie du script. |
| Validation des entrées absente | Le script de prédiction attend quatre arguments, sans contrôle de plage, de cohérence ou de schéma. | Des données incorrectes peuvent provoquer une erreur ou produire un score peu fiable. |
| Continuité de service à confirmer | Les éléments fournis ne documentent ni réplication, ni sauvegarde, ni reprise après incident. | Risque conditionnel d'indisponibilité si la production repose sur une seule machine ou une seule copie du modèle. |
| Architecture d'appel fragile | La prédiction est lancée par SSH et dépend du répertoire courant pour trouver le modèle. | Risque d'indisponibilité lié à la connexion SSH, aux droits ou au chemin d'exécution. |
| Fuite données lors de l'entrainement | Le modèle est entraîné sur l'intégralité du dataset et son score est calculé sur ces mêmes données ; aucun split entraînement/test ni validation croisée n'est visible. | La performance affichée est optimiste et la fiabilité du score sur de nouveaux séjours n'est pas démontrée. |
| Tests limités | Les tests disponibles vérifient seulement que le dataset est lisible, que le modèle est chargeable et que le script s'exécute. | Les entrées invalides, cas limites, régressions et comportements métier ne sont pas couverts avant déploiement. |
| Déploiement non maîtrisé | Le déploiement est réalisé manuellement par SCP, sans versionnement ni métadonnées du modèle visibles. | Difficulté à identifier la version active, à reproduire un résultat ou à revenir à un état connu après incident. |
| Chargement répété du modèle | Le fichier `.joblib` est chargé à chaque exécution du script de prédiction. | Latence et accès disque supplémentaires à chaque requête ; impact à mettre en regard du volume attendu. |
| Pipeline d'entraînement non répétable | Les scripts sont monolithiques ; la préparation des variables n'est pas encapsulée dans un pipeline partagé. | Risque de divergence entre entraînement et prédiction et reproductibilité limitée. |
| Seuil de décision non justifié | La classe positive est décidée avec un seuil fixe de `0,5`, sans justification ni paramétrage visible. | Le niveau de signalement et les conséquences opérationnelles reposent sur un choix non documenté. |
| Supervision insuffisante | Aucun suivi de charge, de latence, d'erreurs ou système d'alerte n'est visible. | Les indisponibilités et dégradations peuvent être détectées tardivement. |
| Traçabilité insuffisante | Les requêtes, probabilités, erreurs et versions de modèle ne sont pas journalisées. | Impossible de retracer ou rejouer une prédiction dans son contexte et d'auditer un incident. |
| Explicabilité non documentée | Aucune explication locale ou globale des facteurs ayant conduit à un score n'est disponible dans les éléments audités. | Les professionnels ne peuvent pas facilement vérifier la cohérence d'un score ou le contester. |

## 5. Volet ressources

| Constat | Éléments techniques / chiffrés | Lecture opérationnelle |
|---|---|---|
| Empreinte disque du modèle historique élevée | Random Forest : **4,956 Mo**. HistGradientBoosting : **0,366 Mo**. XGBoost : **0,971 Mo**. | Le modèle historique est environ 14 fois plus volumineux que HistGradientBoosting ; A évaluer en fonction de la configuration de déploiement. |
| Inférence plus lente à volume élevé | Pour un lot de 10 000 lignes les temps d'inférence sont : Random Forest **45,845 ms**, HistGradientBoosting **15,788 ms**, XGBoost **14,187 ms**. | Les alternatives testées réduisent la latence par lot d'environ un facteur trois. L'impact réel dépend du volume attendu en production. |
| Chargement mémoire de XGBoost important | Variation de RSS à l'entraînement : Random Forest **+6,222 Mo**, HistGradientBoosting **+0,172 Mo**, XGBoost **+119,497 Mo**. | HistGradientBoosting est à envisager. XGBoost est rapide en inférence, mais son entraînement est nettement plus exigeant en mémoire. |
| HistGradientBoosting est l'alternative la plus sobre | Taille de **0,366 Mo**, RSS de **+0,172 Mo** et inférence de **15,788 ms** sur un lot de 10 000 lignes. | Cette alternative présente la plus faible empreinte parmi les modèles testés. |
| Temps d'entraînement comparables | Random Forest : **0,302 s** ; HistGradientBoosting : **0,314 s** ; XGBoost : **0,237 s**. | Sur le dataset fourni, ce critère ne permet pas à lui seul de différencier les modèles. |
| Performance | F1 : Random Forest **0,691**, HistGradientBoosting **0,582**, XGBoost **0,549**. </br>Le modèle **Random Forest** a été évalué sur des **données déjà vues à l'entraînement**. | La performance du modèle historique est optimiste ; sa supériorité ne peut pas être établie sans faire une nouvelle évaluation une fois le pipeline d'entraînement corrigé. |
| Latence de production non mesurée intégralement | Le script recharge le fichier `.joblib` à chaque prédiction ; les mesures réalisées concernent l'inférence du modèle. | Le temps réel inclut aussi le démarrage du script, le chargement disque et l'appel SSH. |
| Dimensionnement cible non documenté | Aucun volume de requêtes, objectif de délai ou capacité d'infrastructure n'est fourni. | Il n'est pas possible de conclure au surdimensionnement de la solution sans connaître les contraintes d'exploitation. |

## 6. Tableau consolidé des risques

| Priorité | Type | Description |
|---|---|---|
| 🔴 P1 | Éthique | Le sexe est utilisé directement comme variable d'entrée du modèle, sans justification clinique ou réglementaire visible. |
| 🔴 P1 | Éthique | Le modèle amplifie l'écart entre femmes et hommes : DI F/M de **0,291** contre **0,653** dans les étiquettes historiques, alors que les durées moyennes sont comparables (**5,62 jours** pour les femmes, **5,59 jours** pour les hommes). |
| 🔴 P1 | Éthique / réglementaire | L'usage opérationnel, la supervision humaine et les effets du score sur le patient ne sont pas documentés. Si le score joue un rôle déterminant dans une décision ayant un effet significatif, l'article 22 du RGPD et la qualification AI Act doivent être instruits. |
| 🔴 P1 | Technique / traçabilité | Les requêtes, scores, versions de modèle et erreurs ne sont pas journalisés : une prédiction ne peut pas être retracée ou rejouée dans son contexte. |
| 🔴 P1 | Technique / sécurité | Un mot de passe de base de données est inscrit en clair dans `legacy/train.py`. |
| 🔴 P1 | Technique | Risque conditionnel de point unique de rupture, à confirmer selon l'existence d'une réplication, de sauvegardes et d'une procédure de reprise en production. |
| 🟠 P2 | Technique | Absence de séparation entraînement/test ou de validation croisée : la performance affichée est calculée sur les données d'entraînement et ne démontre pas la généralisation. |
| 🟠 P2 | Éthique | Le modèle amplifie les écarts liés à l'âge : le DI des 10-29 ans passe de **0,434** dans les étiquettes à **0,129** dans les prédictions, comparé au groupe des 90 ans et plus. |
| 🟠 P2 | Éthique | Selon la référence exploratoire de 7 jours, les taux de signalement diffèrent des taux de séjours longs observés selon l'âge. Cette lecture doit être confirmée par la règle métier de labellisation ; les FNR/FPR par groupe restent à calculer. |
| 🟠 P2 | Technique | Les entrées de prédiction ne sont pas validées : absence de schéma, de contrôle des types métier et des plages de valeurs. |
| 🟠 P2 | Technique | Le déploiement est manuel par SCP, sans versionnement visible du modèle ni métadonnées associées. |
| 🟠 P2 | Technique | L'appel de prédiction par SSH dépend d'une exécution depuis la racine du projet et d'un chemin relatif vers le modèle. |
| 🟠 P2 | Technique | Les scripts sont monolithiques et la préparation des données n'est pas encapsulée dans un pipeline partagé entre entraînement et prédiction. |
| 🟠 P2 | Technique | Le seuil de décision est fixé à `0,5` sans justification, configuration ni traçabilité visible. |
| 🟠 P2 | Technique | Aucune supervision, mesure de charge ou alerte en temps réel n'est visible ; les incidents et dégradations risquent d'être détectés tardivement. |
| 🟠 P2 | Technique | Aucune analyse d'explicabilité locale ou globale n'est fournie pour comprendre les facteurs influençant un score. |
| 🟠 P2 | Technique | Les tests disponibles sont limités au démarrage ; les entrées invalides, cas limites, régressions et comportements métier ne sont pas couverts. |
| 🟠 P2 | Éthique / données personnelles | Le dataset contient `patient_id` et plusieurs quasi-identifiants ; aucune documentation sur l'anonymisation ou la pseudonymisation n'est fournie. Le risque de réidentification par recoupement doit être instruit. |
| 🟡 P3 | Ressources | Le modèle Random Forest legacy est plus lourd que HistGradientBoosting : **4,956 Mo** contre **0,366 Mo**, sans gain de performance démontré par une évaluation indépendante. |
| 🟡 P3 | Ressources | À fort volume, le modèle Random Forest est plus lent : **45,845 ms** pour un lot de 10 000 séjours, contre **15,788 ms** pour HistGradientBoosting et **14,187 ms** pour XGBoost. |
| 🟡 P3 | Ressources | Le modèle Random Forest augmente le RSS de **6,222 Mo** à l'entraînement, contre **0,172 Mo** pour HistGradientBoosting dans le protocole mesuré. |
| 🟡 P3 | Technique / ressources | Le modèle est rechargé depuis le disque à chaque appel de prédiction, ajoutant une latence et des accès disque dont l'impact dépend du volume réel de requêtes. |

## 7. Questions ouvertes pour le client

- Quel usage opérationnel le score déclenche-t-il, qui le consulte et peut-il être contredit ?
- La variable sexe est-elle justifiée médicalement ?
- Quelle base légale et quelle condition de l’art. 9 RGPD couvrent ce traitement ?
- S’agit-il d’un dispositif médical, d’un outil de triage ou d’un système influençant une décision relevant de l’AI Act ?
- Quelle est la règle clinique/métier de labellisation de sejour_prolonge ? La règle de labellisation de sejour_prolonge n’est pas documentée dans les éléments audités. La variable ne correspond pas à un seuil unique observable de dms_jours : les deux classes se chevauchent entre 5,6 et 14,9 jours.
- L’infrastructure a-t-elle une réplication, une sauvegarde et une procédure de reprise ?