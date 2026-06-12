# FIFA World Cup 2026 Prediction Project

## Table of Contents

- [Project Overview](#project-overview)
- [Introduction](#introduction)
- [Data](#data)
- [Methodology](#methodology)
  - [Data Preparation](#data-preparation)
  - [Team Strength Model (Poisson Regression)](#team-strength-model-poisson-regression)
  - [Group Stage Simulation](#group-stage-simulation)
  - [Knockout Stage Prediction](#knockout-stage-prediction)
- [Validation](#validation)

## Introduction

This project aims to predict the outcomes of the 2026 FIFA World Cup matches, from the group stage through to the final. With 48 teams and 104 matches, the tournament presents a significant challenge for prediction. The goal is to provide accurate forecasts for match scores, corners, and cards, adhering to a precision-rewarding scoring system where exact scorelines and correct winners earn maximum points.

## Data

The following datasets are utilized:

-   `data/group_fixtures.csv`: Details of all 72 group stage matches, including `match_id`, `group`, `home_team`, `away_team`, `date`, and `venue`.
-   `data/knockout_slots.csv`: Information for all 32 knockout round slots, specifying `match_id`, `round`, `multiplier`, `slot_home`, and `slot_away`.
-   `fifa_ranking.csv`: External data containing FIFA team rankings.
-   `results.csv`: A comprehensive dataset of international football match results from 1872 to 2023.

## Methodology

### Data Preparation

1.  **Loading Data**: Group fixtures, knockout slots, FIFA rankings, and historical match results are loaded into pandas DataFrames.
2.  **Cleaning Historical Results**: Rows with missing scores (representing future or unplayed matches) are removed. Dates are converted to datetime objects.
3.  **Filtering Recent Matches**: Historical results are filtered to include only non-friendly matches from January 1, 2018, onwards, to ensure the model learns from relevant and recent team performances.
4.  **Handling Playoff Teams**: Placeholder teams like 'UEFA Playoff A' are mapped to specific national teams based on a predefined `playoff_map`.

### Team Strength Model (Poisson Regression)

A Poisson regression model is implemented to estimate attack and defense ratings for each team. The model assumes that the number of goals scored by a team in a match follows a Poisson distribution, whose mean is influenced by the team's attack strength, the opponent's defense strength, and a home-field advantage.

-   **Parameters**: Attack ratings for each team, defense ratings for each team, and a universal home advantage parameter.
-   **Optimization**: The `scipy.optimize.minimize` function is used to find the parameters that maximize the log-likelihood of observing the actual match scores.

### Group Stage Simulation

1.  **Match Prediction Functions**: Functions `expected_goals`, `predict_scoreline`, `win_probabilities`, and `pick_winner` are defined using the Poisson distribution based on the learned attack and defense ratings.
2.  **Monte Carlo Simulation**: To account for variability and determine group standings, a Monte Carlo simulation with `N = 10000` iterations is performed:
    -   Each iteration simulates all group stage matches by drawing random scores from Poisson distributions parameterized by the expected goals.
    -   Group standings are updated based on simulated match outcomes (points, goal difference, goals scored).
    -   Group winners, runners-up, and best third-place teams are determined for each simulation.
3.  **Consensus**: The most frequently occurring winners, runners-up, and best third-place teams across all simulations are identified as the final group stage predictions.

### Knockout Stage Prediction

1.  **Populating Knockout Slots**: The predicted group stage outcomes (winners, runners-up, and best third-place teams) are used to populate the initial Round of 32 slots.
2.  **Round-by-Round Prediction**: Matches are predicted sequentially. The winners of earlier knockout matches feed into the slots for subsequent rounds.
3.  **Penalties**: A simple rule is applied where matches with a close strength difference (absolute difference in win probabilities < 0.15) are predicted to go to penalties, reflecting a historical base rate of ~27% for knockout matches.

## Validation

-   **Backtest Dataset**: A backtest dataset is created using past FIFA World Cup matches (before 2018) that were not used in the model training.
-   **Scoring Mechanism**: Custom functions `score_prediction` and `score_winner` are defined to replicate the competition's scoring rules for exact scores, correct goal difference/total goals, and correct match winners.
-   **Performance Metrics**: The model's performance on the backtest set is evaluated by calculating average score points per match, average winner points per match, exact score rate, and correct winner rate.
