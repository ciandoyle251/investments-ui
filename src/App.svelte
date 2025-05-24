<script>
	import { tick } from 'svelte';
	import { Chart, registerables } from 'chart.js';
	Chart.register(...registerables);

	let ticker = 'AAPL';
	let start = '2020-01-01';
	let end = '2024-12-31';
	let investment_amount = 1000;
	let monthly_contribution = 50;
	let country = 'US';
	let result = null;
	let loading = false;
	let chartCanvas;
	let chart;

	async function fetchData() {
		loading = true;
		result = null;

		const url = `https://web-production-feab.up.railway.app/investment_performance?ticker=${ticker}&start=${start}&end=${end}&investment_amount=${investment_amount}&monthly_contribution=${monthly_contribution}&country=${country}`;
		try {
			const res = await fetch(url);
			if (!res.ok) throw new Error('API error');
			result = await res.json();
			console.log('API result:', result);

			if (result?.monthly_data?.length) {
				await tick();
				drawChart(result.monthly_data);
			}
		} catch (e) {
			console.error(e);
			result = { error: 'Failed to fetch data. Please check inputs or try again later.' };
		} finally {
			loading = false;
		}
	}

	function drawChart(data) {
		console.log("Drawing chart with corrected keys:", data);

		const labels = data.map(item => item.month);
		const values = data.map(item => parseFloat(item.investment_value));

		if (!labels.length || !values.length || values.some(isNaN)) {
			console.warn("Chart data invalid:", { labels, values });
			return;
		}

		if (chart) chart.destroy();

		chart = new Chart(chartCanvas, {
			type: 'line',
			data: {
				labels,
				datasets: [{
					label: `${ticker} Investment Value Over Time`,
					data: values,
					borderColor: 'royalblue',
					backgroundColor: 'rgba(65, 105, 225, 0.2)',
					fill: true,
					tension: 0.3,
				}]
			},
			options: {
				responsive: true,
				scales: {
					x: {
						title: { display: true, text: 'Month' }
					},
					y: {
						title: { display: true, text: 'Investment Value ($)' },
						beginAtZero: false
					}
				}
			}
		});
	}

</script>

<style>
	form {
		margin: 1em 0;
		display: flex;
		flex-direction: column;
		gap: 0.5em;
		max-width: 400px;
	}
	input, button {
		padding: 0.5em;
		font-size: 1em;
	}
	pre {
		background: #f0f0f0;
		padding: 1em;
		overflow: auto;
	}
	canvas {
		margin-top: 2em;
		min-height: 300px;
		width: 100%;
	}
</style>

<h1>Investment Performance Tracker</h1>

<form on:submit|preventDefault={fetchData}>
	<input bind:value={ticker} placeholder="Ticker (e.g. AAPL)" />
	<input type="date" bind:value={start} />
	<input type="date" bind:value={end} />
	<input type="number" bind:value={investment_amount} placeholder="Initial Investment" />
	<input type="number" bind:value={monthly_contribution} placeholder="Monthly Contribution" />
	<input bind:value={country} placeholder="Country Code (e.g. US)" />
	<button type="submit" disabled={loading}>{loading ? 'Loading...' : 'Get Performance'}</button>
</form>

{#if result?.monthly_data}
	<h2>Performance Chart</h2>
	<canvas bind:this={chartCanvas} width="600" height="400"></canvas>
{/if}

{#if result?.error}
	<p style="color: red">{result.error}</p>
{/if}
