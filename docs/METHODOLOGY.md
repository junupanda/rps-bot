# Methodology

This document explains how the bot works, how the data is collected and cleaned, and how it will be analyzed.

## 1. Research question

If people played rock-paper-scissors randomly, any bot would win 33% of rounds. Does a bot trained on human play beat 33% against real players, and how many games do we need before we can trust the result?

**Null hypothesis:** the bot's win probability per round is 1/3.
**Alternative:** it is greater than 1/3 (humans have exploitable habits).

Ties count as "not a bot win", so the baseline is exactly 1/3 against random play.

## 2. The bot

**Timing.** The bot picks its move before the player taps, using only earlier rounds. It updates its memory only after the result is revealed. It never sees the current move.

**Pattern detectors ("experts").** Each one predicts the player's next move from a different kind of history:

| Expert | Looks at |
|---|---|
| `freq` | the player's overall move counts |
| `recent` | the last 8 moves |
| `o1` | the previous move |
| `o2` | the previous two moves |
| `o3` | the previous three moves |
| `mr` | the previous move and whether it won, lost or tied |
| `bm` | the bot's previous move |

**Shifted versions.** Each expert is used three ways: predict the move directly (shift 0), predict the player will counter it (shift 1), or counter the counter (shift 2). Together with the seven experts this gives 21 candidate strategies.

**Choosing a strategy.** Each candidate keeps a running accuracy score, updated after every round with a decay of 0.9, so recent performance matters most. The bot follows the best-scoring candidate if its score is at least 0.40 (above the 0.33 chance level). Otherwise it plays a random move.

**Forgetting.** Habit counts are multiplied by 0.95 each time they are updated, so old habits fade. A situation needs about two sightings before the bot will guess from it.

**Per-player memory.** The bot's memory is stored in each player's own browser, so it adapts only to that player. Everyone starts with the same blank bot.

**Parameters (fixed before data collection)**

| Parameter | Value |
|---|---|
| Score decay | 0.9 |
| Confidence threshold | 0.40 |
| Minimum sightings | 1.5 (about 2) |
| Habit-count decay | 0.95 |
| Rounds per game | 30 |

The bot logic is frozen once public data collection starts, so all results come from the same bot.

## 3. Data collection

- Players open the public link and play 30-round games. Players can play as many games as they like.
- Each browser receives a random anonymous ID through Firebase Anonymous Authentication.
- Every round is stored in Cloud Firestore with the fields listed in the README.
- Team members use a link ending in `?g=team`, so their games can be analyzed separately or excluded.

## 4. Data cleaning rules

Rules are set in advance so that cleaning does not depend on the results. The number of games and rounds removed under each rule will be reported.

| Rule | Reason |
|---|---|
| Drop sessions with fewer than 10 rounds | too short to show any pattern |
| Drop sessions containing a run of 10 or more identical moves | looks like button-mashing, not real play |
| Flag sessions with very fast median reaction time (for example under 300 ms) | likely mashing without reading the screen |
| Analyze `team` and `public` groups separately, and together | check that results do not depend on who played |

Thresholds may be adjusted if the early data shows they are clearly too strict or too loose, and any change will be documented here.

## 5. Analysis plan

1. **Baseline simulation (Monte Carlo).** Simulate many games of the bot against a perfectly random player and plot the distribution of win rates. It should be centered near 33%.
2. **Expected variation.** Compute the standard deviation of the win rate for different sample sizes, and compare with the observed spread across players.
3. **Observed win rate.** Compute the bot's overall win rate and a confidence interval using the normal approximation, and test it against 1/3.
4. **Per-player view.** Compute each player's results and compare them with the pooled result. Players with more games are noted, since they carry more weight when pooled.
5. **Small vs. large samples.** Show win rates from 10-game samples next to the full dataset to illustrate the Law of Large Numbers.
6. **Learning curve.** Fit a line of best fit to the bot's win rate (and its hit rate on informed guesses) against round number, using linear regression, to see whether it improves as it sees more play.
7. **Random-generator control.** Let a random number generator (1 = rock, 2 = paper, 3 = scissors) play the bot over many rounds. A result near 33% shows the bot cannot beat true randomness.

## 6. Limitations

- Players can see the bot's guesses and may change their play in response.
- Participants are volunteers, so the sample is not random.
- Repeated games by the same person are not independent observations.
- A player who clears their browser data resets their bot's memory.
- The bot is intentionally simple. A different or stronger model could do better or worse.
- Tie handling affects the baseline, so all reported win rates count ties as non-wins.

## 7. Course concepts used

- Expected value and variance
- Binomial distribution, normal approximation and the Central Limit Theorem
- Monte Carlo simulation and the Law of Large Numbers
- Random number generation
- Linear regression and line of best fit
