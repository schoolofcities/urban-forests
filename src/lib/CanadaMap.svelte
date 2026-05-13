<script>
	import { geoConicConformal, geoPath } from 'd3-geo';
	import provincesData from './provinces.geo.json';
	import ecozonesData from './ecozones.geo.json';
	import cityData from './city-data.json';

	const HIGHLIGHTED_ECOZONES = [
		'Pacific Maritime',
		'Montane Cordillera',
		'Prairie',
		'MixedWood Plain',
		'Atlantic Maritime',
	];

	// Normalise GeoJSON zone names to display names
	const ZONE_NAME_MAP = { 'MixedWood Plain': 'Mixedwood Plain' };
	function normZone(name) { return ZONE_NAME_MAP[name] ?? name; }

	export let ecozoneColours = {};

	const cities = cityData.map(d => ({ name: d.City, province: d.Province, coords: [d.lng, d.lat] }));

	const width = 360;
	const mapHeight = 300;
	const DOT_R = 3;
	const MIN_DIST = DOT_R * 2;

	const projection = geoConicConformal()
		.parallels([49, 77])
		.rotate([96, 0])
		.center([5, 63])
		.scale(400)
		.translate([width / 2, mapHeight / 2]);

	const pathGen = geoPath().projection(projection);

	const provinces = provincesData.features.map(f => ({ d: pathGen(f) }));

	$: ecozones = ecozonesData.features
		.filter(f => HIGHLIGHTED_ECOZONES.includes(f.properties.ZONE_NAME))
		.map(f => ({
			d: pathGen(f),
			name: normZone(f.properties.ZONE_NAME),
			fill: ecozoneColours[normZone(f.properties.ZONE_NAME)] ?? '#ccc',
		}));

	function resolveCollisions(pts) {
		const result = pts.map(p => ({ ...p }));
		for (let iter = 0; iter < 80; iter++) {
			for (let i = 0; i < result.length; i++) {
				for (let j = i + 1; j < result.length; j++) {
					const dx = result[j].x - result[i].x;
					const dy = result[j].y - result[i].y;
					const dist = Math.sqrt(dx * dx + dy * dy);
					if (dist < MIN_DIST && dist > 0) {
						const push = (MIN_DIST - dist) / 2;
						result[i].x -= (dx / dist) * push;
						result[i].y -= (dy / dist) * push;
						result[j].x += (dx / dist) * push;
						result[j].y += (dy / dist) * push;
					}
				}
			}
		}
		return result;
	}

	const dots = resolveCollisions(
		cities.map(c => {
			const [x, y] = projection(c.coords);
			return { name: c.name, province: c.province, x, y };
		})
	);
</script>

<div class="map-container">
	<svg viewBox="0 0 {width} {mapHeight}" preserveAspectRatio="xMidYMid meet">
		<g class="provinces">
			{#each provinces as p}
				<path d={p.d} fill="#d8d8d8" stroke="#fff" stroke-width="0.05" />
			{/each}
		</g>
		<g class="ecozones">
			{#each ecozones as e}
				<path d={e.d} fill={e.fill} stroke="#fff" stroke-width="0.5" opacity="0.85" />
			{/each}
		</g>
		<g class="province-borders">
			{#each provinces as p}
				<path d={p.d} fill="none" stroke="#fff" stroke-width="0.75" />
			{/each}
		</g>
		<g class="cities">
			{#each dots as dot}
				<circle cx={dot.x} cy={dot.y} r={DOT_R} fill="#1a4a1a" stroke="#fff" stroke-width="1.5" opacity="0.9">
					<title>{dot.name}, {dot.province}</title>
				</circle>
			{/each}
		</g>
	</svg>

	<div class="legend">
		<div><a class="legend-title" href="https://en.wikipedia.org/wiki/Ecozones_of_Canada" target="_blank" rel="noopener noreferrer">Ecozone</a></div>
		{#each HIGHLIGHTED_ECOZONES as name}
			{@const display = normZone(name)}
			<div class="legend-item">
				<span class="swatch" style="background: {ecozoneColours[display] ?? '#ccc'}"></span>
				<span class="legend-label">{display}</span>
			</div>
		{/each}
		<div class="legend-item legend-item--city">
			<span class="swatch swatch--city"></span>
			<span class="legend-label">City in this study with street tree data</span>
		</div>
	</div>
</div>

<style>

	.map-container {
		width: 100%;
	}

	svg {
		width: 100%;
		height: auto;
		display: block;
	}

	.legend {
		display: flex;
		flex-direction: column;
		gap: 4px;
		padding: 6px 0 0;
		margin-top: -40px;
		margin-left: 90px;
		width: calc(100% - 90px);
	}

	.legend-item {
		display: flex;
		align-items: center;
		gap: 3px;
	}

	.swatch {
		width: 12px;
		height: 12px;
		border-radius: 2px;
		flex-shrink: 0;
		opacity: 0.9;
	}

	.legend-title {
		font-family: OpenSans, sans-serif;
		font-size: 0.72rem;
		font-weight: 600;
		color: #444;
		text-decoration: underline;
		text-underline-offset: 2px;
		margin-bottom: 2px;
		display: inline;
		transition: opacity 0.15s ease;
	}

	.legend-title:hover {
		opacity: 0.6;
	}

	.legend-label {
		font-family: OpenSans, sans-serif;
		font-size: 0.72rem;
		color: #444;
	}

	.legend-item--city {
		margin-top: 4px;
	}

	.swatch--city {
		background: #1a4a1a;
		border-radius: 50%;
		width: 8px;
		height: 8px;
		opacity: 0.9;
	}
</style>
