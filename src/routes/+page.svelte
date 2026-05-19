<script>

	import "../assets/styles.css";

	import TitleVideo from "$lib/TitleVideo.svelte";
	import GraphicSingle from "$lib/GraphicSingle.svelte";
	import AuthorDate from "$lib/AuthorDate.svelte";
	import ImageSingle from "$lib/ImageSingle.svelte";

	
	import CanadaMap from "$lib/CanadaMap.svelte";
	import CityChart from "$lib/CityChart.svelte";
	import LorenzCurve from "$lib/LorenzCurve.svelte";
	import LeafRadar from "$lib/LeafRadar.svelte";
	import GiniChart from "$lib/GiniChart.svelte";
	import RegressionWaffle from "$lib/RegressionWaffle.svelte";
	import Footnote from "$lib/Footnote.svelte";
	import Footnotes from "$lib/Footnotes.svelte";
	import { createFootnoteStore } from "$lib/footnoteUtils";
	const footnoteStore = createFootnoteStore();
	const { footnotes, addFootnote } = footnoteStore;
	const fns = [
		' N.A. Richards, “Diversity and stability in a street tree population,” Urban Ecology 7, no. 2(1983): 159–171, <a href="https://doi.org/10.1016/0304-4009(83)90034-7" target="_blank">DOI</a>; and Andrew A. Millward and Senna Sabir, “Structure of a forested urban park: Implications for strategic management,” Journal of Environmental Management 91, no. 11 (2010): 2215–2224, <a href="https://doi.org/10.1016/j.jenvman.2010.06.006" target="_blank">DOI</a>.',
		'Elliott Volin, et al., “Assessing macro-scale patterns in urban tree canopy and inequality,” Urban Forestry and Urban Greening, 55, (2020): 126818, <a href="https://doi.org/10.1016/j.ufug.2020.126818" target="_blank">DOI</a>, and Dexter H. Locke, et al., “Residential housing segregation and urban tree canopy in 37 US Cities,” Urban Sustainability 1, n.15 (2021), <a href="https://doi.org/10.1038/s42949-021-00022-0" target="_blank">DOI</a>',
		'Statistics Canada. (2023). The Canadian Index of Multiple Deprivation, 2021. Statistics Canada Catalogue no. 45‑20-0001.'

	];

	const ecozoneColours = {
		'Atlantic Maritime':  '#c0de90',
		'Prairie':            '#d4c97a',
		'Pacific Maritime':   '#85c9bf',
		'Montane Cordillera': '#b5bda8',
		'Mixedwood Plain':    '#93bd84',
	};

	const credits = [
		{
			role: "Research",
			people: [
				{ name: "Tenley Conway" },
				{ name: "Alexander Martin" },
			],
		},
		{
			role: "Writing",
			people: [
				{ name: "Tenley Conway" }, 
				{ name: "Alexander Martin" },
				{ name: "Serene Tan" }
			],
		},
		{
			role: "Data visualization",
			people: [
				{ name: "Jeff Allen", 
				// url: "https://jamaps.github.io" 
			},
				{ name: "Isabeaux Graham"} 
			],
		},
	];


</script>


<svelte:head>

	<!-- <title>Kensington Market Commercial Change | School of Cities</title>

	<meta name="description" content="Mapping the changing commercial landscape of Kensington Market.">
	<meta name="author" content="Farzaneh Hemmasi, Emily Herzman, Scott McCallum">

	<meta property="og:title" content="Commercial Change in Kensington Market" />
	<meta name="og:description" content="Mapping the changing commercial landscape of Kensington Market." />
	<meta property="og:type" content="website" />
	<meta property="og:url" content="https://schoolofcities.github.io/kmclt" />
	<meta property="og:image" content="https://schoolofcities.github.io/kensington-market/archives_km_4.jpg" />
	<meta property="og:locale" content="en_CA">

	<meta name="twitter:card" content="summary_large_image" />
	<meta name="twitter:site" content="https://schoolofcities.github.io/kmclt/" />
	<meta name="twitter:creator" content="@UofTCities" />
	<meta name="twitter:title" content="Commercial Change in Kensington Market" />
	<meta name="twitter:description" content="Mapping the changing commercial landscape of Kensington Market." />
	<meta name="twitter:image" content="https://schoolofcities.github.io/kensington-market/archives_km_4.jpg" /> -->


