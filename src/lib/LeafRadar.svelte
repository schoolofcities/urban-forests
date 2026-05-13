<script>
	import { tweened } from 'svelte/motion';
	import { cubicOut } from 'svelte/easing';
	import cityData from './city-data.json';

	export let cityA = 'Winnipeg';
	export let cityB = 'Montreal';
	export let ecozoneColours = {};

	// Lobe shape controls — tweak these to adjust roundedness/width
	export let lobeSpread   = 28;   // half-angle of base spread in degrees (larger = wider base)
	export let lobeShoulder = 0.62; // how far along axis the shoulder sits (0–1)
	export let lobeWidth    = 0.55; // lateral offset of shoulder as fraction of r (larger = rounder)

	const axes = [
		{ key: 'Street Tree Density', label: 'Street tree\ndensity', unit: 'trees/km²', max: 1300, angle: 180 },
		{ key: 'Median DBH (cm)',      label: 'Median\nDBH',          unit: 'cm',         max: 35,   angle: 135 },
		{ key: 'Basal Area Density',   label: 'Basal\narea',          unit: 'm²/km²',     max: 145,  angle: 90  },
		{ key: 'Species Richness',     label: 'Species\nrichness',    unit: '',            max: 375,  angle: 45  },
		{ key: 'Native %',             label: '% native\ntrees',      unit: '%',           max: 90,  angle: 0   },
	];

	// colour scale: low = red, high = dark green
	const colourStops = ['#DC4633', '#F1C500', '#8DBF2E', '#00A189', '#0D534D'];

	function valueColour(val, max) {
		if (val === null || val === 0) return colourStops[0];
		const t = Math.min(val / max, 1);
		const seg = t * (colourStops.length - 1);
		const lo = Math.floor(seg);
		const hi = Math.min(lo + 1, colourStops.length - 1);
		return lerpColour(colourStops[lo], colourStops[hi], seg - lo);
	}

	function lerpColour(a, b, t) {
		const ah = a.slice(1), bh = b.slice(1);
		const ar = parseInt(ah.slice(0,2),16), ag = parseInt(ah.slice(2,4),16), ab = parseInt(ah.slice(4,6),16);
		const br = parseInt(bh.slice(0,2),16), bg = parseInt(bh.slice(2,4),16), bb = parseInt(bh.slice(4,6),16);
		const r  = Math.round(ar + (br-ar)*t).toString(16).padStart(2,'0');
		const g  = Math.round(ag + (bg-ag)*t).toString(16).padStart(2,'0');
		const bv = Math.round(ab + (bb-ab)*t).toString(16).padStart(2,'0');
		return `#${r}${g}${bv}`;
	}

	// SVG geometry
	// Each chart is W × VH. Two charts side-by-side total ~1080px.
	const W        = 480;
	const labelPad = 52;  // room above maxR for labels
	const bottomPad = 14; // room below cy so 0°/180° lobes don't clip
	const maxR     = 170;
	const cx       = W / 2;
	const cy       = maxR + labelPad;
	const VH       = maxR + labelPad + bottomPad;
	const rings    = 4;

	function toRad(deg) { return (deg * Math.PI) / 180; }

	function polarToXY(angleDeg, r) {
		const a = toRad(angleDeg);
		return { x: cx + r * Math.cos(a), y: cy - r * Math.sin(a) };
	}

	// Rounded lobe: cubic bezier with two shoulder control points that sit
	// perpendicular to the petal axis, giving a broad teardrop shape.
	function petalPath(angleDeg, r) {
		const spread   = lobeSpread;
		const shoulder = lobeShoulder;
		const width    = lobeWidth;

		const axRad = toRad(angleDeg);
		const perpX =  Math.sin(axRad); // perpendicular unit vector
		const perpY =  Math.cos(axRad);

		const tip   = polarToXY(angleDeg, r);
		// shoulder control points: out along axis by shoulder*r, offset laterally by width*r
		const sh = shoulder * r;
		const wd = width * r;
		const cp1x = cx + sh * Math.cos(axRad) + wd * perpX;
		const cp1y = cy - sh * Math.sin(axRad) + wd * perpY;
		const cp2x = cx + sh * Math.cos(axRad) - wd * perpX;
		const cp2y = cy - sh * Math.sin(axRad) - wd * perpY;

		// base spread points at origin
		const baseSpread = toRad(spread);
		const b1x = cx + 8 * Math.cos(axRad + baseSpread);
		const b1y = cy - 8 * Math.sin(axRad + baseSpread);
		const b2x = cx + 8 * Math.cos(axRad - baseSpread);
		const b2y = cy - 8 * Math.sin(axRad - baseSpread);

		return [
			`M ${b1x.toFixed(2)} ${b1y.toFixed(2)}`,
			`C ${cp1x.toFixed(2)} ${cp1y.toFixed(2)} ${tip.x.toFixed(2)} ${tip.y.toFixed(2)} ${tip.x.toFixed(2)} ${tip.y.toFixed(2)}`,
			`C ${tip.x.toFixed(2)} ${tip.y.toFixed(2)} ${cp2x.toFixed(2)} ${cp2y.toFixed(2)} ${b2x.toFixed(2)} ${b2y.toFixed(2)}`,
			`L ${cx} ${cy} Z`
		].join(' ');
	}

	function getValue(cityName, key) {
		const row = cityData.find(d => d.City === cityName);
		return row ? (row[key] ?? null) : null;
	}

	function petalR(cityName, axis) {
		const val = getValue(cityName, axis.key);
		if (val === null) return 0;
		return (val / axis.max) * maxR;
	}

	function ringPath(r) {
		return `M ${cx - r} ${cy} A ${r} ${r} 0 0 1 ${cx + r} ${cy}`;
	}

	function fmtVal(val, unit) {
		if (val === null) return 'No data';
		const v = Math.round(val);
		return v.toLocaleString() + (unit ? ' ' + unit : '');
	}

	// Label placement: push labels a bit further out and vertically centre multiline
	function labelAnchor(angle) {
		if (angle >= 135) return 'end';
		if (angle <= 45)  return 'start';
		return 'middle';
	}

	const cityList = [...cityData].sort((a, b) => a.City.localeCompare(b.City));

	let selectedA = cityA;
	let selectedB = cityB;

	const tweenOpts = { duration: 400, easing: cubicOut };

	function initialRadii(cityName) {
		return axes.map(ax => petalR(cityName, ax));
	}

	const radiiA = tweened(initialRadii(selectedA), tweenOpts);
	const radiiB = tweened(initialRadii(selectedB), tweenOpts);

	$: radiiA.set(initialRadii(selectedA));
	$: radiiB.set(initialRadii(selectedB));

	// custom dropdown open state per chart
	let openDropdown = [false, false];

	function toggleDropdown(ci) {
		openDropdown = openDropdown.map((v, i) => i === ci ? !v : false);
	}

	function selectCity(ci, name, setFn) {
		setFn(name);
		openDropdown = [false, false];
	}

	function getCityInfo(name) {
		return cityData.find(d => d.City === name) ?? null;
	}

	let tooltips = [null, null];

	function showTooltip(ci, event, label, val, unit) {
		const svgRect = event.currentTarget.closest('svg').getBoundingClientRect();
		const t = [...tooltips];
		t[ci] = {
			x: event.clientX - svgRect.left,
			y: event.clientY - svgRect.top - 10,
			label: label.replace('\n', ' '),
			value: fmtVal(val, unit),
		};
		tooltips = t;
	}

	function handleOutsideClick(e) {
		if (!e.target.closest('.dropdown')) openDropdown = [false, false];
	}

	function hideTooltip(ci) {
		const t = [...tooltips];
		t[ci] = null;
		tooltips = t;
	}
