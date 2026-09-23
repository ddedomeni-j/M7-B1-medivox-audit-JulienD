# Audit — ethique

### Données personnelles et risque de réidentification

Le dataset contient un identifiant `patient_id` ainsi que plusieurs quasi-identifiants : âge, sexe, département, service, type d'admission et durée de séjour. Aucune documentation fournie ne permet de déterminer si `patient_id` est un identifiant direct, un pseudonyme ou le résultat d'une anonymisation.

La combinaison de ces informations constitue un risque de réidentification indirecte si les données sont recoupées avec d'autres sources. Ce constat ne permet pas de conclure à l'absence d'anonymisation ou de pseudonymisation, mais il justifie de confirmer le statut des identifiants et les mesures de protection appliquées par MediVox.

### Disparate impact par sexe : amplification à investiguer

Le taux de signalement de séjour prolongé est de **14,1 %** pour les femmes, contre **48,6 %** pour les hommes. Le DI F/M du modèle est de **0,291**, nettement sous le repère conventionnel de 0,80.

Un écart est déjà présent dans les étiquettes historiques : **32,1 %** des séjours féminins et **49,1 %** des séjours masculins y sont qualifiés de prolongés, soit un DI F/M de **0,653**. Le modèle amplifie donc l’écart relatif historique : le DI diminue de **0,653** à **0,291**, soit une baisse de **0,361**.

Ce résultat est un signal d’alerte, non une preuve de discrimination. Il faut vérifier, à partir de `dms_jours`, si l’écart observé reflète des différences de durée réellement constatées ou un biais dans l’étiquetage, le codage ou l’organisation des soins. L’usage opérationnel du score doit également être précisé : être signalée comme présentant un risque de séjour prolongé peut constituer un bénéfice d’anticipation ou, au contraire, entraîner une décision restrictive.

### Disparate impact par âge : amplification à investiguer

Le groupe de référence est celui des **90 ans et plus**, avec un taux de signalement de **58,3 %** et un taux historique de séjour prolongé de **55,7 %**.

Pour les groupes plus jeunes, le modèle accentue les écarts déjà présents dans les étiquettes historiques :

| Groupe d'âge | DI étiquette | DI modèle | Évolution |
|---|---:|---:|---:|
| 10-29 ans | 0,434 | 0,129 | -0,305 |
| 30-49 ans | 0,552 | 0,254 | -0,298 |
| 50-69 ans | 0,771 | 0,566 | -0,205 |
| 70-89 ans | 0,979 | 0,934 | -0,045 |
| 90 ans et plus | 1,000 | 1,000 | 0,000 |

Les classes de moins de 70 ans sont sous le repère conventionnel de **0,80** dans les données historiques comme dans les prédictions. Le modèle accroît cet écart : par exemple, les 10-29 ans passent d’un taux historique de séjour prolongé de **24,2 %** à un taux de signalement de seulement **7,5 %**, soit un DI qui diminue de **0,434** à **0,129**.

Ce résultat est un signal d’alerte. L’augmentation du risque avec l’âge peut être médicalement ou organisationnellement explicable. Il faut toutefois vérifier, à partir de `dms_jours`, si ces écarts reflètent bien les durées de séjour réellement observées et si les erreurs du modèle sont comparables entre tranches d’âge.

### Durée réelle, prédiction et disparate impact

La variable `sejour_prolonge` ne repose pas sur un seuil unique observable dans `dms_jours` : les séjours non prolongés atteignent 14,9 jours, tandis que les séjours prolongés commencent à 5,6 jours. Le seuil de **7 jours** est donc retenu comme référence exploratoire, car il est proche de la moyenne des séjours étiquetés prolongés (7,61 jours) et de leur médiane (7,3 jours). Cette hypothèse doit toutefois être confirmée par MediVox.

L'analyse par sexe montre des durées réelles comparables : **5,62 jours** en moyenne pour les femmes, contre **5,59 jours** pour les hommes, et des médianes de **5,6** et **5,5 jours**. Avec le seuil de 7 jours, les taux de séjours longs sont également très proches : **29,4 %** pour les femmes et **29,0 %** pour les hommes. Ces données ne suffisent donc pas à expliquer l'écart de prédiction très marqué : **14,1 %** de femmes signalées contre **48,6 %** d'hommes, soit un DI F/M de **0,291**.

Le croisement avec le DI renforce le signal d'alerte : alors que la durée réelle de séjour est équivalente entre les deux groupes, l'étiquette historique crée déjà un écart de signalement (**32,1 %** pour les femmes contre **49,1 %** pour les hommes), puis le modèle l'amplifie. Le DI F/M diminue de **0,653** sur les étiquettes à **0,291** sur les prédictions.

Pour l'âge, la durée moyenne augmente de **4,41 jours** chez les 10-29 ans à **6,91 jours** chez les 90 ans et plus. Cette tendance est cohérente avec une hausse de la fréquence des séjours d'au moins 7 jours, de **13,8 %** à **47,9 %**. Elle explique partiellement le DI défavorable aux groupes jeunes. Cependant, le modèle amplifie l'écart : il ne signale que **7,5 %** des 10-29 ans, pour **13,8 %** de séjours longs selon cette référence, tandis qu'il signale **58,3 %** des 90 ans et plus pour **47,9 %** de séjours longs.

Ces conclusions restent exploratoires : le seuil de 7 jours ne remplace pas une définition clinique ou métier du séjour prolongé. Elles justifient néanmoins le calcul des FNR et FPR par sexe et par âge afin d’identifier quels groupes subissent le plus de séjours longs non signalés ou de signalements injustifiés.
