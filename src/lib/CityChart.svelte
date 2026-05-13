<script>
	import cityData from './city-data.json';

	import iconDbh       from '../assets/icon-dbh.svg';
	import iconBasal     from '../assets/icon-basal-area.svg';
	import iconDensity   from '../assets/icon-density.svg';
	import iconRichness  from '../assets/icon-richness.svg';
	import iconNative    from '../assets/icon-native.svg';

	export let ecozoneColours = {};

	const metrics = [
		{
			key: 'Street Tree Density',
			label: 'Street Tree Density',
			unit: 'trees/km²',
			icon: iconDensity,
			definition: 'The number of street trees per square kilometre of city land area. Higher density suggests more trees lining streets, which increases the potential for shade, stormwater absorption, and urban cooling.',
		},
		{
			key: 'Median DBH (cm)',
			label: 'Median DBH',
			unit: 'cm',
			icon: iconDbh,
			definition: 'Diameter at breast height (DBH) is measured at 1.3 m above ground. The median DBH indicates the typical trunk size across a city\'s street tree population — larger values suggest a more mature urban forest.',
		},
		{
			key: 'Basal Area Density',
			label: 'Basal Area Density',
			unit: 'm²/km²',
			icon: iconBasal,
			definition: 'Basal area density is the total cross-sectional area of all tree trunks (measured at breast height) per square kilometre of city land. It captures both tree size and abundance, making it a good proxy for overall canopy biomass.',
		},
		{
			key: 'Species Richness',
			label: 'Species Richness',
			unit: 'n',
			icon: iconRichness,
			definition: 'The total number of distinct tree species found in a city\'s street tree inventory. Greater diversity reduces the risk of widespread canopy loss from pests or disease, and supports richer urban ecosystems.',
		},
		{
			key: 'Native %',
			label: 'Native Species',
			unit: '%',
			icon: iconNative,
			definition: 'The proportion of street trees belonging to species native to the local region. Native trees are better adapted to local climate and soils, provide more value to native wildlife, and often require less maintenance.',
		},
	];

	const sorts = [
		{ key: 'high-low', label: 'High → Low' },
		{ key: 'low-high', label: 'Low → High' },
		{ key: 'west-east', label: 'West → East' },
	];

	let selectedMetric = metrics[0];
	let selectedSort = 'high-low';

	$: filtered = cityData.filter(d => d[selectedMetric.key] != null && d[selectedMetric.key] !== 0);

	$: rows = [...filtered].sort((a, b) => {
		if (selectedSort === 'high-low') return b[selectedMetric.key] - a[selectedMetric.key];
		if (selectedSort === 'low-high') return a[selectedMetric.key] - b[selectedMetric.key];
		return a.westEastOrder - b.westEastOrder;
	});

	$: maxVal = filtered.length
		? Math.max(...filtered.map(d => d[selectedMetric.key]))
		: 1;

	function barColour(ecozone) {
		return ecozoneColours[ecozone] ?? '#aaa';
	}

	function fmt(val, unit) {
		if (unit === '%') return val.toFixed(1) + '%';
		if (unit === 'cm') return val.toFixed(1) + ' cm';
		if (Number.isInteger(val)) return val.toLocaleString();
		return val.toFixed(1);
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
		<img src={selectedMetric.icon} alt={selectedMetric.label} class="metric-icon" />
		<div class="metric-text">
			<h4>{selectedMetric.label} <span class="unit">({selectedMetric.unit})</span></h4>
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

	<div class="bar-chart">
		{#each rows as row}
			<div class="bar-row">
				<span class="city-label">{row.City}</span>
				<div class="bar-track">
					<div
						class="bar-fill"
						style="width: {(row[selectedMetric.key] / maxVal) * 100}%; background-color: {barColour(row.Ecozone)};"
					>
						<span class="bar-value">{fmt(row[selectedMetric.key], selectedMetric.unit)}</span>
					</div>
				</div>
			</div>
		{/each}
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
		background: white;
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

	.metric-info {
		display: flex;
		align-items: flex-start;
		gap: 1rem;
		margin-bottom: 1rem;
	}

	.metric-icon {
		width: 64px;
		height: 64px;
		flex-shrink: 0;
	}

	.metric-text h4 {
		margin: 0 0 0.3rem;
		font-size: 1rem;
		font-family: OpenSans, sans-serif;
		font-weight: 600;
	}

	.metric-text .unit {
		font-weight: 400;
		color: #777;
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
	}

	.bar-fill {
		height: 100%;
		border-radius: 2px;
		transition: width 0.35s ease;
		opacity: 0.9;
		display: flex;
		align-items: center;
		overflow: visible;
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
