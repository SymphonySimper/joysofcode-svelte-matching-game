<script lang="ts">
	import { emoji } from './emoji';

	type State = 'start' | 'playing' | 'paused' | 'won' | 'lost';

	let gameState: State = $state('start');
	let size = 20;
	let maxMatches = $state(0);
	let grid = $state(createGrid());
	let selected: number[] = $state([]);
	let matches: string[] = $state([]);
	let timerId: number | null = $state(null);
	let time = $state(60);

	function createGrid() {
		// only want unique cards
		let cards = new Set<string>();
		// half because we duplicate the cards
		let maxSize = size / 2;

		while (cards.size < maxSize) {
			// pick random emoji
			const randomIndex = Math.floor(Math.random() * emoji.length);
			cards.add(emoji[randomIndex]);
		}

		// duplicate and shuffle cards
		const shuffled = shuffle([...cards, ...cards]);
		maxMatches = shuffled.length / 2;

		return shuffled;
	}

	function shuffle<Items>(array: Items[]) {
		return array.sort(() => Math.random() - 0.5);
	}

	function startGameTimer() {
		function countdown() {
			gameState !== 'paused' && (time -= 1);
			time === 0 && gameLost();
		}
		timerId = setInterval(countdown, 1000);
	}

	function selectCard(cardIndex: number) {
		selected = selected.concat(cardIndex);
		if (selected.length === 2) matchCards();
	}

	function matchCards() {
		// array destructuring can have any name for the values
		const [first, second] = selected;

		if (grid[first] === grid[second]) {
			matches = matches.concat(grid[first]);
			if (maxMatches === matches.length) gameWon();
		}

		// clear selected
		setTimeout(() => (selected = []), 300);
	}

	function pauseGame(e: KeyboardEvent) {
		if (e.key === 'Escape') {
			switch (gameState) {
				case 'playing':
					gameState = 'paused';
					break;
				case 'paused':
					gameState = 'playing';
					break;
			}
		}
	}

	function resetGame() {
		timerId && clearInterval(timerId);
		grid = createGrid();
		selected = [];
		matches = [];
		timerId = null;
		time = 60;
	}

	function gameStart() {
		gameState = 'playing';
		//	in case you pause the game
		!timerId && startGameTimer();
	}

	function gameWon() {
		gameState = 'won';
		resetGame();
	}

	function gameLost() {
		gameState = 'lost';
		resetGame();
	}
</script>

<svelte:window onkeydown={pauseGame} />

{#if gameState === 'start'}
	<h1>Matching game</h1>
	<button onclick={gameStart}>Play</button>
{/if}

{#if gameState === 'paused'}
	<h1>Game paused</h1>
{/if}

{#if gameState === 'playing'}
	<h1 class="timer" class:pulse={time <= 10}>
		{time}
	</h1>

	<div class="matches">
		{#each matches as card}
			<div>{card}</div>
		{/each}
	</div>

	<div class="cards">
		{#each grid as card, cardIndex}
			{@const isSelected = selected.includes(cardIndex)}
			{@const isSelectedOrMatch = selected.includes(cardIndex) || matches.includes(card)}
			{@const match = matches.includes(card)}

			<button
				class="card"
				class:selected={isSelected}
				class:flip={isSelectedOrMatch}
				disabled={isSelectedOrMatch}
				onclick={() => selectCard(cardIndex)}
			>
				<div class="back" class:match>
					{card}
				</div>
			</button>
		{/each}
	</div>
{/if}

{#if gameState === 'lost'}
	<h1>You lost! 💩</h1>
	<button onclick={gameStart}>Play again</button>
{/if}

{#if gameState === 'won'}
	<h1>You win! 🎉</h1>
	<button onclick={gameStart}>Play again</button>
{/if}

<style>
	.cards {
		display: grid;
		grid-template-columns: repeat(5, 1fr);
		gap: 0.4rem;
	}

	.card {
		height: 140px;
		width: 140px;
		font-size: 4rem;
		background-color: var(--bg-2);
		transition: rotate 0.3s ease-out;
		transform-style: preserve-3d;

		&.selected {
			border: 4px solid var(--border);
		}

		&.flip {
			rotate: y 180deg;
			pointer-events: none;
		}

		& .back {
			position: absolute;
			inset: 0;
			display: grid;
			place-content: center;
			backface-visibility: hidden;
			rotate: y 180deg;
		}

		& .match {
			transition: opacity 0.3s ease-out;
			opacity: 0.4;
		}
	}

	.matches {
		display: flex;
		gap: 1rem;
		margin-block: 2rem;
		font-size: 3rem;
	}

	.timer {
		transition: color 0.3s ease;
	}

	.pulse {
		color: var(--pulse);
		animation: pulse 1s infinite ease;
	}

	@keyframes pulse {
		to {
			scale: 1.4;
		}
	}
</style>
