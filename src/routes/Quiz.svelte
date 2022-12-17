<script>
	import { fly } from 'svelte/transition';
	import { onMount, beforeUpdate, afterUpdate, onDestroy } from 'svelte';
	import Question from './Question.svelte';
	import Modal from './Modal.svelte';
	import { score } from '../store.js';
	let activeQuestion = 0;
	let quiz = getQuiz();
	let isModalOpen = false;

	// Doesn't work on server side
	onMount(() => {
		console.log('Mounted');
	});

	beforeUpdate(() => {
		console.log('Before update');
	});

	afterUpdate(() => {
		console.log('After update');
	});

	onDestroy(() => {
		console.log('Destoyed');
	});

	async function getQuiz() {
		const res = await fetch('https://opentdb.com/api.php?amount=10&category=12&type=multiple');
		quiz = await res.json();

		return quiz;
	}

	function resetQuiz() {
		isModalOpen = false;
		activeQuestion = 0;
		score.set(0);
		quiz = getQuiz();
	}

	function nextQuestion() {
		++activeQuestion;
	}

	// Reactive Statement
	$: if ($score > 5) {
		isModalOpen = true;
	}

	// Reactive Declaration
	$: questionNumber = activeQuestion + 1;
</script>

<div>
	<button on:click|once={resetQuiz}>Start new quiz</button>
	<h3>My score: {$score}</h3>
	<h4>Question #{questionNumber}</h4>

	<div class="container">
		{#await quiz}
			loading...
		{:then data}
			{#each data.results as question, index}
				{#if index === activeQuestion}
					<div in:fly={{ x: 100 }} out:fly={{ x: -200 }} class="fade-wrapper">
						<Question {nextQuestion} {question} />
					</div>
				{/if}
			{/each}
		{/await}
	</div>
</div>

{#if isModalOpen}
	<Modal on:close={resetQuiz}>
		<h2>You won!</h2>
		<p>Congratz!</p>
		<button on:click={resetQuiz}>Start over</button>
	</Modal>
{/if}

<style>
	.fade-wrapper {
		position: absolute;
	}

	.container {
		min-height: 500px;
	}
</style>
