<script lang="ts">
	import { supabase } from '$lib/supabaseClient';
	import Quill from 'quill';
	import { onMount } from 'svelte';

	let editor;
	let quill = null;

	onMount(() => {
		quill = new Quill(editor, {
			modules: {
				toolbar: [
					[{ header: [1, 2, 3, false] }],
					['bold', 'italic', 'underline', 'strike'],
					['link', 'code-block']
				]
			},
			placeholder: 'Type something...',
			theme: 'snow' // or 'bubble'
		});
	});

	const sendData = async () => {
		let quillData = quill.getContents();

		try {
			const { data, error } = await supabase
				.from('comments')
				.insert([{ content: JSON.stringify(quillData) }]);
		} catch (error) {
			console.error(error);
		}
	};
</script>

<svelte:head>
	<link href="//cdn.quilljs.com/1.3.6/quill.snow.css" rel="stylesheet" />
</svelte:head>

<div id="editor" bind:this={editor} />
<button on:click={sendData}>Send</button>
