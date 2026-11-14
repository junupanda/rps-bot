# Can the Bot Read You?

A rock-paper-scissors experiment on how predictable people are. You play 30 rounds against a bot that tries to learn your habits as you go, and every round is saved anonymously so we can check the results with real data.

Play it here: https://rps-bot-8c916.web.app

This is a group project for QRMT.

## The question

If people played rock-paper-scissors randomly, any bot would win about 33% of rounds. So we ask: does a bot that learns from human play do better than 33% against real players, and how many games do we need before we can trust the answer?

We also want to check the opposite case. A random number generator playing the same bot should land near 33%, because nothing can predict true randomness.

## What we are trying to show

1. How far real human play is from random.
2. How much a small sample can mislead. Ten games can look very good or very bad purely by luck, and the win rate settles down as the number of games grows.
3. That the bot is fair and does not cheat, using the random-generator check.
4. How course ideas (expected value, standard deviation, the normal approximation, Monte Carlo simulation, the Law of Large Numbers, random number generation and linear regression) apply to a real dataset.

## How the game works

You pick rock, paper or scissors. A short countdown plays, then the bot's move is revealed along with who won. The page keeps a running scoreboard, a 30-square progress bar, and a short history of recent rounds (your move on top, the bot's underneath). After each round the bot also shows what it guessed you would throw and whether it was right.

The bot chooses its move before you tap, using only earlier rounds, and it learns only after the result is shown. It never sees your current move.

## How the bot works

The bot watches several simple patterns at once and follows whichever has been predicting you best lately. The patterns are:

- your overall favourite move
- your last few moves
- what you usually throw after your previous move
- what you usually throw after your previous two or three moves
- how you react to winning, losing or tying
- how you react to the bot's previous move

For each pattern it also tries two twists: that you will counter it, and that you will counter the counter. This lets it keep up with players who change their play after seeing its guesses. Old habits count for less over time. If no pattern is doing better than luck, the bot plays randomly instead of guessing badly.

The bot's memory is kept in each player's own browser, so it only learns from that one person, and everyone starts with the same blank bot. More detail is in `docs/METHODOLOGY.md`.

## The maths

For each round, the bot either wins or it does not (ties count as not winning). Under random play the chance of a bot win is 1/3. Over n rounds, the win rate varies from 33% by a standard deviation of sqrt((1/3)(2/3)/n):

| Rounds | Typical swing from 33% |
|---|---|
| 10 | 14.9 points |
| 30 | 8.6 points |
| 100 | 4.7 points |
| 300 | 2.7 points |
| 500 | 2.1 points |

So a single 30-round game says very little. That is the reason we are collecting games from many people.

## Data we record

Each round is stored as one row with no names or personal details:

- an anonymous random player ID
- whether the player is in the public group or the team group (team members use `?g=team` at the end of the link)
- the session and game number
- the round number (1 to 30)
- the player's move and the bot's move
- the result (win, loss or tie)
- the previous move and previous result
- whether the bot made a real guess or played randomly
- reaction time in milliseconds
- how many times in a row the same move was played
- a server timestamp

Before analysis we remove sessions that look like button mashing: fewer than 10 rounds, or ten or more identical moves in a row. We report how many were removed. The full rules are in `docs/METHODOLOGY.md`.

## Built with

- HTML, CSS and JavaScript in a single page, with no framework
- Firebase Hosting for the site
- Cloud Firestore for storing rounds
- Firebase Anonymous Authentication, so no one has to log in

## Files

- `public/index.html`: the game and the bot
- `docs/METHODOLOGY.md`: bot design, data cleaning rules, analysis plan and limitations
- `firestore.rules`: security rules, so players can only add correctly formatted rounds and cannot read, change or delete any data
- `firebase.json` and `.firebaserc`: Firebase configuration

The Firebase config inside `index.html` is meant to be public in web apps. Access to the data is controlled by the rules file. Private keys are never committed.

## Running it yourself

Open `public/index.html` in a browser to see the game. Rounds only save when the page is served from the Firebase project's own domain.

To deploy your own copy, create a Firebase project, turn on Anonymous sign-in and Firestore, publish `firestore.rules`, paste your own config into `public/index.html`, and run:

```
npm install -g firebase-tools
firebase login
firebase deploy --only hosting
```

## Results

Data collection is still in progress. This section will be filled in after analysis.

## Limitations

- Players can see the bot's guesses, so some will adapt on purpose.
- People choose whether to play, so the sample is not random.
- Rounds from the same person are not fully independent.
- A player who clears their browser data resets the bot's memory for them.
- The bot is deliberately simple. A different model could do better or worse.

## Team

- Thakur Garg: game, bot and data collection
- Other members to be added

## License

MIT. See `LICENSE`.