<script lang="ts">
	import { onMount } from 'svelte';
	var new_projects: any[] = [];
	var offset = 0;
	var step = 100;
	var projects: any[] = [];
	var visible = true;
	var x: HTMLDivElement;
	var loadMoreButton: HTMLButtonElement;
	var disabled = false;

	let loadNext = async () => {
		let response = await fetch(
			'https://api.modrinth.com/v2/search?limit=' +
				step +
				'&offset=' +
				offset +
				'&facets=[["project_type:mod"]]&filters=categories="quilt" AND (NOT categories="fabric")'
		);
		let body = await response.json();
		body.hits.forEach((project) => {
			console.log(project);
			projects.push(project);
		});
		if (body.hits.length == 0) {
			loadMoreButton.disabled = true;
			disabled = true;
			return;
		}
		visible = false;
		setTimeout(() => (visible = true), 100);
		offset += step;
	};

	onMount(async () => {
		loadNext();
	});
</script>

<svelte:head>
	<title>Quilt-only mods (on modrinth)</title>
</svelte:head>
<h1>Quilt-only mods from Modrinth</h1>
<h2>Shows mods which are Quilt-only on Modrinth.</h2>
<h3>This project is not affiliated with, nor endorsed by, <i><b>Rinth, Inc.</b></i> or <b><i>The Quilt Project</i></b></h3>
<div bind:this={x}>
	{#if visible}
		{#each projects as project}
			<div role="link" on:keypress={(x) => {
                if (x.key == "Enter" || x.key == " ") {
                    window.open('https://modrinth.com/project/' + project.project_id, '_blank');
                } 
            }} tabindex="0"
				on:click={() => {
					window.open('https://modrinth.com/project/' + project.project_id, '_blank');
				}}
				class="project"
			>
				<div class="image-text-align">
					<div class="image-align">
						<img alt={project.title + ' logo'} src={project.icon_url} />
					</div>
					<div class="text-align">
						<h2>{project.title}</h2>
						<p>{project.description}</p>
					</div>
				</div>
			</div>
		{/each}
	{/if}
	<button bind:this={loadMoreButton} on:click={loadNext}>Load More</button>
	{#if disabled}
		<p style="font-weight: bolder; text-color: red;">No more results.</p>
	{/if}
</div>

<style>
	div.project {
		border-radius: 15px;
		background-color: grey;
		width: max-content;
		padding: 5px;
		margin-bottom: 10px;
		min-width: 60rem;
		max-width: 60rem;
		cursor: pointer;
	}
	div.image-align > img {
		width: 96px;
		height: 96px;
		border-radius: 0.75rem;
		border-right: 8rem;
	}
	div.image-align {
		border-right: 8rem;
		max-width: 96px;
	}
	div.image-text-align {
		display: flex;
	}
	div.text-align {
		margin-left: 0.75rem;
	}
	div.text-align > h2:hover {
		text-decoration: underline;
	}
</style>
