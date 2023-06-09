<script lang="ts">
	export let postID;
	export let onProfile;
	import { docStore } from "$lib/docCollectionStore.ts";
	import { dbFireStore, auth } from ".././firebase";
	import { userStore } from "$lib/authStore.ts";
	import { deletePost, postComment, likePost } from "$lib/postHandling.js";
	import Comment from "$lib/Comment.svelte";
	import MediaGallery from "$lib/MediaGallery.svelte";
	let user = userStore(auth);
	export let postData = docStore(dbFireStore, `posts/${postID}`);
	$: authorData = docStore(dbFireStore, `profiles/${$postData?.authorID}`);
	$: likedPost = $postData?.likesUsers.includes($user?.uid.toString());

	let postCommentData = "";
</script>

<article
	class="bg-base-300 flex flex-col items-center justify-evenly gap-2 rounded-lg shadow-2xl text-center relative p-10 py-4 pb-24"
>
	<!--TITLE OF POST-->
	<h1 class="font-bold text-4xl break-words max-w-full h-12 truncate">{$postData?.title.slice(0, 25)}</h1>
	<!--FRONT IMAGE/VIDEO OF POST-->
	{#if $postData?.elements}
		{#if $postData?.elements.length !== 0}
			{#if $postData?.elements[0].filetype == "image" && $postData?.elements[0].url.length != 0}
				<img
					src={$postData?.elements[0].url}
					alt={$postData?.title}
					class="rounded-lg border-accent shadow-xl h-[55%] w-[100%]"
				/>
			{:else if $postData?.elements[0].filetype == "video" && $postData?.elements[0].url.length != 0}
				<video
					src={$postData?.elements[0].url}
					class="rounded-lg border-accent shadow-xl h-[55%] w-[100%] bg-black"
					controls
				/>
			{/if}
		{/if}
	{/if}
	<!--DESCRIPTION-->
	<p class="break-words max-w-full h-24 truncate">{$postData?.description.slice(0, 50)}</p>
	<!--LIKE POST-->
	<button
		class="btn bg-base-300 rounded-lg flex flex-row gap-2 absolute right-5 bottom-5 {likedPost == true
			? 'bg-primary text-white'
			: ''}"
		on:click={() => {
			likePost(postID, $postData, $user);
		}}
	>
		<img src="/Icons/heart.svg" width="24px" height="24px" alt="Like icon" />
		<span class="font-bold text-xl text-white">{$postData?.totalLikes}</span>
	</button>
	<!-- The button to open post MODAL.  -->
	<label for="{postID}modal" class="btn btn-accent absolute left-5 bottom-5">More details.</label>
</article>

<!-- POST MODAL  -->
<input type="checkbox" id="{postID}modal" class="modal-toggle" />
<div class="modal">
	<div class="modal-box max-w-5xl relative p-7 pb-20 flex flex-col gap-6">
		<label for="{postID}modal" class="btn btn-sm btn-circle absolute right-2 top-2">✕</label>
		<h3 class="text-5xl text-center font-bold">{$postData?.title}</h3>
		<div class="flex flex-row gap-4 justify-evenly">
			{#if $postData?.elements}
				{#if $postData?.elements.length !== 0}
					<div class="carousel rounded-lg shadow-xl aspect-square max-h-[50vh] bg-black w-1/2">
						<MediaGallery elements={$postData?.elements} {postID} />
					</div>
				{/if}
			{/if}
			<div class="flex flex-col text-center w-1/2">
				<h3 class="text-2xl font-bold">Description:</h3>
				<section>
					<p class="text-center">{$postData?.description}</p>
				</section>
			</div>
		</div>
		<div class="grid grid-cols-{$user ? 2 : 1} grid-rows-1 gap-10 w-full">
			<div class="flex flex-col gap-2">
				<h3 class="text-2xl font-bold text-center">Comments:</h3>
				{#if $postData}
					<section class="p-4 rounded-lg bg-neutral max-h-[30vh] overflow-scroll flex-1 flex flex-col gap-3">
						{#if $postData?.comments == undefined || $postData?.comments == null}
							<span class="font-bold text-xl">No comments posted.</span>
						{:else if $postData?.comments.length == 0}
							<!-- Has to be checked afterwards because we cannot read length of undefined.-->
							<span class="font-bold text-xl">No comments posted.</span>
						{:else}
							{#each $postData?.comments.sort((a, b) => b.commentTime - a.commentTime) as curComment (postID + curComment.commentAuthorID + curComment.commentTime)}
								<Comment
									commentAuthorID={curComment.commentAuthorID}
									commentData={curComment.commentData}
									commentTime={curComment.commentTime}
									{postData}
									{postID}
								/>
							{/each}
						{/if}
					</section>
				{/if}
			</div>
			{#if $user}
				<div class="flex flex-col gap-2 text-center">
					<label for="publishComment" class="text-2xl font-bold">Post Comment:</label>
					<div class="flex flex-row gap-2 items-center flex-1">
						<textarea
							class="textarea textarea-lg textarea-accent flex-1 max-h-44 min-h-[7rem]"
							placeholder="Comment..."
							id="publishComment"
							maxlength="150"
							minlength="5"
							bind:value={postCommentData}
						/>
						<div class="divider lg:divider-horizontal" />
						<button
							class="btn btn-accent"
							on:click={() => {
								postComment($user, $postData, postID, postCommentData);
							}}>Upload</button
						>
					</div>
				</div>
			{/if}
		</div>

		<div class="flex flex-row justify-evenly items-center">
			{#if !onProfile}
				<a
					href="/profile/{$postData?.authorID}"
					class="p-1 btn-ghost flex flex-row gap-3 justify-center items-center rounded-md"
				>
					<img
						src={$authorData?.photoURL}
						alt="{$authorData?.username} profile"
						class="rounded-full aspect-square"
						style="border:2px solid hsl(var(--a))"
						width="50px"
						height="50px"
					/><span>{$authorData?.username}</span></a
				>
			{/if}
			{#if $postData?.authorID == $user?.uid}
				<button
					class="btn bg-base-300 rounded-lg bg-primary hover:bg-error text-white"
					on:click={() => {
						deletePost($user, postID, $postData, $authorData, false);
					}}
				>
					<span class="font-bold text-xl">DELETE POST</span>
				</button>
			{/if}
			<button
				class="btn bg-base-300 rounded-lg hover:bg-success {likedPost == true ? 'bg-primary text-white' : ''}"
				on:click={() => {
					likePost(postID, $postData, $user);
				}}
			>
				<img src="/Icons/heart.svg" width="24px" height="24px" alt="Like icon" />
				<span class="font-bold text-xl text-white">{$postData?.totalLikes}</span>
			</button>
		</div>
	</div>
</div>
