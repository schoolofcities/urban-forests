<script>
	import cityData from './city-data.json';

	export let ecozoneColours = {};

	const metrics = [
		{
			key: 'Gini Tree Count',
			label: 'Tree Count',
			definition: 'The Gini index for neighbourhood tree count measures how evenly street trees are distributed across a city\'s neighbourhoods. Values closer to 0 indicate more equal distribution; values closer to 1 indicate high inequality.',
		},
		{
			key: 'Gini Basal Area',
			label: 'Basal Area',
			definition: 'The Gini index for neighbourhood basal area captures inequality in the cumulative volume of street trees. Because basal area reflects both tree number and size, this measure captures inequalities in the overall benefits trees provide.',
		},
	];

	const sorts = [
		{ key: 'high-low',  label: 'High → Low' },
		{ key: 'low-high',  label: 'Low → High' },
		{ key: 'west-east', label: 'West → East' },
		{ key: 'east-west', label: 'East → West' },
	];

	let selectedMetric = metrics[0];
	let selectedSort = 'high-low';

	$: filtered = cityData.filter(d => d[selectedMetric.key] != null);

	$: rows = [...filtered].sort((a, b) => {
		if (selectedSort === 'high-low') return b[selectedMetric.key] - a[selectedMetric.key];
		if (selectedSort === 'low-high') return a[selectedMetric.key] - b[selectedMetric.key];
		if (selectedSort === 'west-east') return a.westEastOrder - b.westEastOrder;
		return b.westEastOrder - a.westEastOrder;
	});

	// Fixed scale 0–1 for Gini so bars are comparable across metrics
	const maxVal = 1;

	// Gridlines every 0.1 from 0.1 to 0.8; labels only at 0.2 and 0.5
	const refLines = [0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8].map(v => ({ val: v }));

	function barColour(ecozone) {
		return ecozoneColours[ecozone] ?? '#aaa';
	}

	function fmt(val) {
		return val.toFixed(3);
	}
</script>

