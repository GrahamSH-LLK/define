<script>
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';

	let words = '';
	let wordList = [];
	let defList = [];
	let tableEl;
	const check = async () => {
		let newList = [];
		const list = await Promise.all(
			words.split('\n').map(async (word) => {
				try {
					let res = await fetch(`https://thesaurus.grahamsh.workers.dev/${word}`, {
						headers: {
							'X-Api-Key': 'HbOUeqXSM5JLJovbPqRz4Q==yMlBTFYhfm5eLqQc'
						}
					});

					return res.json();
				} catch {}
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
	let copying = false;
	const wait = (x) => { return new Promise((resolve) => {
		setTimeout(()=> {resolve(true)}, x)
	})};
	const copy = async() => {
		copying = true;
		await wait(0);
		const spreadSheetRow = new Blob([tableEl.outerHTML], { type: 'text/html' });
		navigator.clipboard.write([new ClipboardItem({ 'text/html': spreadSheetRow })]);
		copying = false;
	};
</script>

<div class="container">
	<header class="mb-2">
		<h1 class="title has-text-primary is-2">Definitions 📙</h1>
	</header>

	<main>
		<div class="box">
			<h3 class="title is-5">Add your words</h3>
			<div class="field">
				<textarea class="textarea" placeholder="Words, newline separated" bind:value={words} />
			</div>
			<div class="field">
				<button class="button is-primary" on:click={check}>Get definitions</button>
			</div>
			<div class="field">
				<button class="button is-primary" on:click={copy}>Copy</button>
			</div>
		</div>
		<table class="table" bind:this={tableEl}>
			<thead
				><tr
					><th>Word</th><th>Synonym</th><th>Synonym</th><th>Antonym</th><th>Antonym</th>
					<th>Part of Speech</th> <th>Definition</th> <th>Sentence</th></tr
				></thead
			>
			{#each wordList as word, idx}
				<tr
					><td
						><a target="_blank" href={`https://thesaurus.com/browse/${word.word}`}>{word.word}</a
						></td
					>
					<td
						><span class="copy-only">{word.synonyms[0].targetWord}</span><input
							class="input"
							type="text"
							bind:value={word.synonyms[0].targetWord}
						/></td
					>
					<td
						><span class="copy-only">{word.synonyms[1].targetWord}</span><input
							class="input"
							type="text"
							bind:value={word.synonyms[1].targetWord}
						/></td
					>
					<td
						><span class="copy-only">{word.antonyms[0].targetWord}</span><input
							class="input"
							type="text"
							bind:value={word.antonyms[0].targetWord}
						/></td
					>
					<td
						><span class="copy-only">{word.antonyms[1].targetWord}</span><input
							class="input"
							type="text"
							bind:value={word.antonyms[1].targetWord}
						/></td
					>
					<td>
						{#if !copying}
						<div class="select">
							<select bind:value={defList[idx][0].currPOS}>
								{#each defList[idx][0].meanings as meaning, i}
									<option value={i}>{meaning.partOfSpeech}</option>
								{/each}
							</select>
						</div>
						{/if}
						<span class="copy-only"
							>{defList[idx][0].meanings[defList[idx][0].currPOS || 0].partOfSpeech}</span
						>
					</td>
					<td
						><span class="copy-only"
							>{defList[idx][0].meanings[defList[idx][0].currPOS || 0].definitions[0]
								.definition}</span
						><textarea
							class="textarea"
							style="max-width: 400px; min-width: 250px;"
							bind:value={defList[idx][0].meanings[defList[idx][0].currPOS || 0].definitions[0]
								.definition}
						/>
					</td>
					<td
						><span class="copy-only">{word.sentence ? word.sentence : ''}</span><textarea
							class="textarea"
							style="min-width: 250px"
							bind:value={word.sentence}
						/>
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
	@media print {
		.box,
		header {
			display: none;
		}
		input {
			display: none;
		}
		textarea {
			display: none;
		}
	}

	@media not print {
		.copy-only {
			position: absolute;
			width: 1px;
			height: 1px;
			padding: 0;
			margin: -1px;
			overflow: hidden;
			clip: rect(0, 0, 0, 0);
			white-space: nowrap;
			border-width: 0;
		}
	}
</style>