</svelte:head>


<main>

	<TitleVideo
		title="The trees lining our streets: diversity and equity of Canada’s street trees"
		subtitle=""
		video="./stock-video-surrey-bc.mp4"
		videoOpacity="0.9"
		videoSpeed="1.15"
		titleFontColour="white"
		logoType="White"
	/>

	<div class="text">
		<div
			class="caption-container"
			style="margin-top: -60px; margin-bottom: 70px; padding-left: 0px; opacity: 0.4;"
		>
			<p>
				<span class="caption-text"
					><span class="caption-source"
						>Video source: <a
							href=""
							target="_blank"
							>Storyblocks</a
						></span
					></span
				>
			</p>
		</div>

		<AuthorDate
			{credits}
			date="May 2026"
		/>

		<p>
			Cities across Canada are working to grow their urban forest based on the range of environmental, social, and health benefits provided by trees. These benefits include cooling during hot summer months, reduction in local flooding vulnerability, fostering community cohesion and a sense of place, and contributing to positive mental and physical health outcomes. The number of trees, their size, and species composition influence the type and extent of benefits created. However, most benefits are localized: experienced in the immediate area adjacent to the trees. As a result, an even spatial distribution of trees is important to ensure that all urban residents are receiving those benefits.
		</p>

		<p>
			Drawing on a recently created repository of municipal street tree inventories across Canada, Alexander Martin (UofT Geography MSc, 2025) and Professor Tenley Conway examined the density, size and species of street tree assemblages in 32 Canadian cities to understand if trees are spatially distributed evenly and if distributions of street trees unfairly burden marginalized populations. They calculated the Gini index, a measure of inequality, to determine if trees are evenly distributed. They then conducted regression analyses using variables from the Canadian Index of Multiple Deprivation to determine if marginalized communities are inequitably associated with fewer, smaller, or less diverse trees in their neighbourhoods.  
		</p>

		<p>
			There are substantial inequalities in street tree distribution in the 32 cities, and also inequities associated with population density and four measures of socio-economic deprivation. However, specific street tree inequities varied between cities, highlighting the importance of understanding local context. Cities should address inequities, ensuring all residents equally benefit from street trees. Steps toward just street tree assemblages include planning for adequate space for large trees to thrive, focusing on diversifying species in neighbourhoods with high populations, supporting street tree planting and stewardship in marginalized communities, and using tools like the Gini index to track progress over time. 
		</p>

		<div class="oufp-section">
			<h3>Open Urban Forests Project (OUFP) repository </h3>

			<p>
				The Canadian Institute of Forestry through their <a href="https://www.cif-ifc.org/get-engaged/open-urban-forests/" target="_blank">Open Urban Forests Project</a> (OUFP) created a repository of municipalities’ tree inventories in 2024. The repository represents the first effort of its kind in Canada to support learning and informed decision-making about urban trees.
			</p>

			<p>
				Conway and Martin used the repository data to explore 32 cities’ street tree populations. Street trees are often the most visible trees in urban areas. These trees face challenging growing conditions, like high levels of vehicle pollution and limited growing space, that may restrict their growth and limit the type of species that can survive. In most cases, street trees are the responsibility of municipal governments, meaning management by municipal urban foresters and arborists directly impacts the planting and health of street trees. These trees and their attributes are also most frequently recorded in municipal tree inventories.
			</p>

			<div class="map-float">
				<CanadaMap {ecozoneColours} />
			</div>
		</div>

	</div>


	<div class="text">

		<h3>Comparing Canada’s street trees populations</h3>
	
	<!-- </div> -->

	<CityChart {ecozoneColours} />

	<!-- <div class="text"> -->

		<p>
			Canada’s 32 cities contain over 3 million street trees, with the number of trees ranging from 9,787 trees in New Westminster, BC (population 78,916 in 2016) to 652,547 trees in Toronto (population 2,731,571 in 2016). All municipalities have more small trees and fewer large trees than recommended size class benchmarks, which may be a result of smaller stature trees being planted or a dominance of young trees that have not yet reached their mature height<Footnote id={addFootnote(fns[0])} />.
		</p>

		<p>
			Collectively, there is a diversity of species planted along Canada's city streets, with 726 street tree species across the 32 study cities. Vancouver has the most species (373), while Regina has the fewest (29). In most cities a few species dominate, with just four species representing 34% of the trees in the dataset. This is even more pronounced in the six Prairie cities in the dataset, with green ash and American elm comprising more than half of their street trees. 
		</p>

		<p>
			The percentage of each municipality’s street trees that are native species is highly variable, ranging from just under 5% in Maple Ridge BC to 89% in Regina. While warmer Canadian municipalities have a larger number of species, they also tend to have a smaller percentage of native street trees. Additionally, there are notable differences between nearby cities. For example, in Ontario, St. Catherine has 28% native street trees and Welland has 48%, but they are only 25 km apart.  This suggests that management decisions, in addition to climate and other biophysical factors, are important influences on what species are present in a given city.  
		</p>

		

	</div>


	<LeafRadar cityA="Winnipeg" cityB="Montreal" {ecozoneColours} />


	<div class="text">

		<h3>
			Street tree equality
		</h3>

		<p>
			Street tree equality is reached when there is an even spatial distribution of trees, which can provide similar levels of benefits to all neighbourhoods in a city. Many Canadian municipalities are working to create more equal distributions of urban trees. 
		</p>

		<p>
			Conway and Martin set out to explore if street tree populations are equally distributed across each city. To measure equality, we used the Gini Index. Traditionally, the Gini Index has been used to measure income (in)equality, but in recent years it has been applied to understand inequalities in urban tree distributions. Index values less than 0.2 are generally considered to represent equal distributions and those over 0.5 indicate significant disparity, however, benchmarks specific to urban forests have not been identified.  
		</p>

		<div class="oufp-section">
			<p>
				The Gini index can also be depicted graphically through Lorenz curves, which show the proportion of trees associated with the cumulative share of people. Perfect equality is represented by the straight <span style="font-family: OpenSansBold; font-weight: normal; font-size: 16px;">y=x</span>. Curves further from the line of equality indicate greater inequality.
			</p>
			<div class="map-float">
				<LorenzCurve />
			</div>
		</div>

		<p>
			The curves show relatively high levels of inequality for neighbourhood tree count (median Gini = 0.401), with Fredericton, NB having the most equal distribution (Gini = 0.259) and Maple Ridge, BC (Gini = 0.681) having the least. The distribution of neighbourhood basal area, a measure reflecting the cumulative volume of trees associated with the magnitude of benefits produced, showed a median Gini value of 0.469, with highest equality in Windsor (Gini = 0.324) and least equality in Lethbridge (0.709). 
		</p>
	
		<GiniChart {ecozoneColours} />
	

		<h3>
			Street tree equality
		</h3>

		<p>
			Are unequal distributions of street trees unfairly burdening marginalized populations? In other words, are the uneven distributions creating inequities between neighbourhoods with different socio-demographics? Such inequities are common in many US cities<Footnote id={addFootnote(fns[1])} />, with lower income households and racial minorities often living in neighbourhoods with relatively low tree canopy cover.  
		</p>

		<p>
			Conway and Martin explore the possibility of inequities using the Canadian Index of Multiple Deprivation, developed by Statistics Canada<Footnote id={addFootnote(fns[2])} />. The index includes four measures:
		</p>

		<ul>
			<li><strong>Residential instability</strong>, which measures fluctuations in neighborhood residents,</li>
			<li><strong>Economic dependency</strong>, which addresses a population’s reliance on the workforce,</li>
			<li><strong>Ethno-cultural composition</strong>, which measures the immigrant and visible minority populations, and</li>
			<li><strong>Situational vulnerability</strong>, which describes additional demographic characteristics such as education levels and home value.</li>
		</ul>


	</div>



	<div class="text">
		<RegressionWaffle />
	</div>

	<div class="insights-grid">
		<div class="text">
			<h3>Insights</h3>

			<div class="callout-list">
				<div class="callout-item">
					<p><strong>Street tree populations reflect a complex landscape of inequities across Canadian cities.</strong> There are specific relationships between deprivation index measures and tree density, tree size, and species richness varying between cities. The findings highlight street tree inequities cannot be generalized across cities, and municipal responses must be tailored to local realities.</p>
				</div>
				<div class="callout-item">
					<p><strong>Neighborhoods with higher population density regularly have a higher density of street trees but a lower diversity of species.</strong> This may reflect how municipal urban foresters prioritize planting along streets in denser residential neighbourhoods, where other greenspaces are more limited, but relying on a small number of species that are known to survive in stressful conditions associated with constrained planting sites.</p>
				</div>
				<div class="callout-item">
					<p><strong>Residential instability is frequently negatively associated with tree density, median DBH, and basal area.</strong> This may be because in areas with lower instability – generally longer-term, wealthier residents living in owner-occupied, single-family houses – residents are requesting more street tree plantings and performing more consistent stewardship to helping trees thrive.</p>
				</div>
				<div class="callout-item">
					<p><strong>Situational vulnerability was often positively or not significantly associated with tree density and our two measures of tree size.</strong> This differs from the negative associations often observed for residential instability and ethnocultural composition. This distinction may be due to neighbourhoods with higher situational vulnerability containing older housing and lower levels of recent re-development, which results in larger trees.</p>
				</div>
			</div>
		</div>

		<div class="text">
			<h3>Call to Action</h3>

			<div class="callout-list callout-list--action">
				<div class="callout-item">
					<p><strong>Cities need to plant and maintain large trees.</strong> Many cities already prioritize protection and planting of shade trees, which can be challenging due to stressful roadside conditions and infrastructure conflicts. However, the overabundance of smaller street trees across the dataset suggests that more work is needed to meet size class benchmarks that are associated with increased and sustained street tree benefits. Re-designing spaces for larger stature trees, including adopting soil volume standards – like the City of Toronto has within the Toronto Green Standard – is needed to ensure adequate space for root growth, supporting larger tree growth.</p>
				</div>
				<div class="callout-item">
					<p><strong>Cities should plant a diversity of species.</strong> Increased species diversity will ensure a variety of benefits are produced, while also reducing a city’s vulnerability to pests and disease. This is particularly important in prairie cities, which are dominated by green ash and American elm. Both species are vulnerable to pests (emerald ash borer and Dutch elm disease) that have decimated ash and elm populations elsewhere in North America.</p>
				</div>
				<div class="callout-item">
					<p><strong>Monitor equality of spatial distributions.</strong> Adopting metrics like the Gini Index can quantify current unevenness and allow cities to track changes over time. Many Canadian cities have invested in tree planting initiatives, but it remains unclear if new plantings have increased or decreased unequal distributions.</p>
				</div>
				<div class="callout-item">
					<p><strong>Investigate local drivers of inequity.</strong> The analysis found that inequities in street trees are common, but the specific socio-demographic relationship varied between cities. This suggests that local histories and current conditions matter, with efforts to understand city-specific inequities needed to develop strategies to grow more equitable distributions.</p>
				</div>
			</div>
		</div>
	</div>


	<div class="text">
		<h3>Next Steps</h3>
		<p>
			While identifying uneven distributions and inequities within street trees or across the urban forest is an important first step, simply planting more trees in underserved areas will not necessarily achieve a just outcome. Rather, recognitional justice concerns need to be addressed, which acknowledges that different communities have diverse needs and expectations. With funding from the School of Cities Catalyst Grant, Tenley Conway and PhD student Isaac Mendita are working in partnership with the City of Toronto Urban Forestry division and local tree planting NGOs to engage in community mapping in two marginalized communities identified as having below average tree cover. Through this project we hope to learn where community members do and do not want trees planted, along with desired tree characteristics. Findings will be shared with community leaders and research partners to develop an approach to incorporate communities’ needs and help guide future tree planting efforts.
		</p>
	</div>

	<div class="text">
		<h3>Related publications and resources </h3>
		<br>
		<ul class="references">
			<li>Martin, A.J.F., Fleming, A. &amp; Conway, T.M. Distributional inequities in tree density, size, and species diversity in 32 Canadian cities. <em>npj Urban Sustainability</em> 5, 18 (2025). <a href="https://doi.org/10.1038/s42949-025-00210-2" target="_blank" rel="noopener noreferrer">DOI</a></li>
			<li>Martin, A.J.F. &amp; T.M. Conway. Using the Gini Index to quantify urban green inequality: A systematic review and recommended reporting standards. <em>Landscape and Urban Planning</em> 254, 105231 (2025). <a href="https://doi.org/10.1016/j.landurbplan.2024.105231" target="_blank" rel="noopener noreferrer">DOI</a></li>
			<li>Martin, A.J.F., Olson, L.G., Bacon, E.S. et al. A biogeographical analysis of taxonomic diversity and native species dominance in 32 Canadian street tree populations. <em>Urban Ecosystems</em> 28, 158 (2025). <a href="https://doi.org/10.1007/s11252-025-01764-w" target="_blank" rel="noopener noreferrer">DOI</a></li>
			<li>Martin, A.J.F., Olson, L.G., Bacon, E.S. et al. The structural diversity of Canada's urban forests: A national summary and an analysis of street tree size across ecozones, city sizes, and within cities. <em>Journal of Forestry</em> 123, 217–240 (2025). <a href="https://doi.org/10.1007/s44392-025-00011-6" target="_blank" rel="noopener noreferrer">DOI</a></li>
			<li>Canadian Institute of Forestry/Institut forestier du Canada. Open Urban Forests. <a href="https://www.cif-ifc.org/get-engaged/open-urban-forests/" target="_blank" rel="noopener noreferrer">Link</a></li>
		</ul>
	</div>

	<Footnotes {footnotes} />

	<br><br><br><br>
		

