<script>
	import mapboxgl from 'mapbox-gl';
	import { onMount } from "svelte";
	import { afterUpdate } from 'svelte';
	import * as d3 from "d3";
  
	let lng, lat, zoom;
	lng = -117.1611;
	lat = 32.7157;
	zoom = 9;

	let isHidden = true;
	let selectedCrime = "ASSAULT";
	let selectedMonth = "02"; 
	let selectedYear = "2024";
  
	let csvData;
  
	// Colors of categories
	const crimeColors = {
		"THEFT/LARCENY": "#3498db",     // Blue
		"DUI": "#e74c3c",               // Red
		"ROBBERY": "#e67e22",           // Orange
		"BURGLARY": "#9b59b6",          // Purple
		"VANDALISM": "#2ecc71",         // Green
		"FRAUD": "#f1c40f",             // Yellow
		"ASSAULT": "#e91e63",           // Pink
		"VEHICLE BREAK-IN/THEFT": "#00bcd4", // Cyan
		"DRUGS/ALCOHOL VIOLATIONS": "#d35400", // Brown
		"ARSON": "#1abc9c",             // Teal
		"MOTOR VEHICLE THEFT": "#303f9f",  // Indigo
		"SEX CRIMES": "#800000",        // Maroon
		"WEAPONS": "#95a5a6",           // Gray
		"HOMICIDE": "#000000",          // Black
};


  
	async function fetchData() {
		try {
			let csv_data = await d3.csv('ARJIS_Public_Crime_Data_w__Day_of_Week_20240215.csv');
			return csv_data
			.filter(row => {
          		return row.geometry && row.geometry.match(/POINT \((-?\d+\.\d+) (-?\d+\.\d+)\)/);
        	})
			.map(row => {
				const matches = row.geometry.match(/POINT \(([-0-9.]+) ([-0-9.]+)\)/);
				if (matches && matches.length === 3) {
					const [_, longitude, latitude] = matches;
					row.coordinates = [parseFloat(longitude), parseFloat(latitude)];
				}
				return row;
			});
	  } catch (error) {
		console.error('Error loading CSV:', error);
	  }
	}


	async function updatemap(){


		mapboxgl.accessToken = "pk.eyJ1IjoiYmx3MDAyIiwiYSI6ImNsc21kam9xcTBtaHQycXBkNGVudTZmNXAifQ.Zxv8Wui2RBVxFDRB0mbvMw";
			
		const map = new mapboxgl.Map({
			container: "map",
			style: "mapbox://styles/mapbox/streets-v12",
			center: [lng, lat],
			zoom: zoom, // starting zoom level
			minZoom: 8,
			maxZoom: 20,
		});	

		const filteredData = csvData.filter(point => {
			const crimeDate = new Date(point.Date);
			const crimeMonth = crimeDate.getMonth() + 1; // Months are zero-indexed
			const crimeYear = crimeDate.getFullYear().toString();

		return (
			point.Crime_Category === selectedCrime &&
			crimeMonth.toString().padStart(2, '0') === selectedMonth &&
			crimeYear === selectedYear
      );
    });


		if ((Number( selectedYear ) === 2023) & (Number(selectedMonth) < 8)){
			isHidden = false;
		}
		else if ((Number(selectedYear) === 2024) & (Number(selectedMonth) > 2)){
			isHidden = false;
		}
		else{
			isHidden = true;
		}



		filteredData.forEach(point => {
		const marker = new mapboxgl.Marker({
			color: crimeColors[point.Crime_Category] || 'gray',
		})
			.setLngLat(point.coordinates)
			.setPopup(new mapboxgl.Popup().setHTML(`
			<h3>${point.Agency}</h3>
			<p><strong>Type:</strong> ${point.Crime_Category}</p>
			<p><strong>Date:</strong> ${point.Date}</p>
			<p><strong>Description:</strong> ${point.Charge_Description}</p>
			`))
			.addTo(map);
		});

		map.on('click', (e) => {
			console.log('Map clicked at:', e.lngLat);
		});
	};

  
	// Use await within an async context
	onMount(async () => {
	  // Assign the result of fetchData to csvData

		// Initial data load
		try {
			csvData = await fetchData();
			console.log(d3.csvData);
			updatemap();
		} catch (error) {
			console.error('Error loading CSV data:', error);
		}
		
	});

	afterUpdate(() => {
    	updatemap();
  	});

  </script>
  

  
  <main>
	<div id = "map"></div>
	<!-- <div id="map"></div> -->
	{#if !isHidden}
		<div id = "hidden">
			There is No Data For This Time Period
		</div>
	{/if}

	<div id="ui-elements">

	  <label>
		Crime:
		<select bind:value={selectedCrime}>
		  {#each Object.keys(crimeColors) as crimeCategory}
			<option value={crimeCategory}>{crimeCategory}</option>
		  {/each}
		</select>
	  </label>
  
	  <label>
		Month:
		<select bind:value={selectedMonth}>
		  {#each Array.from({ length: 12 }, (_, i) => (i + 1).toString().padStart(2, '0')) as month}
			<option value={month}>{month}</option>
		  {/each}
		</select>
	  </label>
  
	  <label>
		Year:
		<select bind:value={selectedYear}>
		  {#each Array.from({ length: 2 }, (_, i) => (2024 - i).toString()) as year}
			<option value={year}>{year}</option>
		  {/each}
		</select>
	  </label>
	</div>

	<div id = "writeup" >
		<h2>Write Up</h2>
		<p>
			- A rationale for your design decisions. How did you choose your particular visual encodings and interaction techniques? What alternatives did you consider and how did you arrive at your ultimate choices?
		</p>
		<p>	
			- An overview of your development process. Describe how the work was split among the team members. Include a commentary on the development process, including answers to the following questions: Roughly how much time did you spend developing your application (in people-hours)? What aspects took the most time?
		</p>
	</div>
  </main>
  
  <style>
	body {
		margin: 0;
		padding: 0;
	}

	#map {
	position: absolute;
	top: 0;
	bottom: 0;
	width: 100%;
	height: 100%;
	}


	input {
		display: inline-block;
		width: 100%;
		position: relative;
		margin: 0;
		cursor: pointer;
	}

	#ui-elements {
		font-size: 0.9em;
		background-color: rgba(255, 255, 255, 0.4);
		position: absolute;
		min-width:250px;
		width: 15%;
		top: 10px;
		right: 10px;
		padding: 10px;
		z-index: 3;
	}

	#hidden {
		font-size: 0.9em;
		background-color: rgba(255, 255, 255, 0.4);
		position: absolute;
		margin : auto; 
		min-width:250px;
		width: 15%;
		top: 10px;
		padding: 10px;
		z-index: 3;	
	}

	#writeup {
		font-size: 0.9em;
		background-color: rgba(255, 255, 255, 0.4);
		position: absolute;
		margin: auto;
		min-width: 250px;
		width: 15%;
		bottom: 10px; /* Adjust this value as needed */
		left: 10px;   /* Adjust this value as needed */
		padding: 10px;
		z-index: 3;
	}
  </style>

  