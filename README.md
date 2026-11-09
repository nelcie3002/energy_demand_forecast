# Prévision de la demande d'électricité en Californie (CISO)

Modèle de prévision **à 24 heures** de la demande électrique horaire du réseau
California ISO, à partir de données publiques de réseau et de météo.

## Question métier

Les opérateurs de réseau doivent prévoir la demande heure par heure pour
planifier la production. Ce projet reproduit cet exercice : peut-on prévoir la
demande de demain avec une erreur faible, et mieux qu'une règle simple ?

## Données

- **Demande horaire** : U.S. Energy Information Administration (API EIA v2),
  région CISO, janvier 2023 à décembre 2025 (26 304 heures).
- **Météo** : Open-Meteo (température et humidité, Los Angeles).
- Une valeur aberrante (31 juillet 2025, 21:00 UTC : 11 819 MWh au lieu
  d'environ 30 000) a été identifiée et corrigée par interpolation linéaire.

## Méthode

- **Variables** : heure, jour de la semaine et mois en heure locale
  (Los Angeles), week-end, jours fériés californiens, demande décalée de
  24 h et 168 h, moyenne glissante sur 24 h, température, humidité,
  degrés de climatisation.
- **Absence de fuite** : toutes les variables de demande utilisent
  uniquement des valeurs antérieures d'au moins 24 h.
- **Découpage chronologique** : entraînement (jusqu'à mars 2025),
  validation (avril à août 2025, pour l'arrêt anticipé),
  test (septembre à décembre 2025, jamais utilisé pour régler le modèle).
- **Modèle** : LightGBM, comparé à une baseline naïve
  (demande de la même heure la semaine précédente).

## Résultats (test : septembre à décembre 2025)

| Modèle | MAE (MWh) | MAPE |
|---|---|---|
| Baseline naïve (même heure, semaine dernière) | 1 475 | 5,50 % |
| **LightGBM** | **699** | **2,64 %** |

L'erreur est réduite d'un facteur 2,1 par rapport à la baseline.

![Réel vs prévu](images/reel_vs_prevu.png)

Variables les plus importantes (gain) : `lag_24`, `lag_168`, `dayofweek`.

## Analyse d'erreurs

- L'erreur est la plus faible la nuit (environ 2 %) et la plus forte en milieu
  de journée (jusqu'à [X] % vers 14 h locale), période où la production solaire
  en toiture, non mesurée ici, fait varier la demande nette.
- [Compléter avec l'analyse des pires heures après la correction.]

## Limites et pistes d'amélioration

- Météo d'une seule ville, alors que CISO couvre tout l'État.
- Température **observée** et non **prévue** : en conditions réelles, la
  météo à J+1 est elle-même incertaine.
- Pas de données de production solaire ni d'événements locaux.
- Pistes : météo multi-villes, validation croisée temporelle,
  prévision probabiliste (intervalles).

## Reproduire le projet

1. Obtenir une clé API gratuite sur https://www.eia.gov/opendata/
2. Ouvrir le notebook dans Google Colab et ajouter la clé dans les
   Secrets sous le nom `EIA_API_KEY`.
3. Exécuter toutes les cellules.

## Stack

Python, pandas, LightGBM, Plotly, scikit-learn.
