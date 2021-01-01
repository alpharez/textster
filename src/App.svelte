<script>
	import "./words.js";
	import { createEventDispatcher, onMount } from "svelte";

	let TICK_DELAY = 1000;
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
		if (!word.includes(letters[buttonId])) {
			guesses_left -= 1;
		}
		if (allGuessed()) {
			alert("YOU WON");
			reset();
		}
		score += 1;
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
			if (guessedLetters.includes(word[i])) {
				grid[wordposition[0]][wordposition[1] + i][1] = word[i];
			} else {
				grid[wordposition[0]][wordposition[1] + i][1] = "";
			}
		}
	}

	const fn = (n) => {
		setTimeout(() => {
			if (wordposition[0] < 19) {
				wordposition[0] += 1;
				score += 1;
			} else {
				lost = true;
				return;
			}
			if (guesses_left < 1) {
				lost = true;
				return;
			}

			fn(TICK_DELAY);
		}, n);
	};
	onMount(() => {
		fn(TICK_DELAY);
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
		//gridwithword = grid;
		word = wordlist[getRandomInt(wordlist.length)].toUpperCase();
		wordposition[1] = getRandomInt(GRID_SIZE - word.length);
		fn(TICK_DELAY);
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
		font-weight: 100;
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
	.letters {
		display: flex;
		justify-content: left;
		align-items: center;
	}
	.square {
		width: 30px;
		height: 30px;
		border: solid 1px #fff;
	}
	.empty {
		color: green;
		background-color: white;
		font-size: 1.75em;
	}
	.notguessed {
		color: black;
		background-color: lightblue;
	}
	.letter {
		color: white;
		background-color: green;
		font-size: 1.75em;
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
	{#if lost}
		<h1 class="tcenter">you lost</h1>
		<h3 class="tcenter"><button on:click={restart}>RESTART</button></h3>
	{/if}
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
						class={guessed[i]}>{letters[i]}</button>
				{/each}
			</div>
		</div>
	</div>
	{#if lost}
		<div class="center restart">
			<button on:click={restart}> Start again </button>
		</div>
	{/if}
</main>