</script>

<svelte:window on:click={handleOutsideClick} />

<div class="radar-widget">
	<div class="charts">
		{#each [
			{ city: selectedA, set: v => { selectedA = v; }, radii: $radiiA },
			{ city: selectedB, set: v => { selectedB = v; }, radii: $radiiB }
		] as cfg, ci}
		{@const info = getCityInfo(cfg.city)}
		<div class="chart-wrap">

			<!-- custom dropdown -->
			<div class="dropdown" class:open={openDropdown[ci]}>
				<button class="dropdown-trigger" on:click={() => toggleDropdown(ci)}>
					<span>{cfg.city}</span>
					<span class="chevron">▾</span>
				</button>
				{#if openDropdown[ci]}
					<ul class="dropdown-list">
						{#each cityList as row}
							<li>
								<button
									class="dropdown-option"
									class:selected={row.City === cfg.city}
									on:click={() => selectCity(ci, row.City, cfg.set)}
								>{row.City}</button>
							</li>
						{/each}
					</ul>
				{/if}
			</div>
			{#if info}
				<p class="city-sub">
					<span class="city-name">{info.City},</span>
					<span class="city-prov">{info.Province}</span>
					<span class="city-dot">·</span>
					<span class="ecozone-swatch" style="background:{ecozoneColours[info.Ecozone] ?? '#aaa'}"></span>
					<span class="city-eco">{info.Ecozone}</span>
				</p>
			{/if}

			<div class="svg-wrap">
				<svg viewBox="0 0 {W} {VH}" preserveAspectRatio="xMidYMax meet">

					{#each Array(rings) as _, i}
						<path d={ringPath(maxR * (i + 1) / rings)} fill="none" stroke="#ddd" stroke-width="1" stroke-dasharray="4 3" />
					{/each}

					{#each axes as ax}
						{@const end = polarToXY(ax.angle, maxR)}
						<line x1={cx} y1={cy} x2={end.x} y2={end.y} stroke="#ccc" stroke-width="1" stroke-dasharray="4 3" />
					{/each}

					{#each axes as ax, ai}
						{@const r = cfg.radii[ai]}
						{@const val = getValue(cfg.city, ax.key)}
						{#if r > 4}
							<path
								d={petalPath(ax.angle, r)}
								fill={valueColour(val, ax.max)}
								opacity="0.82"
								style="cursor:pointer"
								on:mouseenter={e => showTooltip(ci, e, ax.label, val, ax.unit)}
								on:mouseleave={() => hideTooltip(ci)}
							/>
						{/if}
					{/each}

					{#each axes as ax}
						{@const lp = polarToXY(ax.angle, maxR + 16)}
						{@const anchor = labelAnchor(ax.angle)}
						{@const lines = ax.label.split('\n')}
						<text text-anchor={anchor} class="axis-label">
							{#each lines as line, li}
								<tspan x={lp.x} y={lp.y} dy={li === 0 ? -(lines.length - 1) * 6 : li * 12}>{line}</tspan>
							{/each}
						</text>
					{/each}

				</svg>

				{#if tooltips[ci]}
					<div class="tooltip" style="left:{tooltips[ci].x}px; top:{tooltips[ci].y}px;">
						<span class="tt-label">{tooltips[ci].label}</span>
						<span class="tt-value">{tooltips[ci].value}</span>
					</div>
				{/if}
			</div>

		</div>
		{/each}
	</div>
</div>

<style>
	.radar-widget {
		width: 100%;
		max-width: 1080px;
		margin: 0 auto;
		padding: 1rem 0 2rem;
	}

	.charts {
		display: flex;
		gap: 2rem;
		align-items: flex-end;
		justify-content: center;
	}

	@media (max-width: 700px) {
		.charts {
			flex-direction: column;
			align-items: center;
			gap: 3rem;
		}
	}

	.chart-wrap {
		display: flex;
		flex-direction: column;
		align-items: center;
		flex: 1;
		min-width: 0;
		max-width: 500px;
	}

	.dropdown {
		position: relative;
		width: 100%;
		max-width: 190px;
		margin-bottom: 0.2rem;
	}

	.dropdown-trigger {
		width: 100%;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0.25rem 0.6rem;
		border: 1.5px solid var(--brandDarkGreen);
		border-radius: 4px;
		background: #fbf9f6;
		color: var(--brandDarkGreen);
		font-family: OpenSansBold, sans-serif;
		font-size: 0.85rem;
		cursor: pointer;
		text-align: left;
	}

	.dropdown-trigger .chevron {
		color: var(--brandDarkGreen);
		flex-shrink: 0;
		margin-left: 0.4rem;
		font-size: 0.75rem;
		line-height: 1;
		transition: transform 0.15s;
		display: inline-block;
	}

	.dropdown.open .chevron {
		transform: rotate(180deg);
	}

	.dropdown-list {
		position: absolute;
		top: calc(100% + 2px);
		left: 0;
		right: 0;
		background: #fbf9f6;
		border: 1.5px solid var(--brandDarkGreen);
		border-radius: 4px;
		max-height: 200px;
		overflow-y: auto;
		z-index: 100;
		margin: 0;
		padding: 0;
		list-style: none;
	}

	.dropdown-list li {
		margin: 0;
		padding: 0;
	}

	.dropdown-option {
		width: 100%;
		text-align: left;
		padding: 0.28rem 0.6rem;
		background: none;
		border: none;
		font-family: OpenSans, sans-serif;
		font-size: 0.82rem;
		color: #222;
		cursor: pointer;
	}

	.dropdown-option:hover {
		background: #e8f2f0;
		color: var(--brandDarkGreen);
	}

	.dropdown-option.selected {
		background: var(--brandDarkGreen);
		color: #fbf9f6;
		font-family: OpenSansBold, sans-serif;
	}

	.city-sub {
		font-family: OpenSans, sans-serif;
		font-size: 0.82rem;
		color: #222;
		margin: 0 0 0.5rem 0;
		text-align: left;
		display: flex;
		align-items: center;
		gap: 0.3rem;
		flex-wrap: wrap;
	}

	.city-dot {
		color: #aaa;
	}

	.ecozone-swatch {
		display: inline-block;
		width: 9px;
		height: 9px;
		border-radius: 2px;
		flex-shrink: 0;
	}

	.svg-wrap {
		position: relative;
		width: 100%;
	}

	svg {
		width: 100%;
		height: auto;
		display: block;
		overflow: visible;
	}

	.axis-label {
		font-family: OpenSans, sans-serif;
		font-size: 13px;
		fill: #333;
		line-height: 1.1;
	}

	.tooltip {
		position: absolute;
		pointer-events: none;
		transform: translate(-50%, -100%);
		background: rgba(0,0,0,0.78);
		color: white;
		border-radius: 4px;
		padding: 4px 8px;
		display: flex;
		flex-direction: column;
		align-items: center;
		white-space: nowrap;
		gap: 1px;
	}

	.tt-label {
		font-family: OpenSans, sans-serif;
		font-size: 0.75rem;
		opacity: 0.85;
	}

	.tt-value {
		font-family: OpenSansBold, sans-serif;
		font-size: 0.8rem;
	}
</style>
