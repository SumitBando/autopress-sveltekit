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

  // Function to calculate final result
  function calculateFinalResult(finalScore) {
    if (!finalScore) return { winner: 'Tie', difference: 0 };

    let aMatches = 0;
    let bMatches = 0;

    for (let i = 0; i < finalScore.length; i++) {
      if (i % 2 === 0) {
        aMatches += finalScore[i];
      } else {
        bMatches += finalScore[i];
      }
    }

    const difference = Math.abs(aMatches - bMatches);
    const winner = aMatches > bMatches ? 'A' : (bMatches > aMatches ? 'B' : 'Tie');

    return { winner, difference };
  }
</script>

<main>
  <h1>Autopush Golf Scoring</h1>

  {#if $gameState.gameOver}
    <h2>Game Over</h2>
    {#if $gameState.scores.length > 0}
      {@const finalScore = $gameState.scores[$gameState.scores.length - 1].score}
      {@const { winner, difference } = calculateFinalResult(finalScore)}
      <p>Final Score: {formatScore(finalScore)}</p>
      {#if winner === 'Tie'}
        <p>The game ended in a tie!</p>
      {:else}
        <p>Team {winner} won by {difference} {difference === 1 ? 'match' : 'matches'}!</p>
      {/if}
    {:else}
      <p>No scores recorded.</p>
    {/if}
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