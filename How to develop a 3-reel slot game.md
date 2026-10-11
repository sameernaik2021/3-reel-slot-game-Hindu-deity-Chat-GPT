# How to develop a 3-reel slot game 🎰

You can build a simple **3-reel slot game using HTML, CSS, and JavaScript**. It can run in a web browser on a computer or mobile phone without installing anything.

I'll show you how to create a working prototype with:

- 🎰 Three spinning reels.
- 🍒 Symbols such as cherries, lemons, bells, and lucky sevens.
- 🎲 Random symbol selection.
- 💰 A virtual-credit balance and adjustable bet.
- 🏆 Winning-combination detection.
- ✨ A spinning animation and win messages.

We'll build a **demo using virtual credits only**, so you can learn the mechanics without real-money gambling or payment processing.

## 1\. How the game works

The basic game flow is:

1. The player starts with 1,000 virtual credits.
2. The player chooses a bet.
3. The player presses **SPIN**.
4. The game randomly selects one symbol for each reel.
5. The game checks the three symbols against its payout rules.
6. The game deducts the bet and awards any winnings.

For example:

| Reel 1 | Reel 2 | Reel 3 | Example result |
| --- | --- | --- | --- |
| 🍒 | 🍒 | 🍒 | Win |
| 🍋 | 🍋 | 🍋 | Win |
| 7️⃣ | 7️⃣ | 7️⃣ | Larger win |
| 🍒 | 🔔 | 🍋 | No win |

These are example rules. You can define your own winning combinations and payouts.

## 2\. Complete working game code

Create a file named `slot.html` and paste the following code into it.

The game includes three reels, virtual credits, adjustable bets, spinning animations, and a payout system.

HTMLWrap linesCopiedPreview

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>3-Reel Slot Game</title>

  <style>
    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      min-height: 100vh;
      display: grid;
      place-items: center;
      padding: 20px;
      background: #101525;
      color: white;
      font-family: Arial, sans-serif;
    }

    .game {
      width: 100%;
      max-width: 420px;
      padding: 25px;
      text-align: center;
      background: #202a40;
      border: 2px solid #e7b94e;
      border-radius: 22px;
    }

    h1 {
      margin-top: 0;
      color: #ffd45c;
    }

    .balance {
      font-size: 19px;
      margin: 20px 0;
    }

    .reels {
      display: flex;
      justify-content: center;
      gap: 10px;
      margin: 25px 0;
    }

    .reel {
      width: 95px;
      height: 100px;
      display: grid;
      place-items: center;
      background: #fff;
      color: #111;
      border: 3px solid #e7b94e;
      border-radius: 12px;
      font-size: 48px;
    }

    .reel.spinning {
      animation: shake 0.12s infinite alternate;
    }

    @keyframes shake {
      from { transform: translateY(-3px); }
      to { transform: translateY(3px); }
    }

    .controls {
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 10px;
      flex-wrap: wrap;
      margin-bottom: 20px;
    }

    button, select {
      padding: 12px 16px;
      border: none;
      border-radius: 8px;
      font-size: 16px;
    }

    select {
      background: white;
      color: #111;
    }

    button {
      cursor: pointer;
      font-weight: bold;
    }

    button:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }

    #spin {
      width: 100%;
      padding: 16px;
      background: #ffd45c;
      color: #171717;
      font-size: 20px;
    }

    #message {
      min-height: 30px;
      margin: 20px 0 10px;
      font-size: 18px;
      font-weight: bold;
    }

    .hint {
      color: #c6cede;
      font-size: 13px;
      line-height: 1.6;
    }
  </style>
</head>