</main>




<style>
	.references {
		font-size: 0.82rem;
		line-height: 1.6;
		color: #555;
		padding-left: 1.2rem;
		display: flex;
		flex-direction: column;
		gap: 0.6rem;
	}

	.references a {
		color: #000;
		word-break: break-word;
	}

	.references a:hover {
		color: var(--brandMedGreen);
	}

	.insights-grid {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 0 2rem;
		max-width: 1080px;
		margin: 0 auto;
	}

	.insights-grid .text {
		max-width: none;
		margin: 0;
	}

	@media (max-width: 1080px) {
		.insights-grid {
			grid-template-columns: 1fr;
		}
		.insights-grid .text {
			max-width: 720px;
			margin: 0 auto;
		}
	}

	.callout-list {
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
		margin: 1rem 0 2rem;
	}

	.callout-item {
		border-left: 3px solid var(--brandLightGreen);
		padding: 0.6rem 0.9rem;
		background: rgba(141, 191, 46, 0.07);
		border-radius: 0 4px 4px 0;
	}

	.callout-list--action .callout-item {
		border-left-color: var(--brandMedGreen);
		background: rgba(0, 161, 137, 0.07);
	}

	.callout-item p {
		margin: 0;
		font-size: 18px;
		line-height: 29px;
		color: #000000;
	}

	@media (max-width: 600px) {
		.callout-item p {
			font-size: 16px;
			line-height: 24px;
		}

		.references {
			font-size: 0.75rem;
			line-height: 1.5;
		}
	}

	.oufp-section {
		position: relative;
	}

	.map-float {
		position: absolute;
		top: 0;
		left: calc(100% - 20px);
		width: 360px;
	}

	@media (max-width: 1300px) {
		.map-float {
			position: static;
			width: 360px !important;
			margin: 1.5rem auto;
		}
	}

	@media (max-width: 500px) {
		.map-float {
			width: 300px !important;
			max-width: 100%;
		}
	}
</style>
