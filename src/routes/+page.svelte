<script lang="ts">
	import Icon from '@iconify/svelte';
	import { page } from '$app/stores';
	import { replaceState } from '$app/navigation';

	// Get initial values from URL parameters
	const initialCount = parseInt($page.url.searchParams.get('count') || '100');
	const initialPaneForge = $page.url.searchParams.get('paneforge') === 'true';
	let iconCount = $state(initialCount);
	let usePaneForge = $state(initialPaneForge);

	// Generate array of icon names for testing - using known working icons
	function generateIconNames(count: number): string[] {
		// Using well-known, commonly available icons across different sets
		const iconList = [
			'mdi:home',
			'mdi:account',
			'mdi:cog',
			'mdi:magnify',
			'mdi:heart',
			'mdi:star',
			'mdi:plus',
			'mdi:minus',
			'mdi:pencil',
			'mdi:delete',
			'mdi:folder',
			'mdi:file',
			'mdi:image',
			'mdi:video',
			'mdi:music',
			'mdi:download',
			'mdi:upload',
			'mdi:share',
			'mdi:content-copy',
			'mdi:content-cut',
			'mdi:content-save',
			'mdi:printer',
			'mdi:email',
			'mdi:phone',
			'mdi:calendar',
			'mdi:clock',
			'mdi:map',
			'mdi:map-marker',
			'mdi:weather-cloudy',
			'mdi:weather-sunny',
			'mdi:weather-night',
			'mdi:cloud',
			'mdi:weather-rainy',
			'mdi:weather-snowy',
			'mdi:weather-windy',
			'mdi:fire',
			'mdi:water',
			'mdi:earth',
			'mdi:tree',
			'mdi:flower',
			'mdi:car',
			'mdi:airplane',
			'mdi:train',
			'mdi:ship',
			'mdi:bike',
			'mdi:walk',
			'mdi:run',
			'mdi:human-handsup',
			'mdi:music-note',
			'mdi:bell'
		];

		const icons = [];
		for (let i = 0; i < count; i++) {
			icons.push(iconList[i % iconList.length]);
		}
		return icons;
	}

	const icons = $derived(generateIconNames(iconCount));

	// Update URL parameters when values change using SvelteKit's replaceState
	$effect(() => {
		if (typeof window !== 'undefined') {
			const url = new URL($page.url);
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
		<p>Current: {iconCount} icons {usePaneForge ? 'in pane 2 only' : 'in simple grid'} (Total: {iconCount} icons)</p>
	</div>

	{#if usePaneForge}
		<!-- Import PaneForge dynamically when needed -->
		{#await import('paneforge') then { PaneGroup, Pane, PaneResizer }}
			<PaneGroup direction="horizontal" class="pane-group">
				<Pane defaultSize={50} minSize={20}>
					<div class="pane-content">
						<h3>Pane 1 (Empty)</h3>
						<p>This pane is empty to test if the issue is specific to multiple panes with icons.</p>
					</div>
				</Pane>

				<PaneResizer class="pane-resizer" />

				<Pane defaultSize={50} minSize={20}>
					<div class="pane-content">
						<h3>Pane 2 (Icons Only)</h3>
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
		{/await}
	{:else}
		<div class="simple-container">
			<h3>Simple Icon Grid (No PaneForge)</h3>
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

	.checkbox-label {
		font-size: 0.9rem;
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
