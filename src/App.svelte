
<script>
	import { onMount } from "svelte";
	import { scaleLinear } from "d3-scale";
	import { scaleOrdinal } from "d3-scale";
	import { current_component, each } from "svelte/internal";
	import { schemeCategory10 } from "d3-scale-chromatic";



	let xScale;
	let xScale_new;
	let yScale;
	let yScale_new;
	let instances;
	let selected_point = null;
	let colorScale
	const canvasSize = 350;
	let xScaleTicks = [];
	let yScaleTicks;

	const numClasses = 10;
	const numBins = 10;
	let binsByClasses = [];
	let data = [];

	onMount(async () => {
		const fetched = await fetch("static/prediction_results.json");
		instances = (await fetched.json()).test_instances;
		console.log("raw data",instances);
		selected_point = instances[0];
		colorScale = schemeCategory10;



		// Transform data
		for (let k = 0; k < numClasses; ++k) {
			let binsForClass = [];
			for (let b = 0; b < numBins; ++b) {
				binsForClass.push({"class": k, "binNo": b, "instances": []});
			}
			binsByClasses.push({"class": k, "bins": binsForClass});
		}

		
		instances.forEach(instance => {
			data.push(instance)

		});

		data.forEach(record => {
			record.selected = false;
			record.show = false;
		});


		// Axis scales
		xScale = scaleLinear()
			.domain([
				Math.min(...data.map(record => record["projection"][0])),
				Math.max(...data.map(record => record["projection"][0]))
			])
			.range([0, canvasSize]);

		yScale = scaleLinear()
			.domain([
				Math.min(...data.map(record => record["projection"][1])), 
				Math.max(...data.map(record => record["projection"][1]))
			])
			.range([0, canvasSize]);

		xScale_new = scaleLinear()
		.domain([
			0,
			1
		])
		.range([0, 900]);


		yScale_new = scaleLinear()
		.domain([
			0,
			1
		])
		.range([0, 500]);
	

	selected_point = data[0]

	xScaleTicks = [0.0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0]
	console.log(xScaleTicks)
	
});

	function getColor(label){
		return colorScale[parseInt(label)]
	};

	function highlight_bin(bin){
		data.forEach(record => {
			if (record.true_label == bin){
				record.selected = true;
			}
		});
	}
	function un_highlight(bin){
		data.forEach(record => {
			record.selected = false;
		});
	}

	

	

</script>

<main>
	<h1>Visual Analytics HW 3 Alex Barajas
	</h1>

	<div id="container">
		<div id="sidebar" style="width: 450px;">
			<div id="projection-view" class="view-panel">
				<div class="view-title">Projection View</div>
				<svg style="width: 400; height: 400">
					<g id="scatterplotData">
						{#each data as record}
							<circle
							id="datapoint-{record.id}"
							class="point {record.selected == true ? "selected" : "cirlce" }"
							cx = {xScale( record.projection[0]) + 25}
							cy = {yScale( record.projection[1]) + 25}
							r=3
							style="fill: {getColor(record["true_label"])};
									stroke: {getColor(record["predicted_label"])};
									stroke-width: {2.2};
									stroke-opacity: {.5};
									fill-opacity: {.5};"
							
							on:click={()=>{
								if (record.selected != true){
									selected_point = record
									highlight_bin(record.true_label);
								}
								else{
									record.selected = false;
									un_highlight(record.true_label);
								}
					
							}}
							/> 
						{/each}
					</g>

					
				</svg>
			</div>
			<div id="selected-image-view" class="view-panel">
				<div class="view-title">Selected Image</div>
				<div id="selected-image-view-content"> <!-- display selected image -->	
					<div id="the whole thing" style="height:100%; width:100%; overflow: hidden;">
						<div id="content" style="float: left; width:50%">
							{#if selected_point !== null}
								<img src={"/static/images/" + selected_point.filename} alt="" width="80" height="80"/>								
							{/if}
						
						</div>

						<div id="rightThing" style="float: left; width:25%;">
								{#if selected_point !== null}
									ID: {selected_point.id} <br>
									Labeled as {selected_point.true_label} <br>
									Predicted as {selected_point.predicted_label}<br>
								{/if}
			
						</div>
					</div>

					
				</div>
			</div>
		</div>

		<div id="main-section" style="width: 1000px;">
			<div id="score-distributions-view" class="view-panel">
				<div class="view-title">Score Distributions</div>
				<svg width="2000" height="600">
					<line class="axis" x1= {50} x2={950} />
					<g id="xAxisTicks">
						{#each xScaleTicks as tick, i}
							<g transform="translate({50 + xScale_new(tick)}, 0)">
								<line class="axis" y2="-5" />
								<text x= {-10} y="20">{tick}</text>
								
							</g>
						{/each}

						{#each xScaleTicks as tick, i} 
							<g>
								<text x = {0}, y = {yScale_new(tick) + 100}>Class </text>
								<text x = {40}, y = {yScale_new(tick) + 100}, style="fill: {getColor(i)};"> {i} </text>
								<text x = {0}, y = {yScale_new(tick) + 112}>Labeled as {i}</text>
								<text x = {0}, y = {yScale_new(tick) + 124}>Predicted as {i}</text>
								<line class="axis" y1 = {yScale_new(tick) + 100} y2 = {yScale_new(tick) +100} x1= {50} x2={950} />
								{#each xScaleTicks as tick_1}
									<g transform="translate({50 + xScale_new(tick_1)}, {yScale_new(tick) +100})">
										<line class="axis" y2="-5" />
									</g>
								{/each}

							</g>
						{/each}
					</g>
				</svg>
			</div>
		</div>
	</div>

</main>

<style>
	h1 {
		font-size: 1.3rem;
		font-weight: 500;
		margin-top: 0;
	}
	#container {
		display: flex;
	}
	#sidebar, #main-section {
		display: flex;
		flex-direction: column;
	}
	.view-panel {
		border: 2px solid #eee;
		margin-bottom: 15px;
		margin-right: 15px;
	}
	.view-title {
		background-color: #f3f3f3;
		font-size: 1.0rem;
		margin-bottom: 8px;
		padding: 3px 8px 5px 12px;
	}
	#selected-image-view-content {
		height: 100px;
		padding: 15px;
	}
	svg {
		margin: 5px;
	}
	circle.selected{
		stroke-width: 2.2;
		stroke: black;
		fill-opacity: 0.1;
	}
	line.axis {
		stroke: gray;
	}



</style>
