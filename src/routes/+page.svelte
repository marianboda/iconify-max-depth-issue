<script lang="ts">
	import { PaneGroup, Pane, PaneResizer } from 'paneforge';
	import Icon from '@iconify/svelte';

	let iconCount = $state(100);
	
	// Generate array of icon names for testing
	function generateIconNames(count: number): string[] {
		const iconSets = ['mdi', 'heroicons', 'tabler', 'carbon', 'lucide'];
		const iconNames = [
			'home', 'user', 'settings', 'search', 'heart', 'star', 'plus', 'minus', 'edit', 'delete',
			'folder', 'file', 'image', 'video', 'music', 'download', 'upload', 'share', 'copy', 'cut',
			'save', 'print', 'email', 'phone', 'calendar', 'clock', 'map', 'location', 'weather', 'sun',
			'moon', 'cloud', 'rain', 'snow', 'wind', 'fire', 'water', 'earth', 'tree', 'flower',
			'car', 'plane', 'train', 'ship', 'bike', 'walk', 'run', 'jump', 'dance', 'music-note'
		];
		
		const icons = [];
		for (let i = 0; i < count; i++) {
			const set = iconSets[i % iconSets.length];
			const name = iconNames[i % iconNames.length];
			icons.push(`${set}:${name}`);
		}
		return icons;
	}
	
	const icons = $derived(generateIconNames(iconCount));
</script>

<div class="container">
	<div class="controls">
		<h1>Iconify Max Depth Issue Reproduction</h1>
		<label>
			Icon Count: 
			<input type="number" bind:value={iconCount} min="1" max="5000" step="50" />
		</label>
		<p>Current: {iconCount} icons in pane 2 only (Total: {iconCount} icons)</p>
	</div>

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
	
	.controls label {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		margin-bottom: 0.5rem;
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
</style>
