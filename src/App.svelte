<script>
	import { onMount } from "svelte";
	import { scaleLinear, scaleOrdinal } from "d3-scale";
	import { schemeCategory10 } from "d3-scale-chromatic";

	export const myName = "REPLACE_WITH_YOUR_NAME";

	let data;
	let popularGenres;
	
	let xScale;
	let yScale;
	let xScaleTicks;
	let yScaleTicks;
	let colorScale;

	const canvasSize = 400;
	const leftPadding = 50;
	const topPadding = 20;
	
	onMount(async () => {
		const fetched = await fetch("/static/movies.json");
		let rawData = (await fetched.json());
		console.log("rawData", rawData);

		// Remove null values
		data = rawData.filter(record =>
			record["IMDB Rating"] != null && 
			record["Rotten Tomatoes Rating"] != null &&
			record["Major Genre"] != null);
		console.log("data", data);

		// Show only popular genres
		let countByGenre = groupByCount(data, "Major Genre");
		console.log("countByGenre", countByGenre);

		// A genre is popular if there exist >5% of movies
		popularGenres = Object.keys(countByGenre).filter(key =>
			countByGenre[key] >= data.length * 0.05);
		console.log("popularGenres", popularGenres);

		data = data.filter(record =>
			popularGenres.includes(record["Major Genre"]));
		console.log("data.length", data.length);

		
		// Axis scales
		xScale = scaleLinear()
			.domain([
				0, 
				Math.max(...data.map(record => record["Rotten Tomatoes Rating"]))
			])
			.range([0, canvasSize])
		yScale = scaleLinear()
			.domain([
				Math.min(...data.map(record => record["IMDB Rating"])), 
				Math.max(...data.map(record => record["IMDB Rating"]))])
			.range([canvasSize, 0]);
			
		xScaleTicks = xScale.ticks(10);
		yScaleTicks = yScale.ticks(10);

		// Encode genre as color
		// https://github.com/d3/d3-scale-chromatic/blob/main/README.md#schemeCategory10
		colorScale = scaleOrdinal(schemeCategory10)
			.domain(data.map(row => row["Major Genre"]));
	});


	// Group by 
	const groupByCount = (items, key) => 
		items.reduce((result, item) => {
			result[item[key]] = result[item[key]] + 1 || 0;
			return result;
		}, {});
</script>

<main>
	<h1>In-Class Acitivity by {myName}!</h1>

	<svg id="visualization-activity">
		{#if data !== undefined}	
			<g id="scatterplotData" transform="translate({leftPadding}, {topPadding})">
				{#each data as record}
					<circle 
						id="datapoint-{record.id}"
						class="point"
						cx={xScale(record["Rotten Tomatoes Rating"])}
						cy={yScale(record["IMDB Rating"])} 
						r="3"
						style="fill: {colorScale(record["Major Genre"])}"
					/>
				{/each}
			</g>
			
			<g id="axes" transform="translate({leftPadding}, {topPadding})">
				<line class="axis" y2={canvasSize} />
				<line class="axis" x2={canvasSize} y1={canvasSize} y2={canvasSize} />

				<g id="xAxisTicks" transform="translate(0, {canvasSize})">
					{#each xScaleTicks as tick}
						<g transform="translate({xScale(tick)}, 0)">
							<line class="axis" y2="5" />
							<text y="14">{tick}</text>
						</g>
					{/each}
					<text x={canvasSize * 0.5} y="35">
						Rotten Tomatoes Rating
					</text>
				</g>
				<g id="yAxisTicks" transform="translate(0, 0)">
					{#each yScaleTicks as tick}
						<g transform="translate(0, {yScale(tick)})">
							<line class="axis" x2="-5" />
							<text x="-14">{tick}</text>
						</g>
					{/each}
					<text transform="translate(-30, {canvasSize * 0.5}) rotate(-90)">
						IMDb Rating
					</text>
				</g>				
			</g>

			<g id="legend" transform="translate({canvasSize + leftPadding + 30}, 30)">
				{#each popularGenres as genre, index}
					<g class="legendItem" transform="translate(0, {index * 17})">
						<rect 
							width="10" height="10"
							style="fill: {colorScale(genre)}"
						/>
						<text x="15" y="9">{genre}</text>
					</g>
				{/each}
			</g>
		{/if}
	</svg>
</main>

<style>
	h1 {
		font-size: 1.5rem;
	}
	svg#visualization-activity {
		width: 600px;
		height: 600px;
	}

	circle.point {
		fill: gray;
		stroke-width: 0;
	}
	text {
		font-size: 0.8rem;
	}
	#axes text {
		font-size: 0.7rem;
		text-anchor: middle;
	}
	line.axis {
		stroke: gray;
	}
</style>
