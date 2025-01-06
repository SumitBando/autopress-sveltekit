<script>
  import { onMount } from 'svelte';

  let score = [1, 1, 1];
  let holesPlayed = [];
  let gameOver = false;

  function handleButtonClick(winner) {
    if (gameOver) return;

    // Update the score based on the winner
    if (winner === 'A') {
      score = score.map((match, index) => {
        if (match > 0) return match + 1;
        return 1;
      });
    } else if (winner === 'B') {
      score = score.map((match, index) => {
        if (match > 0) return match - 1;
        return -1;
      });
    }

    // Open a new match if the last match is 2
    if (score[score.length - 1] === 2) {
      score.push(0);
    }

    // Record the hole result
    holesPlayed.push({ winner, score: [...score] });

    // Check if the game is over
    if (holesPlayed.length === 18) {
      gameOver = true;
    }
  }

  function calculateMatchDifference() {
    let aMatches = 0;
    let bMatches = 0;
    for (const hole of holesPlayed) {
      for (const match of hole.score) {
        if (match > 0) aMatches += match;
        if (match < 0) bMatches += Math.abs(match);
      }
    }
    return aMatches - bMatches;
  }
</script>

<main>
  <h1>Bahama Auto Press Golf Scoring</h1>

  {#if !gameOver}
    <div>
      {#each score as match, index}
        <button on:click={() => handleButtonClick(index % 2 === 0 ? 'A' : 'B')}>
          {match > 0 ? `A +${match}` : match < 0 ? `B +${Math.abs(match)}` : 'Halved'}
        </button>
      {/each}
    </div>
  {/if}

  {#each holesPlayed as hole}
    <div>
      <span>{hole.winner}</span>
      <span>{hole.score.join(' ')}</span>
    </div>
  {/each}

  {#if gameOver}
    <h2>Game Over</h2>
    <p>Winner: {calculateMatchDifference() > 0 ? 'A' : 'B'}</p>
    <p>Match Difference: {Math.abs(calculateMatchDifference())}</p>
  {/if}
</main>