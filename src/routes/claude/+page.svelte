<script>
  import { writable } from 'svelte/store';

  // Game state
  const gameState = writable({
    currentHole: 1,
    scores: [],
    gameOver: false
  });

  // Function to calculate score based on previous score and winner
  function calculateScore(prevScore, winner) {
    if (!prevScore) {
      return winner === 'A' ? [1, 1, 1] : [0, 2, 0];
    }

    let newScore = [...prevScore];
    
    for (let i = 0; i < newScore.length; i++) {
      if (winner === 'A') {
        newScore[i] += i % 2 === 0 ? 1 : -1;
      } else {
        newScore[i] += i % 2 === 1 ? 1 : -1;
      }
      newScore[i] = Math.max(0, newScore[i]);
    }

    // Auto press rule
    if (newScore[newScore.length - 1] === 2) {
      newScore.push(0);
    }

    return newScore;
  }

  // Function to handle button click
  function handleClick(winner) {
    gameState.update(state => {
      const prevScore = state.scores.length > 0 ? state.scores[state.scores.length - 1].score : null;
      const newScore = winner === 'Halved' ? prevScore : calculateScore(prevScore, winner);
      
      state.scores.push({
        hole: state.currentHole,
        winner,
        score: newScore
      });

      state.currentHole++;
      state.gameOver = state.currentHole > 18;

      return state;
    });
  }

  // Function to format score for display
  function formatScore(score) {
    if (!score) return 'Halved';
    return score.join('');
  }
</script>

<main>
  <h1>Autopush Golf Scoring</h1>

  {#if $gameState.gameOver}
    <h2>Game Over</h2>
    <p>Final Score: {formatScore($gameState.scores[$gameState.scores.length - 1].score)}</p>
  {:else}
    <h2>Hole {$gameState.currentHole}</h2>
    <div>
      <button on:click={() => handleClick('A')}>A Wins</button>
      <button on:click={() => handleClick('Halved')}>Halved</button>
      <button on:click={() => handleClick('B')}>B Wins</button>
    </div>
  {/if}

  <h3>Score History</h3>
  <ul>
    {#each $gameState.scores as {hole, winner, score}}
      <li>
        Hole {hole}: {winner} - Score: {formatScore(score)}
      </li>
    {/each}
  </ul>
</main>

<style>
  main {
    font-family: Arial, sans-serif;
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
  }

  button {
    margin: 5px;
    padding: 10px 20px;
    font-size: 16px;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    margin-bottom: 10px;
  }
</style>