# Estimation de la Densité de Trafic Urbain par CSRNet

Estimation de la densité de véhicules sur images de vidéosurveillance routière, par régression de carte de densité continue (architecture **CSRNet** : VGG-16 + convolutions dilatées), entraînée sur le dataset **TRANCOS**.

**🔗 Démo en ligne :** [tp-vo-groupe8.streamlit.app](https://tp-vo-groupe8.streamlit.app)

---

## Contexte

Le comptage de véhicules par détection classique (YOLO, Faster R-CNN) échoue dans les scènes de forte congestion : occlusions sévères, distorsion de perspective, conditions d'éclairage variables. Ce projet adopte plutôt une approche par **régression de carte de densité** : le modèle prédit une carte continue dont l'intégrale spatiale donne directement le comptage, sans détection individuelle de chaque véhicule.

Aucun dataset annoté par points adapté au contexte routier camerounais/africain n'ayant été identifié, le modèle est entraîné et évalué sur **TRANCOS**, dataset de référence pour l'estimation de densité de véhicules.

## Méthodologie

- **Génération de la vérité-terrain** : conversion des points annotés en cartes de densité continues par convolution gaussienne, avec un **sigma adaptatif géométrique** (basé sur la distance aux *k* plus proches voisins) pour tenir compte de l'effet de perspective caméra. Correction par renormalisation pour compenser la perte de masse aux bords/frontières de la ROI.
- **Architecture CSRNet** : front-end VGG-16 tronqué (pré-entraîné ImageNet, 3 max-poolings au lieu de 5) + back-end de 6 convolutions dilatées (taux=2), produisant une carte de densité à résolution 1/8 de l'image d'entrée.
- **Loss** : MSE pondérée spatialement, pénalisant davantage les erreurs sur les zones de forte densité.
- **Entraînement** : Adam, gel du front-end VGG durant les 10 premières époques puis fine-tuning progressif, sur GPU (Google Colab, T4).

## Résultats

Évalué sur les 213 images du test set TRANCOS :

| Métrique | Valeur |
|---|---|
| MAE | 4.09 |
| MSE | 57.24 |
| RMSE | 7.57 |

| GAME(0) | GAME(1) | GAME(2) | GAME(3) |
|---|---|---|---|
| 4.09 | 7.44 | 8.91 | 10.82 |

Le MAE de 4.09 signifie une erreur moyenne d'environ 4 véhicules par image, y compris sur des scènes dépassant 50 véhicules enchevêtrés.

## Dashboard interactif

Une application **Streamlit** a été développée pour exploiter le modèle entraîné :

- upload d'une image de caméra urbaine ;
- inférence CSRNet en temps réel ;
- carte de densité affichée en heatmap superposée à l'image ;
- comptage total estimé et seuils de congestion configurables (fluide / modéré / congestion).

**👉 [Tester l'application](https://tp-vo-groupe8.streamlit.app)**

## Limites et pistes d'amélioration

- **Écart de domaine** : le modèle est entraîné sur des scènes autoroutières espagnoles (TRANCOS), structurellement différentes du trafic camerounais (forte proportion de motos/taxis, chevauchement des files, absence de marquage au sol). Une dégradation de performance est attendue en application directe sur des scènes locales.
- **Sous-estimation en très forte congestion**, lorsque les véhicules à l'horizon forment un bloc difficilement discernable.
- Pistes : annotation de données locales (CVAT) pour fine-tuning, ou techniques d'adaptation de domaine non supervisée (CycleGAN, etc.).

## Structure du repo

```
.
├── notebook/           # Notebook complet : exploration, génération de densité, training, évaluation
├── dashboard/           # Code de l'application Streamlit
├── rapport/             # Rapport technique complet (PDF + LaTeX)
└── README.md
```

---

*Projet réalisé dans le cadre du cours INF4238 — Vision par Ordinateur, Université de Yaoundé I.*
