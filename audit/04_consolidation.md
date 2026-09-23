# Audit — consolidation

Le tableau ci-dessous liste les problèmes détectés lors de l'audit. Les priorités P1 regroupent les constats pouvant affecter directement les personnes, la conformité ou la fiabilité du score. Les P2 concernent la sécurité, l’exploitation et la gouvernance technique. Les P3 portent surtout sur l’efficience, à interpréter selon le volume réel et les contraintes de MediVox.

| Priorité | Type | Description |
|---|---|---|
| 🔴 P1 | Éthique | Le sexe est utilisé directement comme variable d'entrée du modèle, sans justification clinique ou réglementaire visible. |
| 🔴 P1 | Éthique | Le modèle amplifie l'écart entre femmes et hommes : DI F/M de **0,291** contre **0,653** dans les étiquettes historiques, alors que les durées réelles moyennes sont comparables (5,62 jours pour les femmes, 5,59 pour les hommes). |
| 🔴 P1 | Risque conditionnel éthique / réglementaire | L'usage opérationnel, le niveau de supervision humaine et les effets du score sur le patient ne sont pas documentés. | Si le score joue un rôle déterminant dans une décision ayant un effet significatif, risque pour les personnes et nécessité d'instruire l'art. 22 RGPD ainsi que la qualification AI Act. |
| 🔴 P1 | Technique / traçabilité | Les requêtes, scores, versions de modèle et erreurs ne sont pas journalisés : une prédiction ne peut pas être retracée ou rejouée dans son contexte. |
| 🔴 P1 | Technique / sécurité | Un mot de passe de base de données est inscrit en clair dans `legacy/train.py`. |
| 🔴 P1 | Technique | Risque conditionnel de point unique de rupture : à confirmer selon l'existence d'une réplication, de sauvegardes et d'une procédure de reprise en production. |
| 🟠 P2 | Technique | Absence de séparation entraînement/test ou de validation croisée : la performance affichée est calculée sur les données d'entraînement et ne démontre pas la généralisation. |
| 🟠 P2 | Éthique | Le modèle amplifie les écarts par âge : le DI des 10-29 ans passe de **0,434** dans les étiquettes à **0,129** dans les prédictions, comparé au groupe des 90 ans et plus. |
| 🟠 P2 | Éthique | Selon la référence exploratoire de 7 jours, les taux de signalement diffèrent des taux de séjours longs observés selon l'âge. Cette lecture doit être confirmée par la règle métier de labellisation ; FNR/FPR par groupe restent à calculer. |
| 🟠 P2 | Technique | Les entrées de prédiction ne sont pas validées : absence de schéma, de contrôle des types métier et des plages de valeurs. |
| 🟠 P2 | Technique | Le déploiement est manuel par SCP, sans versionnement visible du modèle ni métadonnées associées. |
| 🟠 P2 | Technique | L'appel de prédiction par SSH dépend d'une exécution depuis la racine du projet et d'un chemin relatif vers le modèle. |
| 🟠 P2 | Technique | Les scripts sont monolithiques et la préparation des données n'est pas encapsulée dans un pipeline partagé entre entraînement et prédiction. |
| 🟠 P2 | Technique | Le seuil de décision est fixé à `0,5` sans justification, configuration ni traçabilité visible. |
| 🟠 P2 | Technique | Aucune supervision, mesure de charge ou alerte en temps réel n'est visible ; les incidents, erreurs répétées et dégradations de latence risquent d'être détectés tardivement. |
| 🟠 P2 | Technique | Aucune analyse d'explicabilité locale ou globale n'est fournie pour comprendre les facteurs influençant un score. |
| 🟠 P2 | Technique | Les tests disponibles vérifient uniquement le démarrage (données lisibles, modèle chargeable, script exécutable) ; aucune couverture visible des entrées invalides, des cas limites, des régressions ou du comportement métier. |
| 🟠 P2 | Éthique / données personnelles | Le dataset contient un identifiant `patient_id` et plusieurs quasi-identifiants (âge, sexe, département, service, admission, durée de séjour) ; aucune documentation sur l'anonymisation ou la pseudonymisation n'est fournie. | Risque de réidentification croisées ; le statut réel des identifiants et les mesures de protection doivent être confirmés. |

| 🟡 P3 | Ressources | Le modèle Random Forest legacy est nettement plus lourde que HistGB : **4,956 Mo** contre **0,366 Mo**, sans gain de performance démontré par une évaluation indépendante. |
| 🟡 P3 | Ressources | À fort volume, le modèle Random Forest est plus lente : **45,845 ms** pour 10 000 séjours contre **15,788 ms** pour HistGB et **14,187 ms** pour XGBoost. |
| 🟡 P3 | Ressources | Le modèle Random Forest augmente le RSS de **6,222 Mo** lors de l'entraînement, contre **0,172 Mo** pour HistGB dans le protocole mesuré. |
| 🟡 P3 | Technique / ressources | Le modèle est rechargé depuis le disque à chaque appel de prédiction, ce qui ajoute une latence et des accès disque dont l'impact dépendra du volume réel de requêtes. |
pas de test

**Questions** :

- Quel usage opérationnel le score déclenche-t-il, qui le consulte et peut-il être contredit ?
- Quelle est la règle clinique/métier de labellisation de sejour_prolonge ? La règle de labellisation de sejour_prolonge n’est pas documentée dans les éléments audités. La variable ne correspond pas à un seuil unique observable de dms_jours : les deux classes se chevauchent entre 5,6 et 14,9 jours.
- La variable sexe est-elle justifiée médicalement ?
- Quelle base légale et quelle condition de l’art. 9 RGPD couvrent ce traitement ?
- S’agit-il d’un dispositif médical, d’un outil de triage ou d’un système influençant une décision relevant de l’AI Act ?
- L’infrastructure a-t-elle une réplication, une sauvegarde et une procédure de reprise ?