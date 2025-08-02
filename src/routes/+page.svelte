<script lang="ts">
	import Icon from '@iconify/svelte';
	import { page } from '$app/stores';
	import { replaceState } from '$app/navigation';
	import { PaneGroup, Pane, PaneResizer } from 'paneforge';

	// State initialization from URL parameters
	let iconCount = $state(parseInt($page.url.searchParams.get('count') || '100'));
	let usePaneForge = $state($page.url.searchParams.get('paneforge') === 'true');

	// Minimal icon set for testing
	const baseIcons = [
		'mdi:home',
		'mdi:heart',
		'mdi:star',
		'mdi:cog',
		'mdi:account',
		'mdi:folder',
		'mdi:calendar',
		'mdi:email'
	];
	const icons = $derived(
		Array.from({ length: iconCount }, (_, i) => baseIcons[i % baseIcons.length])
	);

	// URL parameter sync effect
	$effect(() => {
		if (typeof window !== 'undefined') {
			const url = new URL(window.location.href);
			url.searchParams.set('count', iconCount.toString());
			url.searchParams.set('paneforge', usePaneForge.toString());
			replaceState(url.toString(), {});
		}
	});
</script>

<div class="container">
	<div class="controls">
		<h1>Iconify Max Depth Issue Reproduction</h1>
		<div class="control-group">
			<label>
				Icon Count:
				<input type="number" bind:value={iconCount} min="1" max="5000" step="50" />
			</label>
			<label class="checkbox-label">
				<input type="checkbox" bind:checked={usePaneForge} />
				Use PaneForge (uncheck to test without PaneForge)
			</label>
		</div>
		<p>
			Current: {iconCount} icons {usePaneForge ? 'in pane 2 only' : 'in simple grid'} (Total: {iconCount}
			icons)
		</p>
	</div>

	{#if usePaneForge}
		<PaneGroup direction="horizontal" class="pane-group">
			<Pane defaultSize={50} minSize={20}>
				<div class="pane-content">
					<h3>Pane 1 (Empty)</h3>
					<p>Empty pane for testing</p>
				</div>
			</Pane>
			<PaneResizer class="pane-resizer" />
			<Pane defaultSize={50} minSize={20}>
				<div class="pane-content">
					<h3>Pane 2 (Icons)</h3>
					<div class="icon-grid">
						{#each icons as iconName, i}
							<div class="icon-item">
								<Icon icon={iconName} width="24" height="24" />
								<span>{i + 1}</span>
							</div>
						{/each}
					</div>
				</div>
			</Pane>
		</PaneGroup>
	{:else}
		<div class="simple-container">
			<h3>Simple Icon Grid</h3>
			<div class="icon-grid">
				{#each icons as iconName, i}
					<div class="icon-item">
						<Icon icon={iconName} width="24" height="24" />
						<span>{i + 1}</span>
					</div>
				{/each}
			</div>
		</div>
	{/if}
</div>

<style>
	.container {
		height: 100vh;
		display: flex;
		flex-direction: column;
	}

	.controls {
		padding: 1rem;
		border-bottom: 1px solid #ccc;
		background: #f5f5f5;
	}
	.controls h1 {
		margin: 0 0 1rem 0;
		font-size: 1.5rem;
	}
	.control-group {
		display: flex;
		gap: 2rem;
		margin-bottom: 0.5rem;
	}
	.controls label {
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}
	.controls input {
		padding: 0.25rem;
		border: 1px solid #ccc;
		border-radius: 4px;
	}

	:global(.pane-group) {
		flex: 1;
	}

	.pane-content {
		height: 100%;
		padding: 1rem;
		overflow-y: auto;
		background: white;
	}
	.pane-content h3 {
		margin: 0 0 1rem 0;
		color: #333;
	}

	.icon-grid {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(60px, 1fr));
		gap: 0.5rem;
	}
	.icon-item {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 0.25rem;
		padding: 0.5rem;
		border: 1px solid #eee;
		border-radius: 4px;
		font-size: 0.75rem;
		color: #666;
	}

	:global(.pane-resizer) {
		background: #ddd;
		width: 4px;
		cursor: col-resize;
	}

	:global(.pane-resizer:hover) {
		background: #bbb;
	}

	.simple-container {
		flex: 1;
		padding: 1rem;
		background: white;
		overflow-y: auto;
	}
	.simple-container h3 {
		margin: 0 0 1rem 0;
		color: #333;
	}
</style>