<body>
  <main class="game">
    <h1>🎰 Lucky Reels</h1>

    <div class="balance">
      Credits: <strong id="balance">1000</strong>
    </div>

    <div class="reels" aria-label="Slot machine reels">
      <div class="reel" id="reel0">🍒</div>
      <div class="reel" id="reel1">🍋</div>
      <div class="reel" id="reel2">🔔</div>
    </div>

    <div class="controls">
      <label for="bet">Bet:</label>
      <select id="bet">
        <option value="10">10 credits</option>
        <option value="25">25 credits</option>
        <option value="50">50 credits</option>
        <option value="100">100 credits</option>
      </select>
    </div>

    <button id="spin">SPIN</button>

    <p id="message" aria-live="polite">
      Good luck!
    </p>

    <p class="hint">
      Match three symbols to win.<br>
      🍒 = 2× &nbsp; 🍋 = 3×<br>
      🔔 = 5× &nbsp; 💎 = 10× &nbsp; 7️⃣ = 20×<br>
      Payouts include the returned bet.
    </p>
  </main>

  <script>
    const symbols = [
      { icon: "🍒", multiplier: 2, weight: 35 },
      { icon: "🍋", multiplier: 3, weight: 28 },
      { icon: "🔔", multiplier: 5, weight: 20 },
      { icon: "💎", multiplier: 10, weight: 12 },
      { icon: "7️⃣", multiplier: 20, weight: 5 }
    ];

    let credits = 1000;
    let spinning = false;

    const balanceEl = document.getElementById("balance");
    const betEl = document.getElementById("bet");
    const spinButton = document.getElementById("spin");
    const messageEl = document.getElementById("message");

    const reels = [
      document.getElementById("reel0"),
      document.getElementById("reel1"),
      document.getElementById("reel2")
    ];

    function updateBalance() {
      balanceEl.textContent = credits;
    }

    // Select a symbol using its configured weight.
    function randomSymbol() {
      const totalWeight = symbols.reduce(
        (sum, symbol) => sum + symbol.weight, 0
      );

      let roll = Math.random() * totalWeight;

      for (const symbol of symbols) {
        roll -= symbol.weight;

        if (roll < 0) {
          return symbol;
        }
      }

      return symbols[0];
    }

    function sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    }

    async function spin() {
      if (spinning) return;

      const bet = Number(betEl.value);

      if (!Number.isInteger(bet) || bet <= 0) {
        messageEl.textContent = "Invalid bet.";
        return;
      }

      if (credits < bet) {
        messageEl.textContent = "Not enough credits!";
        return;
      }

      spinning = true;
      spinButton.disabled = true;
      betEl.disabled = true;

      credits -= bet;
      updateBalance();

      messageEl.textContent = "Spinning...";

      reels.forEach(reel => reel.classList.add("spinning"));

      // Determine the final result before displaying it.
      const result = [
        randomSymbol(),
        randomSymbol(),
        randomSymbol()
      ];

      // Animate and stop each reel in sequence.
      for (let i = 0; i < reels.length; i++) {
        const duration = 800 + i * 500;
        const start = Date.now();

        while (Date.now() - start < duration) {
          reels[i].textContent = randomSymbol().icon;
          await sleep(80);
        }

        reels[i].textContent = result[i].icon;
        reels[i].classList.remove("spinning");
      }

      const allMatch = result.every(
        symbol => symbol.icon === result[0].icon
      );

      if (allMatch) {
        const multiplier = result[0].multiplier;
        const payout = bet * multiplier;

        credits += payout;
        messageEl.textContent =
          `🎉 JACKPOT! You won ${payout} credits!`;
      } else {
        messageEl.textContent =
          "No win this time. Try again!";
      }

      updateBalance();

      if (credits === 0) {
        messageEl.textContent += " No credits left to play.";
      }

      spinning = false;
      spinButton.disabled = false;
      betEl.disabled = false;
    }

    spinButton.addEventListener("click", spin);
  </script>