<div class="chart-widget">

	<div class="metric-selector">
		{#each metrics as m}
			<button
				class="metric-btn"
				class:active={selectedMetric.key === m.key}
				on:click={() => selectedMetric = m}
			>
				{m.label}
			</button>
		{/each}
	</div>

	<div class="metric-info">
		<div class="metric-text">
			<h4>Gini index — {selectedMetric.label}</h4>
			<p>{selectedMetric.definition}</p>
		</div>
	</div>

	<div class="sort-row">
		<span class="sort-label">Sort:</span>
		{#each sorts as s}
			<button
				class="sort-btn"
				class:active={selectedSort === s.key}
				on:click={() => selectedSort = s.key}
			>
				{s.label}
			</button>
		{/each}
	</div>

	<div class="bar-chart-wrap">
		<!-- axis tick label row — same grid as bar rows so positions align exactly -->
		<div class="bar-row axis-row">
			<span></span>
			<div class="axis-track">
				{#each refLines as ref}
					<span class="ref-label" style="left: {ref.val * 100}%">{ref.val.toFixed(1)}</span>
				{/each}
			</div>
		</div>

		<div class="bar-chart">
			{#each rows as row}
				<div class="bar-row">
					<span class="city-label">{row.City}</span>
					<div class="bar-track">
						{#each refLines as ref}
							<div class="track-ref" style="left: {ref.val * 100}%"></div>
						{/each}
						<div
							class="bar-fill"
							style="width: {row[selectedMetric.key] * 100}%; background-color: {barColour(row.Ecozone)};"
						>
							<span class="bar-value">{fmt(row[selectedMetric.key])}</span>
						</div>
					</div>
				</div>
			{/each}
		</div>
	</div>

	<div class="legend">
		{#each Object.entries(ecozoneColours) as [name, colour]}
			<span class="legend-item">
				<span class="legend-swatch" style="background: {colour}"></span>
				{name}
			</span>
		{/each}
	</div>

</div>

<style>
	.chart-widget {
		width: 100%;
		font-family: OpenSans, sans-serif;
		margin-top: 30px;
		margin-bottom: 50px;
	}

	.metric-selector {
		display: flex;
		flex-wrap: wrap;
		gap: 0.5rem;
		margin-bottom: 1.25rem;
	}

	.metric-btn {
		padding: 0.35rem 0.85rem;
		border: 1.5px solid var(--brandLightGreen);
		border-radius: 2rem;
		background: #fbf9f6;
		cursor: pointer;
		font-size: 0.85rem;
		font-family: OpenSans, sans-serif;
		color: var(--brandDarkGreen);
		transition: background 0.15s, border-color 0.15s, color 0.15s;
	}

	.metric-btn.active {
		background: var(--brandDarkGreen);
		border-color: var(--brandDarkGreen);
		color: white;
	}

	.metric-btn:hover {
		opacity: 0.8;
	}

	.metric-info {
		margin-bottom: 1rem;
	}

	.metric-text h4 {
		margin: 0 0 0.3rem;
		font-size: 1rem;
		font-family: OpenSans, sans-serif;
		font-weight: 600;
	}

	.metric-text p {
		margin: 0;
		font-size: 0.85rem;
		color: #555;
		line-height: 1.5;
		font-family: OpenSans, sans-serif;
	}

	.sort-row {
		display: flex;
		align-items: center;
		gap: 0.4rem;
		margin-bottom: 0.85rem;
	}

	.sort-label {
		font-size: 0.8rem;
		color: #888;
		margin-right: 0.2rem;
		font-family: OpenSans, sans-serif;
	}

	.sort-btn {
		padding: 0.2rem 0.65rem;
		border: 1px solid var(--brandDarkGreen);
		border-radius: 2rem;
		background: none;
		cursor: pointer;
		font-size: 0.78rem;
		font-family: OpenSans, sans-serif;
		color: var(--brandDarkGreen);
		opacity: 0.5;
		transition: opacity 0.15s;
	}

	.sort-btn.active {
		opacity: 1;
	}

	.sort-btn:hover {
		opacity: 0.8;
	}

	.bar-chart-wrap {
		position: relative;
	}

	.axis-row {
		height: 18px;
		align-items: flex-end;
	}

	.axis-track {
		position: relative;
		height: 100%;
	}

	.ref-label {
		position: absolute;
		transform: translateX(-50%);
		bottom: 2px;
		font-family: OpenSans, sans-serif;
		font-size: 0.68rem;
		color: #999;
		white-space: nowrap;
	}

	.bar-chart {
		display: flex;
		flex-direction: column;
		gap: 3px;
	}

	.bar-row {
		display: grid;
		grid-template-columns: 140px 1fr;
		align-items: center;
		gap: 0.5rem;
	}

	.city-label {
		font-size: 0.78rem;
		text-align: right;
		white-space: nowrap;
		color: #333;
		font-family: OpenSans, sans-serif;
	}

	.bar-track {
		height: 16px;
		position: relative;
	}

	.track-ref {
		position: absolute;
		top: -3px;
		bottom: 0;
		width: 1px;
		background: #bbb;
		z-index: 1;
	}

	.bar-fill {
		height: 100%;
		border-radius: 2px;
		transition: width 0.35s ease;
		opacity: 0.9;
		display: flex;
		align-items: center;
		overflow: visible;
		position: relative;
		z-index: 2;
	}

	.bar-value {
		font-size: 0.72rem;
		color: #555;
		white-space: nowrap;
		font-family: OpenSans, sans-serif;
		padding-left: 5px;
	}

	.legend {
		display: flex;
		flex-wrap: wrap;
		gap: 0.75rem 1.25rem;
		margin-top: 1.25rem;
		font-size: 0.8rem;
		color: #444;
		font-family: OpenSans, sans-serif;
	}

	.legend-item {
		display: flex;
		align-items: center;
		gap: 0.35rem;
	}

	.legend-swatch {
		width: 14px;
		height: 14px;
		border-radius: 2px;
		display: inline-block;
		opacity: 0.9;
	}
</style>
