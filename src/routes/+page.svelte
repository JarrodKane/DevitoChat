<script>
	import { supabase } from '$lib/supabaseClient';
	import { get, readable } from 'svelte/store';
	import Quill from '../components/quill/index.svelte';

	const posts = readable(null, (set) => {
		supabase
			.from('comments')
			.select('*')
			.then(({ error, data }) => set(data));

		const subscription = supabase
			.channel('any')
			.on(
				'postgres_changes',
				{ event: 'INSERT', schema: 'public', table: 'comments' },
				(payload) => {
					console.log('Change received!', payload.new);
					set([...get(posts), payload.new]);
				}
			)
			.subscribe();

		return () => supabase.removeAllChannels();
	});

	const getContent = (comment) => {
		// TODO: Need to break this down to sending each line not just the first line received
		if (typeof comment === 'string') {
			return JSON.parse(comment).ops[0].insert;
		}
		return comment.ops[0].insert;
	};
</script>

<h1>Devito Chat</h1>

<svelte:head>
	<title>Supabase + SvelteKit</title>
	<meta name="description" content="SvelteKit using supabase-js v2" />
	<link href="//cdn.quilljs.com/1.3.6/quill.snow.css" rel="stylesheet" />
</svelte:head>

<!-- {#if !$page.data.session}
	<Auth />
{:else} -->
<!-- <Account session={$page.data.session} /> -->
<Quill />
<br />
{#if $posts}
	{#each $posts as comment (comment.id)}
		<div>{getContent(comment.content)}</div>
	{/each}
{:else}
	<div>No comments yet</div>
{/if}
<!-- {/if} -->
