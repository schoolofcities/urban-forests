<script>
	const size = 360;
	const pad = { top: 20, right: 20, bottom: 100, left: 100 };
	const W = size - pad.left - pad.right;
	const H = size - pad.top - pad.bottom;

	// Generate a Lorenz curve for Gini ≈ 0.5
	// Using a power curve: y = x^k where k drives the Gini
	// For Gini = 0.5, Gini = (k-1)/(k+1) → k = 3
	const N = 60;
	const lorenzPts = Array.from({ length: N + 1 }, (_, i) => {
		const x = i / N;
		const y = Math.pow(x, 3);
		return { x, y };
	});

	function sx(x) { return pad.left + x * W; }
	function sy(y) { return pad.top + (1 - y) * H; }

	const lorenzPath = lorenzPts
		.map((p, i) => `${i === 0 ? 'M' : 'L'}${sx(p.x).toFixed(1)},${sy(p.y).toFixed(1)}`)
		.join(' ');

	const equalityPath = `M${sx(0)},${sy(0)} L${sx(1)},${sy(1)}`;

	// shaded area between curves
	const shadePath = [
		...lorenzPts.map((p, i) => `${i === 0 ? 'M' : 'L'}${sx(p.x).toFixed(1)},${sy(p.y).toFixed(1)}`),
		`L${sx(1)},${sy(1)}`,
		`L${sx(0)},${sy(0)}`,
		'Z'
	].join(' ');

	const ticks = [0, 0.25, 0.5, 0.75, 1];
	const tickFmt = v => v === 0 ? '0' : v === 1 ? '100%' : `${v * 100}%`;

	// equality line label — centred at midpoint of the line, offset perpendicular (up-left)
	const eqLabelX = sx(0.5);
	const eqLabelY = sy(0.5) - 10;

	// lorenz curve label — to the right of curve at ~x=0.72
	const lorenzLabelX = sx(0.72) + 6;
	const lorenzLabelY = sy(Math.pow(0.72, 3)) + 4;

	// A label — midpoint between equality line and lorenz curve at x≈0.45
	const aLabelX = sx(0.45);
	const aLabelY = sy((0.45 + Math.pow(0.45, 3)) / 2);

	// B label — midpoint between lorenz curve and x-axis at x≈0.65
	const bLabelX = sx(0.65);
	const bLabelY = sy(Math.pow(0.65, 3) / 2);

	// B shading path — area under lorenz curve down to x-axis
	const bShadePath = [
		...lorenzPts.map((p, i) => `${i === 0 ? 'M' : 'L'}${sx(p.x).toFixed(1)},${sy(p.y).toFixed(1)}`),
		`L${sx(1)},${sy(0)}`,
		`L${sx(0)},${sy(0)}`,
		'Z'
	].join(' ');
</script>

<div class="lorenz-container">
	<svg viewBox="0 0 {size} {size}" preserveAspectRatio="xMidYMid meet">

		<!-- B shading — under lorenz curve -->
		<path d={bShadePath} fill="#c8ddd8" opacity="0.6" />

		<!-- A shading — between equality line and lorenz curve -->
		<path d={shadePath} fill="#e8d5b0" opacity="0.6" />

		<!-- axes -->
		<line x1={sx(0)} y1={sy(0)} x2={sx(1)} y2={sy(0)} stroke="#bbb" stroke-width="1" />
		<line x1={sx(0)} y1={sy(0)} x2={sx(0)} y2={sy(1)} stroke="#bbb" stroke-width="1" />

		<!-- tick marks and labels -->
		{#each ticks as t}
			<!-- x axis -->
			<line x1={sx(t)} y1={sy(0)} x2={sx(t)} y2={sy(0) + 4} stroke="#bbb" stroke-width="1" />
			<text x={sx(t)} y={sy(0) + 14} text-anchor="middle" class="tick-label">{tickFmt(t)}</text>
			<!-- y axis -->
			<line x1={sx(0) - 4} y1={sy(t)} x2={sx(0)} y2={sy(t)} stroke="#bbb" stroke-width="1" />
			<text x={sx(0) - 8} y={sy(t) + 4} text-anchor="end" class="tick-label">{tickFmt(t)}</text>
		{/each}

		<!-- equality line -->
		<path d={equalityPath} stroke="#00A189" stroke-width="1.5" stroke-dasharray="5 4" fill="none" />

		<!-- lorenz curve -->
		<path d={lorenzPath} stroke="#0D534D" stroke-width="2" fill="none" />

		<!-- equality line label — centred on line -->
		<text x={eqLabelX} y={eqLabelY} text-anchor="middle" class="curve-label equality-label" transform="rotate(-45, {eqLabelX}, {eqLabelY})">Line of perfect equality</text>

		<!-- lorenz curve label — two lines -->
		<text x={lorenzLabelX} y={lorenzLabelY} class="curve-label gini-label">
			<tspan x={lorenzLabelX} dy="0">Lorenz</tspan>
			<tspan x={lorenzLabelX} dy="14">curve</tspan>
		</text>

		<!-- A area label -->
		<text x={aLabelX} y={aLabelY} text-anchor="middle" class="area-label">A</text>

		<!-- B area label -->
		<text x={bLabelX} y={bLabelY} text-anchor="middle" class="area-label">B</text>

		<!-- x-axis title — just below axis line -->
		<text
			x={pad.left + W / 2}
			y={sy(0) + 33}
			text-anchor="middle"
			class="axis-title"
		>Cumulative population share</text>

		<!-- y-axis title — horizontal, inside plot area near top of axis -->
		<text
			x={pad.left + 6}
			y={sy(1) + 14}
			text-anchor="start"
			class="axis-title"
		>
			<tspan x={pad.left + 6} dy="0">Cumulative</tspan>
			<tspan x={pad.left + 6} dy="15">tree count</tspan>
		</text>

	</svg>
	<p class="formula">Gini = A / (A + B)</p>
</div>

<style>
	.lorenz-container {
		width: 100%;
	}

	@media (max-width: 1300px) {
		.lorenz-container {
			margin-left: -50px;
		}
	}

	svg {
		width: 100%;
		height: auto;
		display: block;
	}

	.tick-label {
		font-family: OpenSans, sans-serif;
		font-size: 11px;
		fill: #666;
	}

	.curve-label {
		font-family: OpenSans, sans-serif;
		font-size: 11px;
		fill: #555;
	}

	.equality-label {
		fill: #00A189;
	}

	.gini-label {
		fill: #0D534D;
		font-family: OpenSansBold, sans-serif;
	}

	.area-label {
		font-family: OpenSansBold, sans-serif;
		font-size: 13px;
		fill: #000000;
	}

	.formula {
		font-family: OpenSansBold, sans-serif;
		font-size: 12px;
		color: #000000;
		margin: 4px 0 0 0;
		text-align: center;
		margin-top: -40px;
		margin-left: -75px;
	}

	.axis-title {
		font-family: OpenSans, sans-serif;
		font-size: 12px;
		fill: #444;
	}
</style>
