<script>
	import data from './regression-data.json';

	const metrics = [
		{ key: 'Tree Density',      lines: ['Tree', 'Density'],      unit: 'trees/km²' },
		{ key: 'Median DBH',        lines: ['Median', 'DBH'],        unit: 'cm'        },
		{ key: 'Basal Area',        lines: ['Basal', 'Area'],        unit: 'm²/km²'    },
		{ key: 'Species Diversity', lines: ['Species', 'Diversity'], unit: 'n'         },
	];

	const socioLabels = [
		{ lines: ['Population', 'Density']        },
		{ lines: ['Residential', 'Instability']   },
		{ lines: ['Ethno-cultural', 'Composition'] },
		{ lines: ['Economic', 'Dependency']       },
		{ lines: ['Situational', 'Vulnerability'] },
	];

	const colours = { pos: '#8DBF2E', ns: '#D0D1C9', neg: '#DC4633' };

	function buildDots(cell) {
		const dots = [];
		for (let i = 0; i < cell.pos; i++) dots.push('pos');
		for (let i = 0; i < cell.neg; i++) dots.push('neg');
		for (let i = 0; i < cell.ns;  i++) dots.push('ns');
		while (dots.length < 23) dots.push('ns');
		return dots.slice(0, 23);
	}

	// Fixed at 680px — matches usable width of .text container (720px - 2×20px padding)
	// SVG rendered with width=680 so 1 viewBox unit = 1 CSS px, zero scaling
	const VW      = 680;
	const labelW  = 150;
	const colW    = (VW - labelW) / metrics.length;  // 132.5px each

	const DOT_R   = 6;
	const DOT_GAP = 15;
	const CELL_W  = 5 * DOT_GAP;   // 75
	const CELL_H  = 5 * DOT_GAP;   // 75

	const headerH = 54;
	const rowH    = CELL_H + 30;   // 105

	const VH = headerH + data.length * rowH + 4;
</script>

<div class="chart-wrap">
	<!-- <p class="chart-title">Regression results: number of cities with significant associations</p> -->

	<div class="legend">
		{#each Object.entries(colours) as [key, col]}
			<span class="leg-item">
				<span class="leg-dot" style="background:{col}"></span>
				{key === 'pos' ? 'Positive association' : key === 'neg' ? 'Negative association' : 'Not significant'}
			</span>
		{/each}
		<span class="leg-note">Each dot = 1 city (n = 23)</span>
	</div>

	<svg viewBox="0 0 {VW} {VH}" width="680" style="max-width:100%; height:auto; display:block">

		<!-- metric column headers -->
		{#each metrics as m, mi}
			{@const gx = labelW + mi * colW + colW / 2}
			<text text-anchor="middle" class="col-header">
				<tspan x={gx} y={16}>{m.lines[0]}</tspan>
				<tspan x={gx} dy={15}>{m.lines[1]}</tspan>
			</text>
			<text x={gx} y={48} text-anchor="middle" class="col-unit">({m.unit})</text>
		{/each}

		<!-- column dividers -->
		{#each metrics as _, mi}
			<line x1={labelW + mi * colW} y1={0} x2={labelW + mi * colW} y2={VH} stroke="#bbb" stroke-width="1"/>
		{/each}
		<line x1={labelW + metrics.length * colW} y1={0} x2={labelW + metrics.length * colW} y2={VH} stroke="#bbb" stroke-width="1"/>

		<!-- data rows -->
		{#each data as row, ri}
			{@const ry  = headerH + ri * rowH}
			{@const sl  = socioLabels[ri]}

			<line x1={0} y1={ry} x2={VW} y2={ry} stroke="#bbb" stroke-width="0.75"/>

			<!-- row label: two lines, vertically centred -->
			<text x={labelW - 10} text-anchor="end" class="row-name">
				<tspan x={labelW - 7} y={ry + rowH/2 - 5}>{sl.lines[0]}</tspan>
				<tspan x={labelW - 10} dy={17}>{sl.lines[1]}</tspan>
			</text>

			<!-- waffle grids centred in cell -->
			{#each metrics as m, mi}
				{@const dots = buildDots(row[m.key])}
				{@const ox = labelW + mi * colW + (colW - CELL_W) / 2}
				{@const oy = ry + (rowH - CELL_H) / 2}
				{#each dots as type, di}
					<circle
						cx={ox + Math.floor(di / 5) * DOT_GAP + DOT_R}
						cy={oy + (di % 5) * DOT_GAP + DOT_R}
						r={DOT_R}
						fill={colours[type]}
						opacity="0.85"
					/>
				{/each}
			{/each}
		{/each}

		<line x1={0} y1={VH - 4} x2={VW} y2={VH - 4} stroke="#bbb" stroke-width="0.75"/>

	</svg>
</div>

<style>
	.chart-wrap {
		width: 100%;
		margin: 1.5rem 0;
	}

	.chart-title {
		font-family: OpenSansBold, sans-serif;
		font-size: 0.85rem;
		color: #333;
		margin: 0 0 0.5rem;
	}

	.legend {
		display: flex;
		flex-wrap: wrap;
		align-items: center;
		gap: 1rem;
		margin-bottom: 1.2rem;
		font-family: OpenSans, sans-serif;
		font-size: 0.78rem;
		color: #555;
	}

	.leg-item {
		display: flex;
		align-items: center;
		gap: 0.35rem;
	}

	/* DOT_R=6px at 1:1 scale → 12px diameter */
	.leg-dot {
		display: inline-block;
		width: 12px;
		height: 12px;
		border-radius: 50%;
		opacity: 0.85;
		flex-shrink: 0;
	}

	.leg-note {
		color: #999;
		font-style: italic;
	}

	svg {
		display: block;
	}

	.col-header {
		font-family: OpenSansBold, sans-serif;
		font-size: 14px;
		fill: #333;
	}

	.col-unit {
		font-family: OpenSans, sans-serif;
		font-size: 13px;
		fill: #888;
	}

	.row-name {
		font-family: OpenSansBold, sans-serif;
		font-size: 13.5px;
		fill: #333;
	}

	.row-unit {
		font-family: OpenSans, sans-serif;
		font-size: 12px;
		fill: #888;
	}
</style>
