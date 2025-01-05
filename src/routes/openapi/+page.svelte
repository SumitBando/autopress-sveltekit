<script lang="ts">
  // Define the possible teams
  type Team = 'A' | 'B';

  // One match = { value: number, side?: Team }
  // if side is undefined, the match is neutral.
  interface Match {
    value: number;
    side?: Team;
  }

  // For recording each hole's result:
  interface HoleResult {
    holeNumber: number;
    result: 'A' | 'B' | 'Halved';
    scoreboard: string; // e.g. "1a 1b 1a"
  }

  // Our scoreboard is a reactive array of matches
  let scoreboard: Match[] = [];

  // Hole history (most recent first)
  let holeHistory: HoleResult[] = [];

  // Current hole number (1 to 18)
  let currentHole = 1;

  // Track if game is over (>= 18 holes played)
  let gameOver = false;

  /**
   * Convert scoreboard matches to a readable string, e.g. "1a 1b 1a"
   */
  function scoreboardToString(sb: Match[]): string {
    return sb.map(matchToString).join(' ');
  }

  /**
   * A small utility to display match data
   *  - e.g. {value: 2, side: 'A'} => "2a"
   *  - e.g. {value: 0} => "0"
   *  - e.g. {value: 1, side: 'B'} => "1b"
   */
  function matchToString(match: Match): string {
    const { value, side } = match;
    if (value === 0 && !side) {
      return '0';
    }
    if (value === 0 && side) {
      return `0${side}`;
    }
    return `${value}${side ?? ''}`;
  }

  /**
   * Main "hole played" function: handle a hole’s result:
   *  - "A" => Team A wins
   *  - "B" => Team B wins
   *  - "Halved" => no change in scoreboard
   */
  function recordHole(result: 'A' | 'B' | 'Halved') {
    if (gameOver) return;

    switch (result) {
      case 'A':
        handleHoleWin('A');
        break;
      case 'B':
        handleHoleWin('B');
        break;
      case 'Halved':
        // do nothing
        break;
    }

    // Record the final scoreboard for this hole
    const newHoleResult: HoleResult = {
      holeNumber: currentHole,
      result,
      scoreboard: scoreboardToString(scoreboard)
    };

    // Unshift so that the most recent hole is at the top
    holeHistory = [...holeHistory, newHoleResult];
    console.log('scoreboard: ', scoreboard, 'holeHistory: ', holeHistory);

    // Move on to the next hole
    currentHole++;

    // End game after hole #18
    if (currentHole > 18) {
      gameOver = true;
    }
  }

  /**
   * Handle a hole win by a given team (A or B).
   */
  function handleHoleWin(winner: Team) {
    const loser = winner === 'A' ? 'B' : 'A';

    // If no matches exist yet, open three matches:
    //  1 in favor of winner, 1 in favor of loser, 1 in favor of winner
    if (scoreboard.length === 0) {
      scoreboard = [
        { value: 1, side: winner },
        { value: 1, side: loser },
        { value: 1, side: winner }
      ];
    } else {
      // Update each existing match
      scoreboard = scoreboard.map((m) => {
        if (m.side === winner) {
          // If it's already in the winner's favor, increment
          return { value: m.value + 1, side: winner };
        } else if (m.side === loser) {
          // If it’s in the loser’s favor, decrement
          const newVal = m.value - 1;
          if (newVal === 0) {
            // If it goes to zero, match becomes neutral
            return { value: 0 };
          } else {
            // Remains in the loser's favor, just new value
            return { value: newVal, side: loser };
          }
        } else {
          // Neutral match => it becomes 1 in the winner's favor
          return { value: 1, side: winner };
        }
      });
    }

    // Auto press: if the right-most match just hit 2, open a new neutral match at 0
    const lastMatch = scoreboard[scoreboard.length - 1];
    if (lastMatch?.value === 2) {
      scoreboard.push({ value: 0 }); // neutral new match
    }
  }
</script>

<h1>Auto Press Golf Scoring (18 Holes)</h1>

<!-- Action buttons -->
<section>
  <button on:click={() => recordHole('A')} disabled={gameOver}>Team A Wins Hole</button>
  <button on:click={() => recordHole('B')} disabled={gameOver}>Team B Wins Hole</button>
  <button on:click={() => recordHole('Halved')} disabled={gameOver}>Hole Halved</button>
</section>

<!-- Current/Final scoreboard -->
<h2>{gameOver ? 'Final' : 'Current'} Scoreboard</h2>
<p style="font-weight: bold; font-size: 1.2em;">
  {#if scoreboard.length === 0}
    No score yet
  {:else}
  <strong>Hole {currentHole-1}</strong> - {scoreboard.map((m) => matchToString(m)).join(' ')}
  {/if}
</p>

<h2>Hole-by-Hole History</h2>
{#key holeHistory}
<ul>
  {#each [...holeHistory].filter(h => h.holeNumber < currentHole - 1).reverse() as hole}
    <li>
      <strong>Hole {hole.holeNumber}</strong> - 
      {hole.result === 'Halved' ? 'Halved' : `Team ${hole.result} won`}  
      &nbsp;| Score: {hole.scoreboard}
    </li>
  {/each}
</ul>
{/key}
