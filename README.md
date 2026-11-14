# 🤖 Can the Bot Read You?

### A rock-paper-scissors experiment on how predictable humans really are

![Status](https://img.shields.io/badge/status-data%20collection-orange)
![Built with](https://img.shields.io/badge/built%20with-JavaScript-yellow)
![Hosting](https://img.shields.io/badge/hosting-Firebase-ffca28)
![License](https://img.shields.io/badge/license-MIT-green)

**▶ Play it live: [rps-bot-8c916.web.app](https://rps-bot-8c916.web.app)**  
**💻 Source code: [github.com/junupanda/rps-bot](https://github.com/junupanda/rps-bot)**

A web game where you play 30 rounds of rock-paper-scissors against a bot that tries to learn your habits as you play. Every round is saved anonymously, so we can test a simple question with real data: **does a pattern-learning bot beat the 33% you would get from pure luck?**

This is a group project for **QRMT (Quantitative Reasoning and Mathematical Thinking)**.

---

## 🎯 Objectives

1. **Test whether humans are predictable.** If people played truly randomly, any bot would win exactly 1/3 of rounds. We measure how far real play deviates from that.
2. **Find out how many games it takes to be sure.** A short run can look extreme purely by luck. We show how the win rate settles down as the sample grows.
3. **Check that randomness cannot be beaten.** A random number generator playing the bot should land near 33%, which confirms the bot is not cheating.
4. **Apply course concepts to real data:** expected value, standard deviation, the normal approximation, Monte Carlo simulation, the Law of Large Numbers, random number generation, and linear regression.

## 🧠 The research question

> If people played rock-paper-scissors randomly, any bot would win 33% of rounds. Does a bot trained on human play beat 33% against real players, and how many games do we need before we can trust the result?

---

## 🎮 How the game works

1. Pick Rock, Paper or Scissors.
2. A short "Rock... Paper... Scissors... SHOOT!" countdown plays, then the bot's move is revealed.
3. The scoreboard, a 30-square progress bar, and a strip of your recent rounds (your move on top, the bot's below) show how the game is going.
4. After each round the bot shows what it guessed you would throw, and whether it was right.
5. After 30 rounds you see your result and how often the bot read you.

**Fairness:** the bot chooses its move *before* you tap, using only past rounds. It never sees your current move, and it only learns after the result is revealed.

## 🔍 How the bot thinks

The bot runs several small pattern detectors at the same time and follows whichever one has been working best on you recently.

| Pattern it watches | Example |
|---|---|
| Overall favorite | "You throw Rock a lot" |
| Recent streak | "Your last 8 throws lean Paper" |
| Last move | "After Rock, you usually throw Scissors" |
| Last two / three moves | "After Rock then Paper, you throw Rock" |
| Reaction to results | "After a loss, you switch to Paper" |
| Reaction to the bot's move | "After the bot throws Rock, you throw Paper" |

For each pattern it also tests three versions: *you will repeat it*, *you will counter it*, and *you will counter the counter*. That is how it catches players who adapt after seeing its guesses. Old habits fade gradually, so it keeps up when you change style. If no pattern is beating luck, it plays randomly instead of guessing badly.

Full details are in [docs/METHODOLOGY.md](docs/METHODOLOGY.md).

---

## 📐 The math behind it

Each round is a win or not-a-win for the bot, with a probability of 1/3 under random play (ties count as not-a-win). The win rate over *n* rounds has standard deviation √(⅓ · ⅔ / n), so luck alone moves it by:

| Rounds | Typical swing from 33% (1 SD) |
|---:|---:|
| 10 | ± 14.9 points |
| 30 | ± 8.6 points |
| 100 | ± 4.7 points |
| 300 | ± 2.7 points |
| 500 | ± 2.1 points |

This is why one person's 30-round game cannot tell us much, and why the project needs lots of players.

---

## 🛠 Tech stack

- **Front end:** plain HTML, CSS and JavaScript in a single page (no framework)
- **Hosting:** Firebase Hosting
- **Database:** Cloud Firestore
- **Identity:** Firebase Anonymous Authentication (no names, no logins)
- **Analysis (planned):** Python or Excel on an exported CSV

## 📁 Repository structure

```
rps-bot/
├── public/
│   └── index.html        # the whole game and bot
├── docs/
│   └── METHODOLOGY.md    # bot design, data cleaning, analysis plan, limitations
├── firestore.rules       # security rules (players can only add their own rounds)
├── firebase.json         # Firebase Hosting config
├── .firebaserc           # Firebase project link
├── LICENSE
└── README.md
```

## 💾 What gets recorded

Each round is saved as one anonymous row:

| Field | Meaning |
|---|---|
| `uid` | Anonymous player ID (random, not linked to a name) |
| `group` | `public` or `team` (team members use `?g=team` in the link) |
| `session`, `game_no` | Which game, and that player's game number |
| `round` | Round number, 1 to 30 |
| `player`, `bot` | Moves played (`R`, `P`, `S`) |
| `result` | `W` (you won), `L` (bot won), `T` (tie) |
| `prev_move`, `prev_result` | What happened the round before |
| `informed` | 1 if the bot made a real guess, 0 if it played randomly |
| `rt_ms` | Reaction time in milliseconds |
| `run_len` | How many times in a row the same move was played |
| `ts` | Server timestamp |

No names, emails, or personal details are collected.

## 🔒 Privacy and security

- Players are identified only by a random anonymous ID.
- The Firebase config in `public/index.html` is meant to be public in web apps. Access is controlled by [`firestore.rules`](firestore.rules): players can only **add** correctly formatted rounds, and nobody can read, edit or delete data through the website.
- Private keys and service-account files are never committed (see `.gitignore`).

---

## 🚀 Run it yourself

**Play online:** open the live link above.

**Run locally:** open `public/index.html` in a browser. The game works, but rounds will not save unless it is served from a Firebase-authorized domain.

**Deploy your own copy**
```bash
npm install -g firebase-tools
firebase login
firebase init hosting     # public directory: public, single-page app: No
firebase deploy --only hosting
```
Then create your own Firebase project, enable Anonymous sign-in and Firestore, publish `firestore.rules`, and paste your own `firebaseConfig` into `public/index.html`.

---

## 📊 Results

*Data collection is in progress. Results will be added here after analysis.*

| Measure | Result |
|---|---|
| Games played | _TBD_ |
| Players | _TBD_ |
| Bot win rate (all players) | _TBD_ |
| Bot read rate on informed guesses | _TBD_ |
| Different from 33%? | _TBD_ |

## ⚠️ Limitations

- Players can see the bot's guesses, so some will deliberately adapt to it.
- Players choose to take part, so the sample is not random.
- Rounds from the same person are not fully independent.
- A person's bot memory lives in their browser, so clearing it resets the bot for that person.
- People who rush or spam buttons are filtered out, using thresholds listed in the methodology.

## 👥 Team

| Name | Role |
|---|---|
| Thakur Garg | Bot, game, data collection |
| _Name 2_ | _TBD_ |
| _Name 3_ | _TBD_ |
| _Name 4_ | _TBD_ |
| _Name 5_ | _TBD_ |

## 📄 License

Released under the [MIT License](LICENSE).
