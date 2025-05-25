<script>
	import { onMount, tick } from 'svelte';
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

	const popularTickers = [
		{ name: 'S&P 500', symbol: '^GSPC' },
		{ name: 'NASDAQ 100', symbol: '^NDX' },
		{ name: 'Dow Jones', symbol: '^DJI' },
		{ name: 'Russell 2000', symbol: '^RUT' },
		{ name: 'FTSE 100', symbol: '^FTSE' },
		{ name: 'DAX', symbol: '^GDAXI' },
		{ name: 'CAC 40', symbol: '^FCHI' },
		{ name: 'Nikkei 225', symbol: '^N225' },
		{ name: 'Hang Seng', symbol: '^HSI' },
		{ name: 'Apple Inc.', symbol: 'AAPL' }
	];

	async function fetchData() {
		loading = true;
		result = null;

		const url = `https://web-production-feab.up.railway.app/investment_performance?ticker=${ticker}&start=${start}&end=${end}&investment_amount=${investment_amount}&monthly_contribution=${monthly_contribution}&country=${country}`;
		
		try {
			const res = await fetch(url);
			if (!res.ok) throw new Error('API error');
			result = await res.json();

			if (result?.monthly_data?.length) {
				await tick(); // wait for canvas to be rendered
				drawChart(result.monthly_data);
			}
		} catch (e) {
			result = { error: 'Failed to fetch data. Please check inputs or try again later.' };
		} finally {
			loading = false;
		}
	}

	function drawChart(data) {
		const labels = data.map(item => item.month);
		const nominalValues = [];
		const realValues = [];
		const cashValues = [];

		let nominalValue = result.investment_amount;
		let realValue = result.investment_amount;
		let cashValue = result.investment_amount;

		data.forEach((item, i) => {
			if (i > 0) {
				nominalValue += result.monthly_contribution;
				realValue += result.monthly_contribution;
				cashValue += result.monthly_contribution;
			}

			// Apply returns
			nominalValue *= (1 + item.nominal_return_percent / 100);
			realValue *= (1 + item.real_return_percent / 100);

			// Adjust for inflation
			cashValue *= (1 / (1 + item.inflation_rate_percent / 100));

			nominalValues.push(nominalValue.toFixed(2));
			realValues.push(realValue.toFixed(2));
			cashValues.push(cashValue.toFixed(2));
		});

		if (chart) chart.destroy();

		chart = new Chart(chartCanvas, {
			type: 'line',
			data: {
				labels,
				datasets: [
					{
						label: `${ticker} Nominal Value`,
						data: nominalValues,
						borderColor: 'royalblue',
						backgroundColor: 'rgba(65, 105, 225, 0.2)',
						fill: false,
						tension: 0.3
					},
					{
						label: `${ticker} Real Value`,
						data: realValues,
						borderColor: 'green',
						backgroundColor: 'rgba(34, 139, 34, 0.2)',
						fill: false,
						tension: 0.3
					},
					{
						label: 'Cash Buying Power',
						data: cashValues,
						borderColor: 'orangered',
						backgroundColor: 'rgba(255, 69, 0, 0.2)',
						fill: false,
						borderDash: [5, 5],
						tension: 0.3
					}
				]
			},
			options: {
				responsive: true,
				scales: {
					x: {
						title: { display: true, text: 'Month' }
					},
					y: {
						title: { display: true, text: 'Value ($)' },
						beginAtZero: false
					}
				}
			}
		});
	}
</script>

<style>
	h1 {
		margin-bottom: 1em;
	}
	.input-row {
		display: flex;
		flex-wrap: wrap;
		gap: 1em;
		margin-bottom: 1em;
	}
	.left-inputs {
		display: flex;
		flex-direction: column;
		gap: 0.5em;
		flex: 1;
		max-width: 400px;
	}
	.dropdown {
		display: flex;
		flex-direction: column;
		gap: 0.3em;
		min-width: 200px;
	}
	input, select, button {
		padding: 0.5em;
		font-size: 1em;
	}
	canvas {
		margin-top: 2em;
		max-width: 100%;
	}
	p.error {
		color: red;
	}
</style>

<h1>Investment Performance Tracker</h1>

<form on:submit|preventDefault={fetchData}>
	<div class="input-row">
		<div class="left-inputs">
			<input bind:value={ticker} placeholder="Ticker (e.g. AAPL)" />
			<input type="date" bind:value={start} />
			<input type="date" bind:value={end} />
			<input type="number" bind:value={investment_amount} placeholder="Initial Investment" />
			<input type="number" bind:value={monthly_contribution} placeholder="Monthly Contribution" />
			<input bind:value={country} placeholder="Country Code (e.g. US)" />
			<button type="submit" disabled={loading}>{loading ? 'Loading...' : 'Get Performance'}</button>
		</div>
		<div class="dropdown">
			<label>Popular Indexes:</label>
			<select on:change={(e) => ticker = e.target.value}>
				<option value="">-- Choose an index --</option>
				{#each popularTickers as index}
					<option value={index.symbol}>{index.name}</option>
				{/each}
			</select>
		</div>
	</div>
</form>

{#if result?.monthly_data}
	<h2>Nominal vs Real Value</h2>
	<canvas bind:this={chartCanvas}></canvas>
{/if}

{#if result?.error}
	<p class="error">{result.error}</p>
{/if}
