<script lang="ts" context="module">
  export function newInitState() {
    return {score: {wounds: 0, media: 0, trail: 0}, events: []}
  }

  export const eventsPerType = [
    /*endless corridors*/ [
      {description: 'You wander into work event. You leave holding a beer.', effect: new Map([['trail', 1]])},
      {description: 'You get stuck in a ministerial revolving door.', effect: new Map([['media', 1]])},
      {description: "It's the chancellor, holding a bag marked SWAG", effect: new Map([['trail', 1]])},
      {description: "You briefly get locked in combat with a cabinet", effect: new Map([['wounds', 1]])},
      {description: 'You detect the smell of fear and silver spoons', effect: new Map([['trail', 2]])},
      {description: 'You stop to cure a passerby of scrofula', effect: new Map([['media', 1]])},
    ],
    /*battle rages on*/ [
      {
        description: "You summon the royal lion. It's not as friendly as you remember",
        effect: new Map([['wounds', 1]])
      },
      {description: "You battle through a room full of red tape", effect: new Map([['wounds', 1], ['trail', 1]])},
      {description: "You are ambushed with a cake. Ow.", effect: new Map([['wounds', 1], ['media', 1]])},
      {description: "You are confronted by a minister. You end them.", effect: new Map([['wounds', 1]])},
      {
        description: "A secretary of state catches you with a lucky blow",
        effect: new Map([['wounds', 1], ['trail', 1]])
      },
      {description: "You choke someone to death with the Magna Carta", effect: new Map([['wounds', 1], ['trail', 1]])},
    ],
    /*affairs of state*/[
      {
        description: "One of your relatives has caused an international scandal",
        effect: new Map([['media', 1], ['trail', 1]])
      },
      {
        description: "One of your relatives has caused an international scandal",
        effect: new Map([['media', 1], ['trail', 1]])
      },
      {description: "You lose your crown in the viscera and have to find it", effect: new Map([['media', 1]])},
      {description: "Your dogs pick up a smell. The smell of vengeance.", effect: new Map([['trail', 2]])},
      {
        description: "You have to stop to celebrate another jubilee. It's quite time consuming.",
        effect: new Map([['media', 1]])
      },
      {
        description: "You have to stop to celebrate another jubilee. It's quite time consuming.",
        effect: new Map([['media', 1]])
      },
    ],
  ]

  let brexitRolls = 0

  export function generateNewEvent() {
    const eventType = Math.floor(Math.random() * 2)
    const event = Math.floor(Math.random() * 6)
    if (event === 5) {
      if (brexitRolls > 1) {
        console.log('sudden brexit!')
        return {
          description: 'You accidentally leave the European Union, and drown in the ocean.',
          effect: new Map([['brexit', true]])
        }
      } else {
        brexitRolls++
      }
    } else {
      brexitRolls = 0
    }
    return eventsPerType[eventType][event]
  }

  export function pushNewEvent(state, event) {
    state.events.push(event)
    return state
  }
</script>
<script lang="ts">
  let state = null
  $: {
    console.log({state})
  }
  $: {
    if (state !== null) {
      const aggregatedScoreEffects = state.events.reduce((accu, {effect}) => {
        if (accu.get('brexit')) {
          return accu
        }
        if (effect.get('brexit')) {
          accu.set('brexit', true)
        } else {
          [...effect].forEach(([scoreType, increment]) => {
            accu.set(scoreType, (accu.get(scoreType) ?? 0) + increment);
          });
        }
        return accu
      }, new Map())
      if (aggregatedScoreEffects.get('brexit')) {
        state.brexit = true
        state.gameOver = true
      }
      state.score = {
        wounds: (aggregatedScoreEffects.get('wounds') ?? 0),
        media: (aggregatedScoreEffects.get('media') ?? 0),
        trail: (aggregatedScoreEffects.get('trail') ?? 0),
      }
      if (Object.values(state.score).some(val => val >= 10)) {
        state.gameOver = true
      }
    }
  }
</script>

{#if state === null}
  <h2>you are the queen of england</h2>
  <h3>and the prime minister won't resign</h3>
  <p>which leaves you no choice but to perform your duty</p>
  <p>don your mask and draw excalibur one last time</p>
  <button on:click={() => state = newInitState()}>Start New Game</button>
{:else}
  <table>
    <caption>Score</caption>
    <tr>
      <th>wounds</th>
      {#each new Array(10) as _value, index}
        <td class:filled={state.score.wounds >= index + 1}></td>
      {/each}
    </tr>
    <tr>
      <th>media</th>
      {#each new Array(10) as _value, index}
        <td class:filled={state.score.media >= index + 1}></td>
      {/each}
    </tr>
    <tr>
      <th>the trail</th>
      {#each new Array(10) as _value, index}
        <td class:filled={state.score.trail >= index + 1}></td>
      {/each}
    </tr>
  </table>

  <section>
    {#if state.events.length > 0}
      <h2 class="banner">Latest Event</h2>
      {#if state?.brexit}
        <h3>Sudden Brexit!</h3>
      {/if}
      {state.events[state.events.length - 1].description}
      {#if !state?.brexit}
        <ul>
          {#each [...state.events[state.events.length - 1].effect] as [scoreType, increment]}
            <li>+{increment} <strong>{scoreType}</strong></li>
          {/each}
        </ul>
      {/if}
    {/if}
    {#if state?.gameOver}
      <h2 class="banner">
        {#if state.score.trail >= 10 && state.score.wounds < 10 && state.score.media < 10 && !state?.brexit}Success!
        {:else}Game Over
        {/if}
      </h2>
      {#if state.score.wounds >= 10}
        You are defeated and your rampage comes to an end. Your funeral is suitably expensive, an a colossus is raised
        over your tomb.
      {:else if state.score.media >= 10}
        Your identity is uncovered and the cameras force you back to the palace to nurse your injured pride.
      {:else if state.score.trail >= 10}
        You finally catch up to the prime minister and bring an end to this farce. You stand on a pile of your defeated
        enemies and howl at the uncaring gods above.
      {/if}
      <button on:click={() => state = null}>Back to Title Screen</button>
      {#if !state?.viewStory}
        <button on:click={() => state = {...state, viewStory: true}}>view story</button>
      {:else }
        <table>
          <caption>Events Encountered</caption>
          <tr>
            <th>Description</th>
            <th>Effect</th>
          </tr>
          {#each state.events as event, i (i)}
            <tr>
              <td>{event.description}</td>
              <td>
                {[...event.effect].map(([scoreType, increment]) => scoreType === 'brexit' ? 'sudden brexit' : `+${increment} ${scoreType}`).join(', ')}
              </td>
            </tr>
          {/each}
        </table>
      {/if}
    {:else }
      <button on:click={() =>{  state = pushNewEvent(state, generateNewEvent()); }}>next event</button>
    {/if}
  </section>
{/if}

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
    Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  }


  button {
    display: block;
    margin: 4ch 2ch
  }


  th {
    text-transform: uppercase;
  }

  td {
    min-width: 1rem;
    border: 1px solid black;
  }

  td.filled {
    background-color: black
  }
</style>
