<script>
	import { page } from '$app/stores';
	import { supabase } from '$lib/supabaseClient';
	import { get, readable } from 'svelte/store';
	import Auth from '../components/auth/index.svelte';
	import Post from '../components/Post/index.svelte';
	import Quill from '../components/quill/index.svelte';

	import { flip } from 'svelte/animate';
	import { quintOut } from 'svelte/easing';
	import { crossfade } from 'svelte/transition';

	const [send, receive] = crossfade({
		duration: (d) => Math.sqrt(d * 100),

		fallback(node, params) {
			const style = getComputedStyle(node);
			const transform = style.transform === 'none' ? '' : style.transform;

			return {
				duration: 600,
				easing: quintOut,
				css: (t) => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`
			};
		}
	});

	export let id;

	const posts = readable(null, (set) => {
		supabase
			.from('comments')
			.select('*', { count: 'exact' })
			.limit(10)
			.then(({ error, data }) => set(data));

		const subscription = supabase
			.channel('any')
			.on(
				'postgres_changes',
				{ event: 'INSERT', schema: 'public', table: 'comments' },
				(payload) => {
					console.log('Change received!', payload.new);
					set([payload.new, ...get(posts)].slice(0, 10));
				}
			)
			.subscribe();

		console.log(subscription);

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

<h1 class="text-3xl">Devito Chat</h1>

<svelte:head>
	<title>Supabase + SvelteKit</title>
	<meta name="description" content="SvelteKit using supabase-js v2" />
	<link href="//cdn.quilljs.com/1.3.6/quill.snow.css" rel="stylesheet" />
</svelte:head>

{#if !$page.data.session}
	<Auth />
{:else}
	<!-- <Account session={$page.data.session} /> -->
	<Quill />
	<br />
	{#if $posts}
		<div class="gap-2 flex flex-col">
			{#each $posts as comment (comment.id)}
				<span in:receive={{ key: comment.id }} out:send={{ key: comment.id }} animate:flip>
					<Post>{getContent(comment.content)}</Post>
				</span>
			{/each}
		</div>
	{:else}
		<div>No comments yet</div>
	{/if}
{/if}
