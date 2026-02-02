# PortfolioSuperForecast

Etait à la base un fork de [monte-carlo-for-investments](https://github.com/kconstable/monte-carlo-for-investments) de kconstable, dont l'objectif initial était de lancer des simulations sur d'autres actifs, cependant j'ai fait beaucoup ajouts et de modifs pour mieux correspondre à mes besoins.

## Description

Dans ce projet, j'utilise des simulations de Monte-Carlo pour sélectionner les pondérations optimales d'un portefeuille d'actifs qui offre le meilleur rendement ajusté au risque.
Il y a deux classes principales : **Security** et **Portfolio**. Les Securities (titres) sont des actifs individuels pour lesquels nous pouvons récupérer les prix historiques via l'API [Alphavantage](https://www.alphavantage.co/). Les Portfolios (portefeuilles) sont composés de paniers de titres avec leurs pondérations associées. L'annexe contient les détails sur les attributs et méthodes utilisés dans chaque classe.

Dans une simulation de Monte-Carlo, nous pouvons déduire le rendement et le risque les plus probables d'un panier d'actifs en nous basant sur les rendements de chaque actif (mu), le risque (sigma), et la covariance entre les actifs du portefeuille. Nous générons des rendements aléatoires normaux pour chaque titre du portefeuille, qui sont corrélés avec les autres rendements aléatoires générés. Nous pouvons ensuite déterminer le rendement pondéré du portefeuille à chaque pas de temps sur une période fixe. Cela nous permet de déterminer la valeur finale du portefeuille. Si nous générons de nombreux chemins, nous pouvons déterminer le rendement et le risque moyens du portefeuille pour un portefeuille donné.

## Installation

### Environnement Virtuel Python (Recommandé)

Je recommande l'utilisation d'un environnement virtuel pour isoler les dépendances du projet :

```bash
# Créer un environnement virtuel
python -m venv venv

# Activer l'environnement virtuel
# Sur Windows :
venv\Scripts\activate
# Sur macOS/Linux :
source venv/bin/activate
```

### Dépendances

Installer les dépendances requises :

```bash
pip install -r requirements.txt
```

### Configuration de la Clé API

Ce projet utilise l'API [Alphavantage](https://www.alphavantage.co/) pour récupérer les prix historiques des titres. Vous aurez besoin de :

1. Obtenir une clé API gratuite depuis [Alphavantage](https://www.alphavantage.co/support/#api-key)
2. Créer un répertoire `keys` à la racine du projet
3. Sauvegarder votre clé API dans `keys/alphavantage.txt`

```bash
mkdir keys
echo "API_KEY" > keys/alphavantage.txt
```

## Structure du Projet

```
monte-carlo-for-investments/
├── config/
│   ├── base_conf.yaml              # Configuration de base (simulation + frontier)
│   └── portfolios/
│       ├── example.yaml            # Exemple de portefeuille
│       └── core_satellite_1.yaml   # Portefeuille Core-Satellite
├── src/
│   ├── __init__.py
│   ├── cli.py                      # Interface en ligne de commande
│   ├── config.py                   # Constantes globales
│   ├── main.py                     # Fonctions métier (create_portfolio, run_simulate, run_frontier)
│   ├── models.py                   # Classes Security, Equity, LeveragedEquity, Portfolio
│   ├── schemas.py                  # Validation Pydantic des fichiers YAML
│   ├── simulation.py               # Logique Monte-Carlo
│   ├── utils.py                    # Fonctions utilitaires
│   └── visualization.py            # Graphiques Plotly
├── keys/
│   └── alphavantage.txt            # Clé API (non versionnée)
├── monte_carlo.ipynb               # Notebook de démonstration
├── requirements.txt
└── README.md
```

## Utilisation

### Option 1 : Interface en Ligne de Commande (CLI)

Le CLI permet de lancer des simulations ou construire des frontières efficientes directement depuis le terminal.

#### Commandes disponibles

```bash
# Afficher l'aide
python -m src.cli --help

# Simulation Monte-Carlo
python -m src.cli simulate --portfolio config/portfolios/example.yaml --config config/base_conf.yaml

# Frontière efficiente
python -m src.cli frontier --portfolio config/portfolios/example.yaml --config config/base_conf.yaml
```

#### Fichiers de Configuration

**Portfolio YAML** (`config/portfolios/example.yaml`) :

```yaml
securities:
  - name: "S&P 500 ETF"
    identifier: "SPY"
    mu: 0.10 # Rendement annuel attendu
    sigma: 0.15 # Volatilité annuelle
    type: "equity"

  - name: "NASDAQ-100 2x Leveraged"
    identifier: "QLD"
    base: "QQQ" # Titre sous-jacent
    leverage: 2
    type: "leveraged"

portfolio:
  name: "Mon Portefeuille"
  value: 100000
  weights: [0.60, 0.40]

rebalancing: false
```

**Configuration YAML** (`config/base_conf.yaml`) :

```yaml
general:
  rf: 0.04
  conf_level: 0.95
  price_start_year: 2013

simulation:
  simulations: 1000
  years: 20
  frequency: "daily"

frontier:
  total_weight: 100
  min_weight: 0
  max_weight: 100
  weight_increment: 10
  num_sims: 50
  years: 10
  frequency: "monthly"
```

### Option 2 : Notebook Jupyter

Pour une utilisation interactive avec visualisations :

```bash
jupyter notebook optimize_invest.ipynb
```

## Validation des Fichiers YAML

Les fichiers de configuration sont validés automatiquement avec Pydantic. En cas d'erreur, un message clair indique le problème :

```
Erreur de validation dans config/portfolios/test.yaml:
  - portfolio -> weights: La somme des poids doit être égale à 1 (actuel: 0.5)
  - securities -> 0 -> identifier: Field required
  - ...
```

## Prix des Securities (Actifs)

L'API Alphavantage est utilisée pour collecter les prix mensuels des actifs. À partir des prix, on calcule le rendement moyen, la volatilité (écart-type des rendements), et les corrélations des rendements entre chaque actifs sous forme de matrice de covariance.

## Simulation de Monte Carlo

TODO

## Frontière Efficiente

TODO

### Pondérations les plus Efficientes

TODO, detailler les divers critères et méthodes de classements

## Annexe

### Classes de Titres

#### Security (classe de base)

- `name` : nom du titre
- `identifier` : ticker (utilisé pour interroger Alphavantage)
- `mu` : rendement moyen annuel
- `sigma` : volatilité annuelle (écart-type des rendements)

#### Equity

Titre classique (actions, ETF) avec mu et sigma définis manuellement.

#### LeveragedEquity

Titre à effet de levier basé sur un autre titre. Les rendements sont calculés automatiquement à partir du titre sous-jacent et du facteur de levier.

### Classe Portfolio

Un portefeuille contient des titres avec des pondérations spécifiques.

**Attributs :**

- `name` : nom du portefeuille
- `securities` : liste de titres
- `target_weights` : pondérations associées à chaque titre
- `portfolio_value` : valeur initiale du portefeuille
- `rf` : taux sans risque (format décimal)
- `cov` : matrice de covariance des rendements
- `simulation_results` : résultats de la simulation Monte-Carlo
- `mean_return`, `mean_volatility`, `sharpe_ratio` : métriques calculées

**Méthodes principales :**

- `calc_covariance()` : calculer la matrice de covariance
- `get_security_prices(yr, plot, requests_per_min)` : récupérer les prix via Alpha Vantage
- `run_simulation(simulations, years, frequency, rebalancing)` : exécuter la simulation
- `build_efficient_frontier(...)` : construire la frontière efficiente
- `get_optimal_portfolios_by_sharpe_ratio(top_n)` : obtenir les meilleurs portefeuilles
- `plot_portfolio_simulations()`, `plot_efficient_frontier()`, `plot_boxplots()` : visualisations

### Fonctions Métier (src/main.py)

- `create_portfolio(securities, ...)` : créer un portefeuille à partir d'objets Security
- `run_simulate(portfolio, ...)` : exécuter une simulation Monte-Carlo
- `run_frontier(portfolio, ...)` : construire la frontière efficiente
