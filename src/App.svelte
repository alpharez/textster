<script>
	import "./words.js";
	import { createEventDispatcher, onMount } from "svelte";

	let START_TICK_DELAY = 1000;
	const dispatch = createEventDispatcher();
	export let name;
	let GRID_SIZE = 20;
	let score = 0;
	let lost = false;
	let won = false;
	let guesses_left = 10;
	//let m = { x: 0, y: 0 };
	//type Cell = "guessed" | "unguessed";
	let letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
	let guessedLetters = [];
	// get word from api
	let wordlist = [
		"pneumatic",
		"success",
		"abruptly",
		"askew",
		"fixable",
		"happy",
		"possibly",
		"soap",
		"january",
		"cookies",
		"donkey",
		"artist",
	];

	function getRandomInt(max) {
		return Math.floor(Math.random() * Math.floor(max));
	}

	let word = wordlist[getRandomInt(wordlist.length)].toUpperCase();
	let wordposition = [0, 5];
	let guessed = [...Array(26)].map(() => "notguessed");

	let grid = [...Array(GRID_SIZE)].map(
		() => [...Array(GRID_SIZE)].map(() => ["empty", ""]) // [cellclass, letter]
	);
	//let gridwithword = grid;

	wordposition[1] = getRandomInt(GRID_SIZE - word.length);

	function allGuessed() {
		for (let i = 0; i < word.length; i++) {
			if (!guessedLetters.includes(word[i])) {
				return false;
			}
		}
		won = true;
		return won;
	}

	function guessedThis(event) {
		let buttonId = event.target.id;

		guessed[buttonId] = "guessed";
		guessedLetters.push(letters[buttonId]);
		// if this guess is wrong decrement guesses left
		if (word.includes(letters[buttonId])) {
			if (wordposition[0] > 2) {
				wordposition[0] -= 2;
			}
		} else {
			guesses_left -= 1;
		}
		if (allGuessed()) {
			//alert("YOU WON");
			won = true;
			score += 135 * word.length;
			reset();
		}
	}

	$: {
		for (let i = 0; i < grid.length; i++) {
			for (let k = 0; k < grid.length; k++) {
				if (grid[i][k][0] === "letter") {
					grid[i][k][0] = "empty";
					grid[i][k][1] = "";
				}
			}
		}
		for (let i = 0; i < word.length; i++) {
			grid[wordposition[0]][wordposition[1] + i][0] = "letter";
			if (guessedLetters.includes(word[i]) || lost) {
				grid[wordposition[0]][wordposition[1] + i][1] = word[i];
			} else {
				grid[wordposition[0]][wordposition[1] + i][1] = "";
			}
		}
	}

	const fn = (n) => {
		setTimeout(() => {
			if (wordposition[0] < 19 && !won && !lost) {
				wordposition[0] += 1;
			} else {
				lost = true;
				//return;
			}
			if (guesses_left < 1) {
				lost = true;
				//return;
			}

			fn(START_TICK_DELAY);
		}, n);
	};
	onMount(() => {
		fn(START_TICK_DELAY);
	});

	function reset() {
		wordposition = [0, 5];
		guessed = [...Array(26)].map(() => "notguessed");
		guesses_left = 10;
		guessedLetters = [];
		lost = false;
		won = false;
		grid = [...Array(GRID_SIZE)].map(
			() => [...Array(GRID_SIZE)].map(() => ["empty", ""]) // [cellclass, letter]
		);
		//gridwithword = grid;
		word = wordlist[getRandomInt(wordlist.length)].toUpperCase();
		wordposition[1] = getRandomInt(GRID_SIZE - word.length);
	}

	function restart() {
		wordposition = [0, 5];
		guessed = [...Array(26)].map(() => "notguessed");
		guessedLetters = [];
		guesses_left = 10;
		lost = false;
		won = false;
		grid = [...Array(GRID_SIZE)].map(
			() => [...Array(GRID_SIZE)].map(() => ["empty", ""]) // [cellclass, letter]
		);
		score = 0;
		word = wordlist[getRandomInt(wordlist.length)].toUpperCase();
		wordposition[1] = getRandomInt(GRID_SIZE - word.length);
		//fn(TICK_DELAY);
	}
</script>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 1.5em;
		font-weight: bold;
	}

	h3 {
		color: #333;
		text-transform: uppercase;
		font-size: 1.25em;
		font-weight: bold;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}

	.center {
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.tcenter {
		text-align: center;
	}
	.vcenter {
		vertical-align: middle;
	}
	.letters {
		display: flex;
		justify-content: left;
		align-items: center;
	}
	.square {
		width: 20px;
		height: 20px;
		border: solid 1px #fff;
	}
	.empty {
		color: green;
		background-color: white;
		font-size: 1em;
		border: solid 1px #ccc;
	}
	.notguessed {
		color: black;
		background-color: lightblue;
	}
	.letter {
		color: white;
		background-color: green;
		font-size: 1em;
	}
	.guessed {
		color: white;
		background-color: red;
	}
	.row {
		display: flex;
	}
</style>

<main>
	<h1>{name}!</h1>
	<h3>Guesses left : {guesses_left} Score : {score}</h3>
	<div>
		<div class="center">
			<div>
				{#each grid as row, i}
					<div class="row">
						{#each row as cell, k}
							<div class={`square ${cell[0]}`}>{cell[1]}</div>
						{/each}
					</div>
				{/each}
			</div>
		</div>
		<div class="center">
			<div>
				{#each letters as row, i}
					<button
						id={i}
						on:click={guessedThis}
						disabled={guessed[i] === 'guessed'}
						class={guessed[i]}>{letters[i]}</button>
				{/each}
			</div>
		</div>
	</div>
	{#if lost}
		<div class="vcenter">
			<h3>Game Over</h3>
		</div>
		<div class="vcenter">
			<button on:click={restart}> Start again </button>
		</div>
	{/if}
</main>
