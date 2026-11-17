# Empoisonnement de données et attaques par porte dérobée sur MNIST

Projet de sécurité de l'IA, INPT, Master Data et Intelligence Artificielle.
Auteure : Asmaa Aandadi. Encadrant : Pr. Belmekki.

Étude complète d'une attaque *backdoor* sur un CNN entraîné sur MNIST : modèle de référence, attaque, détection des échantillons empoisonnés, remédiation, comparaison, puis analyses complémentaires.

## Contenu du dépôt

| Fichier | Rôle |
|---|---|
| `backdoor_poisoning_notebook_securite.ipynb` | Notebook complet (sections 1 à 12), pensé pour Google Colab avec GPU |
| `requirements.txt` | Versions des bibliothèques utilisées |
| `README.md` | Ce fichier |

Le dossier `results/` (tableaux CSV, figures PNG, `config.json`) **n'est pas versionné** : il est créé par le notebook à l'exécution.

## Scénario

- Données : MNIST, 54 000 images d'entraînement, 6 000 de validation, 10 000 de test (jamais modifié).
- Modèle : CNN (2 convolutions, dense 128), Adam, lr 1e-3, lot 128, 6 époques.
- Attaque : carré blanc 3×3 en bas à droite, classe source 7 → classe cible 1, 10 % des images de classe 7 (564 images).
- Détection : activations de la dernière couche cachée (128 dim.), classe par classe. Clustering propre (ACP 10 dim. + 2-means), ART `ActivationDefence`, Spectral Signatures.
- Remédiation : suppression des échantillons signalés, réentraînement depuis zéro.

## Résultats principaux (graine 0)

| Modèle | Accuracy propre | ASR |
|---|---|---|
| Référence | 98,95 % | 0,19 % |
| Compromis | 99,07 % | 100 % |
| Remédié (clustering propre, 5 empoisonnés restants) | 98,97 % | 62,16 % |
| Remédié (Spectral Signatures, 0 restant) | 99,01 % | 0,19 % |

Le clustering propre retrouve 559 des 564 échantillons empoisonnés (11 faux positifs, F1 = 0,986).
Les tableaux détaillés proviennent d'une seule exécution ; l'étude du taux d'empoisonnement (1 à 15 %) utilise 3 graines.

## Exécution

1. Ouvrir le notebook dans Google Colab et choisir un GPU (T4).
2. Tout exécuter dans l'ordre. La première cellule installe `adversarial-robustness-toolbox==1.20.1`. Durée totale : environ 10 minutes pour la section 11, plus l'étude de sensibilité (environ 7 minutes).
3. En local : `pip install -r requirements.txt`, puis lancer Jupyter.

Les graines sont fixées (`CONFIG` en début de notebook), mais des écarts mineurs sont possibles selon le matériel et les versions.

## Note sur ART et NumPy

Avec ART 1.20.1 et NumPy ≥ 2.2, le mode `relative-size` de `ActivationDefence` échoue (`ValueError`, test d'appartenance sur un tableau vide). La section 11.1 du notebook documente la cause et applique un correctif minimal (le seul test d'appartenance). Le mode signale alors les mêmes 570 échantillons que le clustering propre.

## Liens

- Présentation : https://docs.google.com/presentation/d/15aymWvrICiY0Ibl_jQvFsKGDIr9bgYJb/edit?usp=drive_link
- Dépôt : https://github.com/andasmaa/securite
