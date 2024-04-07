<script>
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';

	let words = '';
	let wordList = [];
	let defList = [];
	const check = async () => {
		let newList = [];
		const list = await Promise.all(
			words.split('\n').map(async (word) => {
				let res = await fetch(`https://api.api-ninjas.com/v1/thesaurus?word=${word}`, {
					headers: {
						'X-Api-Key': 'HbOUeqXSM5JLJovbPqRz4Q==yMlBTFYhfm5eLqQc'
					}
				});

				return res.json();
			})
		);
		defList = await Promise.all(
			words.split('\n').map(async (word) => {
				let res = await fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${word}`, {});

				return res.json();
			})
		);

		console.log(list);
		wordList = list;
	};
	onMount(async () => {
		defList = localStorage.getItem('defList') ? JSON.parse(localStorage.getItem('defList')) : [];
		wordList = localStorage.getItem('wordList') ? JSON.parse(localStorage.getItem('wordList')) : [];
		words = localStorage.getItem('words') ? localStorage.getItem('words') : '';
	});
	$: {
		if (browser && wordList.length && defList.length) {
			localStorage.setItem('defList', JSON.stringify(defList));
			localStorage.setItem('wordList', JSON.stringify(wordList));
			localStorage.setItem('words', words);
		}
	}
</script>

<div class="container">
	<header>
		<h1 class="title">Definition app</h1>
	</header>
	<main>
		<textarea class="textarea" bind:value={words} />
		<button class="button is-primary" on:click={check}>Get words</button>
		<table class="table">
			<thead
				><tr
					><th>Word</th><th>Synonym</th><th>Synonym</th><th>Antonym</th><th>Antonym</th>
					<th>Part of Speech</th> <th>Definition</th> <th>Sentence</th></tr
				></thead
			>
			{#each wordList as word, idx}
				<tr
					><td><a target="_blank" href={`https://thesaurus.com/browse/${word.word}`}>{word.word}</a></td>
					<td><input class="input" type="text" bind:value={word.synonyms[0]} /></td>
					<td><input class="input" type="text" bind:value={word.synonyms[1]} /></td>
					<td><input class="input" type="text" bind:value={word.antonyms[0]} /></td>
					<td><input class="input" type="text" bind:value={word.antonyms[1]} /></td>
					<td
						><input
							class="input"
							type="text"
							bind:value={defList[idx][0].meanings[0].partOfSpeech}
						/>
					</td>
					<td
						><textarea
							class="textarea"
							style="max-width: 400px"
							bind:value={defList[idx][0].meanings[0].definitions[0].definition}
						/>
					</td>
					<td
						><textarea class="textarea" style="min-width: 400px" bind:value={word.sentence} />
					</td>
				</tr>
			{/each}
		</table>
	</main>
</div>

<style>
	/*table {
		overflow: scroll;
		max-width: calc(100vw - 2rem);
	}*/
</style>
