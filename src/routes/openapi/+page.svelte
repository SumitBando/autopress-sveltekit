<script>
  import { writable } from 'svelte/store';

  // Store to track the score and history
  let holes = writable([]);
  let score = writable([1, 1, 1]); // Initial score 1 1 1
  let currentHole = writable(1); // Track the current hole
  let firstWin = writable(true); // Track whether this is the first win

  // Handle a win/halved outcome
  function handleHoleResult(result) {
    currentHole.update(n => {
      if (n < 19) {
        holes.update(history => {
          let newScore = [...$score]; // Clone current score

          if (result === 'A') {
            newScore = updateScore(newScore, 'A');
          } else if (result === 'B') {
            newScore = updateScore(newScore, 'B');
          } // No change on Halved

          score.set(newScore);
          history.push({ hole: n, result, score: newScore });
          return history;
        });

        return n + 1; // Increment hole number
      }
      return n;
    });
  }

  // Update score based on team win
  function updateScore(currentScore, winner) {
    let updatedScore = [...currentScore];

    // Only allow score to stay as 1 1 1 for the very first win
    if ($firstWin) {
      firstWin.set(false); // After the first win, allow score changes
      return currentScore; // Keep it 1 1 1 after the first hole win
    }

    if (winner === 'A') {
      updatedScore = updateForWin(updatedScore, 'A');
    } else if (winner === 'B') {
      updatedScore = updateForWin(updatedScore, 'B');
    }

    // Auto press rule - add new match if right-most match becomes 2
    if (updatedScore[updatedScore.length - 1] === 2) {
      updatedScore.push(0); // Add a neutral match
    }

    return updatedScore;
  }

  function updateForWin(score, winner) {
    let updatedScore = [...score];

    // Add 1 to the winner's matches, subtract 1 from the loser's matches
    if (winner === 'A') {
      updatedScore = updatedScore.map((match, index) => {
        if (match === 0) return 1; // Neutral to A's favor
        if (index % 2 === 0) return match + 1; // A's favor for 1st and 3rd
        return Math.max(match - 1, 0); // Subtract from B's favor (middle)
      });
    } else if (winner === 'B') {
      updatedScore = updatedScore.map((match, index) => {
        if (match === 0) return 1; // Neutral to B's favor
        if (index % 2 === 1) return match + 1; // B's favor for middle
        return Math.max(match - 1, 0); // Subtract from A's favor
      });
    }

    return updatedScore;
  }
</script>

<main>
  {#each $holes as hole}
    <div>
      Hole {hole.hole}: {hole.result} - Score: {hole.score.join(' ')}
    </div>
  {/each}

  {#if $currentHole < 19}
    <div>
      <h3>Hole {$currentHole}</h3>
      <button on:click={() => handleHoleResult('A')}>A Wins</button>
      <button on:click={() => handleHoleResult('Halved')}>Halved</button>
      <button on:click={() => handleHoleResult('B')}>B Wins</button>
    </div>
  {:else}
    <div>
      <h3>Game Over</h3>
      <p>Final Score: {$score.join(' ')}</p>
    </div>
  {/if}
</main>

<style>
  button {
    margin: 5px;
  }
</style>
