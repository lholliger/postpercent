<script>
	import { onMount } from 'svelte';
	import Chart from "chart.js"
	
	export let username;
    export let color;
	export let colors;

	let precise, forum;
	let ctx, ctx2, ctx3, ctx4, ctx5;
	let postoffset;

	async function getData()    {
		await fetch(`https://scratchdb.lefty.one/v3/forum/user/info/${username}/`)
        .then(res => res.json())
        .then(data => {
            forum = data;
			let value = [];
			let label = [];
			for (let i = 1; i < Object.keys(forum.counts).length - 1; i++)	{
				label.push(Object.entries(forum.counts)[i][0]);
				value.push(Object.entries(forum.counts)[i][1].count);
			}
			var chart = new Chart(ctx5, {
				type: 'pie',
				data: {
					datasets: [{
						data: value,
						backgroundColor: colors,
						label: 'Scratcher Posts'
					}],
					labels: label,
				},
				options: {
					responsive: true,
				},
			});
        })
		.catch(error => {
            console.error(error);
            throw new Error('Something went wrong D:');
        })
		await fetch(`https://scratchdb.lefty.one/v3/forum/user/graph/${username}/total?segment=month&range=3650`)
		.then(res => res.json())
        .then(data => {
            precise = data;
			let dategraph = [];
			let postgraph = [];
            let postgraph2 = [];
			for (let i = 0; i < precise.length - 1; i++)	{
				dategraph.push(`${new Date(precise[i].date).getMonth() + 1}/${new Date(precise[i].date).getDate()}/${new Date(precise[i].date).getFullYear()}`)
				postgraph.push(precise[i].value)
                if (i == 0) {
                    postgraph2.push(precise[i].value)
                } else  {
                    postgraph2.push(precise[i].value - precise[i-1].value)
                }
			}
			postgraph.length > 11 ? postoffset = postgraph[postgraph.length - 12] : postoffset = 0;
			var chart = new Chart(ctx, {
			type: "line",
			data: {
				labels: dategraph,
				datasets: [
				{
					label: `${username}`,
					backgroundColor: color,
					borderColor: color,
					data: postgraph,
					fill: false,
					tension: 0.05
				}
				]
			},
			options: {
				responsive: true,
				tooltips: {
					mode: 'index',
					intersect: false,
				},
				hover: {
					mode: 'nearest',
					intersect: true,
				}
			}
			});
            var chart = new Chart(ctx3, {
			type: "line",
			data: {
				labels: dategraph,
				datasets: [
				{
					label: `${username}`,
					backgroundColor: color,
					borderColor: color,
					data: postgraph2,
					fill: false,
					tension: 0.05
				}
				]
			},
			options: {
				responsive: true,
				tooltips: {
					mode: 'index',
					intersect: false,
				},
				hover: {
					mode: 'nearest',
					intersect: true,
				}
			}
			});
        })
		.catch(error => {
            console.error(error);
            throw new Error('Something went wrong D:');
        })
		await fetch(`https://scratchdb.lefty.one/v3/forum/user/graph/${username}/total?segment=6&range=365`)
		.then(res => res.json())
        .then(data => {
            precise = data;
			let dategraph = [];
			let postgraph = [];
			let postgraph2 = [];
			for (let i = 0; i < precise.length - 2; i++)	{
				dategraph.push(`${new Date(precise[i].date).getMonth() + 1}/${new Date(precise[i].date).getDate()}/${new Date(precise[i].date).getFullYear()}`)
				postgraph.push(precise[i].value + postoffset)
                if (i == 0) {
                    postgraph2.push(precise[i].value)
                } else  {
                    postgraph2.push(precise[i].value - precise[i-1].value)
                }
			}
			var chart = new Chart(ctx2, {
			type: "line",
			data: {
				labels: dategraph,
				datasets: [
				{
					label: `${username}`,
					backgroundColor: color,
					borderColor: color,
					data: postgraph,
					fill: false,
					tension: 0.05
				}
				]
			},
			options: {
				responsive: true,
				tooltips: {
					mode: 'index',
					intersect: false,
				},
				hover: {
					mode: 'nearest',
					intersect: true,
				}
			}
			});
            var chart = new Chart(ctx4, {
			type: "line",
			data: {
				labels: dategraph,
				datasets: [
				{
					label: `${username}`,
					backgroundColor: color,
					borderColor: color,
					data: postgraph2,
					fill: false,
					tension: 0.05
				}
				]
			},
			options: {
				responsive: true,
				tooltips: {
					mode: 'index',
					intersect: false,
				},
				hover: {
					mode: 'nearest',
					intersect: true,
				}
			}
			});
        })
		.catch(error => {
            console.error(error);
            throw new Error('Something went wrong D:');
        })
    }
    let promise = getData();
    onMount(async() => {
		ctx = document.getElementById("totalpostcountmonthly").getContext("2d");
		ctx2 = document.getElementById("totalpostcountweekly").getContext("2d");

		ctx3 = document.getElementById("postspermonth").getContext("2d");
		ctx4 = document.getElementById("postsperweek").getContext("2d");

		ctx5 = document.getElementById("postdistribution").getContext("2d");
		
		promise = getData();
    })
</script>
<ul class="category-header-container">
    <li class="category-header-left">Post Distribution</li>
    <li class="category-header-right">As of {new Date().toLocaleString('en-US')}</li>
</ul>
<canvas id="postdistribution" height=125></canvas>
<ul class="category-header-container">
    <li class="category-header-left">Total Post Count</li>
    <li class="category-header-right">Monthly, All Time</li>
</ul>
<canvas id="totalpostcountmonthly" height=75></canvas>
<br>
<ul class="category-header-container">
    <li class="category-header-left">Total Post Count</li>
    <li class="category-header-right">Weekly, Past 12 Months</li>
</ul>
<canvas id="totalpostcountweekly" height=75></canvas>
<ul class="category-header-container">
    <li class="category-header-left">Posts Per Month</li>
    <li class="category-header-right">All Time</li>
</ul>
<canvas id="postspermonth" height=75></canvas>
<ul class="category-header-container">
    <li class="category-header-left">Posts Per Week</li>
    <li class="category-header-right">Past 12 Months</li>
</ul>
<canvas id="postsperweek" height=75></canvas>


<style>
	canvas	{
		overflow: hidden;
	}
    .category-header-container	{
		display: flex;
        flex-direction: row;
		justify-content: flex-end;
		align-items: flex-end;
		width: 100%;
        margin: 0;
        padding: 0;
	}
	.category-header-container li:first-of-type	{
		margin-right: auto;
	}
    .category-header-container li	{
        text-decoration: none;
        list-style: none;
        margin: 0 10px;
    }
    .category-header-left	{
        font-size: 32px;
        font-weight: bold;
    }
	
</style>