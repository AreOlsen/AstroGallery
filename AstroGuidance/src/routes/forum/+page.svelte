<script lang="ts">
	import Post from "$lib/Post.svelte";
	import { dbFireStore } from "../../firebase";
	import { collectionStore } from "$lib/docCollectionStore.ts";
	let posts = collectionStore(dbFireStore, "posts");
</script>

<main class="flex flex-col gap-4 text-center">
	<div class="border-b-accent-focus border-b-[1px] py-6">
		<h2 class="text-6xl font-bold">Forum</h2>
	</div>
	<!-- IF POSTS EXIST-->
	{#if $posts}
		<div
			class="m-6 min-h-full grid gap-6 auto-rows-[35rem]"
			style="grid-template-columns: repeat(auto-fill, 23rem);"
		>
			<!--SHOW EACH POST-->
			{#each $posts as post (post.id)}
				<Post onProfile={false} postID={post.id} />
			{/each}
		</div>
	{:else}
		<!--IF NO POSTS-->
		<div class="h-full flex justify-center items-center">
			<h2 class="text-4xl font-bold">Loading posts...</h2>
		</div>
	{/if}
</main>
