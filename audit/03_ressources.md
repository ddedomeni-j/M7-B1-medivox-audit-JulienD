# Audit — ressources

### Comparaison ressources : Random Forest legacy, HistGB et XGBoost

| Indicateur | Random Forest legacy | HistGradientBoosting | XGBoost |
|---|---:|---:|---:|
| F1 sur le jeu de test | 0,691* | 0,582 | 0,549 |
| Temps d'entraînement | 0,302 s | 0,314 s | 0,237 s |
| Taille sérialisée | 4,956 Mo | 0,366 Mo | 0,971 Mo |
| Variation RSS à l'entraînement | +6,222 Mo | +0,172 Mo | +119,497 Mo |
| Inférence, 100 séjours | 2,914 ms | 3,612 ms | 1,053 ms |
| Inférence, 1 000 séjours | 7,174 ms | 4,631 ms | 2,525 ms |
| Inférence, 10 000 séjours | 45,845 ms | 15,788 ms | 14,187 ms |

** Warning ** Le F1 du modèle legacy est optimiste : ce modèle a vu les données utilisées pour son évaluation lors de l'entrainement.

XGBoost est l'alternative la plus rapide à l'inférence : il réduit la latence d'environ 64 % à 69 % par rapport au modèle legacy selon le volume traité. Il est également plus rapide à entraîner que HistGradientBoosting, avec un temps de 0,237 s contre 0,314 s.

En revanche, XGBoost obtient le F1 le plus faible des deux alternatives évaluées sur un jeu de test indépendant : 0,549 contre 0,582 pour HistGradientBoosting. Son modèle sérialisé reste 80 % plus léger que le modèle legacy, mais il est 2,7 fois plus lourd que HistGradientBoosting.

Le principal point de vigilance est la mémoire : XGBoost entraîne une hausse de RSS de 119,497 Mo, très supérieure aux 0,172 Mo observés pour HistGradientBoosting. Cette mesure dépend de l'état du processus Python, mais l'écart est suffisamment important pour indiquer un coût mémoire opérationnel potentiellement plus élevé.

**Conclusion** : La comparaison avec deux modèles alternatifs montre que le modèle Random Forest hérité n'est pas optimal au regard des ressources consommées. HistGradientBoosting fournit un meilleur compromis entre performance mesurée sur un jeu de test indépendant, taille du modèle et consommation mémoire observée. XGBoost réduit davantage la latence d'inférence, mais avec une performance F1 inférieure et une hausse importante de mémoire lors de l'entraînement.

La performance du modèle legacy ne peut pas être utilisée pour justifier ce coût supplémentaire : son F1 est calculé sur des données déjà vues à l'entraînement. L'absence de protocole d'évaluation indépendant empêche donc de démontrer que la Random Forest apporte un gain de performance suffisant pour compenser son fichier 13,5 fois plus volumineux et sa latence plus élevée sur les volumes importants.