</body>
</html>
```

Copied

## 3\. How to run the game

You don't need a server to test this basic version.

1. Open Notepad, VS Code, or another text editor.
2. Copy the code above.
3. Save it as `slot.html`. In Notepad, choose **Save as type: All Files** so it doesn't become `slot.html.txt`.
4. Double-click the file to open it in Chrome, Edge, or another browser.
5. Select your bet and click **SPIN**.

You now have a basic, playable 3-reel slot game.

## 4\. Understand the important JavaScript functions

### A. Symbol configuration

```
const symbols = [
  { icon: "🍒", multiplier: 2, weight: 35 },
  { icon: "🍋", multiplier: 3, weight: 28 },
  { icon: "🔔", multiplier: 5, weight: 20 },
  { icon: "💎", multiplier: 10, weight: 12 },
  { icon: "7️⃣", multiplier: 20, weight: 5 }
];
```

Each symbol has three properties:

- `icon`: The symbol displayed on the reel.
- `multiplier`: The payout multiplier for three matching symbols.
- `weight`: The relative probability of selecting that symbol.

The weights total 100, so the selection probabilities are 35%, 28%, 20%, 12%, and 5%, respectively.

These probabilities apply to each independently selected reel in this prototype. They do not mean the probability of winning is 5% or 20%; the probability of three matching symbols is different.

### B. Random symbol selection

```
function randomSymbol() {
  const totalWeight = symbols.reduce(
    (sum, symbol) => sum + symbol.weight, 0
  );

  let roll = Math.random() * totalWeight;

  for (const symbol of symbols) {
    roll -= symbol.weight;

    if (roll < 0) return symbol;
  }

  return symbols[0];
}
```

This function selects a symbol according to its configured weight.

**Important:** `Math.random()` is suitable for a casual prototype, not a production gambling system. For a real-money game, the outcome generator, payout logic, and accounting would need appropriate security, independent testing, and regulatory review.

### C. Calculate winnings

```
const allMatch = result.every(
  symbol => symbol.icon === result[0].icon
);

if (allMatch) {
  const payout = bet * result[0].multiplier;
  credits += payout;
}
```

If all three symbols match, the game calculates the payout using the selected bet and the matching symbol's multiplier.

For example, a bet of 10 credits with three matching diamonds produces a payout of 100 credits.

The code deducts the original bet before spinning, so the payout includes the return of the winning bet.

---

## 5\. How to calculate the game's theoretical return

When developing a slot game, one important metric is the **Return to Player (RTP)**.

RTP is the theoretical percentage of total wagers returned to players over a very large number of plays.

For the example game's independent reel selections, the probability of getting three matching symbols is:

$$
P(\text{three matching})=\sum_{i=1}^{n}p_i^3
$$

Here, $p_i$ is the probability of selecting symbol $i$ on a single reel.

Because only three matching symbols pay in this prototype, the theoretical RTP is:

$$
\text{RTP}=\sum_{i=1}^{n}p_i^3m_i
$$

where $m_i$ is the winning payout multiplier, including the returned bet.

For the symbol weights and multipliers in the code above, this gives approximately **18.65% RTP**.

That is a mathematical expectation for this particular configuration, not a guarantee for any individual session. It also illustrates why you should calculate the payout model before releasing a game: the example is deliberately simple, and its theoretical return is low.

A production game would need a carefully designed probability model, a verified paytable, and appropriate testing.

## 6\. What to add next

Once the basic version works, you can expand it with:

- **Multiple paylines:** Allow wins on different horizontal or diagonal patterns.
- **More reel positions:** Display several symbols vertically on each reel.
- **Wild symbols:** Let a designated symbol substitute for other symbols.
- **Scatter symbols:** Award bonuses based on symbol counts or positions.
- **Sound effects:** Add spin, stop, and win sounds.
- **Mobile-friendly design:** Improve touch controls, animation, and layout.
- **Game statistics:** Track spins, total bets, total payouts, and simulated RTP.
- **Backend services:** Store accounts and game records securely if you later need a server-based application.

For a browser-based prototype, HTML, CSS, and JavaScript are sufficient. A backend becomes useful when you need persistent accounts, shared game state, or server-controlled outcomes.

**One important distinction:** A playable demo is not the same as a production gambling platform. Real-money wagering introduces security, payment, licensing, and jurisdiction-specific legal requirements.

### One question so I can guide you further

**Where do you want to run your 3-reel slot game?**

1. **Website** — HTML, CSS, and JavaScript.
2. **Android app** — A game designed for Android phones.
3. **PC game** — A desktop application.
4. **Professional casino-style game** — Multiple paylines, bonus rounds, configurable RTP, and a more advanced game architecture.

I can tailor the next version to your choice.
