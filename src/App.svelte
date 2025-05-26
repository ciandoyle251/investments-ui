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

	let avgAnnualNominalReturn = null;
	let avgAnnualRealReturn = null;
	let avgAnnualInflation = null;

	const tickerCategories = [
		{
			label: 'Popular Indexes',
			options: [
				{ name: 'S&P 500', symbol: '^GSPC', description: 'Tracks 500 large US companies' },
				{ name: 'NASDAQ 100', symbol: '^NDX', description: 'Focuses on top 100 tech-heavy US stocks' },
				{ name: 'Dow Jones', symbol: '^DJI', description: '30 major US companies in all sectors' },
				{ name: 'Russell 2000', symbol: '^RUT', description: 'Tracks 2000 small-cap US companies' },
				{ name: 'FTSE 100', symbol: '^FTSE', description: 'Top 100 companies on the London Stock Exchange' },
				{ name: 'DAX', symbol: '^GDAXI', description: 'Top 40 blue-chip German stocks' },
				{ name: 'CAC 40', symbol: '^FCHI', description: 'Top 40 French companies on Euronext Paris' },
				{ name: 'Nikkei 225', symbol: '^N225', description: 'Top 225 Japanese companies' },
				{ name: 'Hang Seng', symbol: '^HSI', description: 'Major Hong Kong-listed companies' }
			]
		},
		{
			label: 'Commodities',
			options: [
				{ name: 'Gold', symbol: 'GC=F', description: 'Gold futures, a hedge against inflation and crisis' },
				{ name: 'Silver', symbol: 'SI=F', description: 'Silver futures, industrial and precious metal' },
				{ name: 'Crude Oil (WTI)', symbol: 'CL=F', description: 'US West Texas Intermediate oil futures' },
				{ name: 'Brent Oil', symbol: 'BZ=F', description: 'Global benchmark for crude oil' },
				{ name: 'Natural Gas', symbol: 'NG=F', description: 'Natural gas futures, seasonal and volatile' },
				{ name: 'Copper', symbol: 'HG=F', description: 'Often used as an economic health indicator' }
			]
		},
		{
			label: 'REITs',
			options: [
				{ name: 'Vanguard Real Estate ETF', symbol: 'VNQ', description: 'Tracks US real estate investment trusts' },
				{ name: 'Schwab U.S. REIT ETF', symbol: 'SCHH', description: 'Diversified exposure to US REITs' },
				{ name: 'iShares Global REIT ETF', symbol: 'REET', description: 'Exposure to global real estate companies' },
				{ name: 'Realty Income Corporation', symbol: 'O', description: 'Monthly dividend REIT focused on retail' },
				{ name: 'Public Storage', symbol: 'PSA', description: 'Leading self-storage REIT in the US' }
			]
		}
	];

	function calculateAverageAnnualReturns(data, initialInvestment, monthlyContribution) {
		const totalMonths = data.length;
		const totalNominalCashInvested = initialInvestment + monthlyContribution * (totalMonths - 1);

		const sumMonthsInvestedForContributions = ((totalMonths - 1) * totalMonths) / 2;
		const weightedMonthsInvested =
			initialInvestment * totalMonths + monthlyContribution * sumMonthsInvestedForContributions;
		const weightedAvgMonthsInvested = weightedMonthsInvested / totalNominalCashInvested;
		const weightedAvgYearsInvested = weightedAvgMonthsInvested / 12;

		const finalNominalValue = data[data.length - 1].nominal_value;
		const finalRealValue = data[data.length - 1].real_value;

		const avgAnnualNominalReturn = Math.pow(finalNominalValue / totalNominalCashInvested, 1 / weightedAvgYearsInvested) - 1;
		const avgAnnualRealReturn = Math.pow(finalRealValue / totalNominalCashInvested, 1 / weightedAvgYearsInvested) - 1;

		return {
			avgAnnualNominalReturn: avgAnnualNominalReturn * 100,
			avgAnnualRealReturn: avgAnnualRealReturn * 100,
			weightedAvgYearsInvested
		};
	}

	async function fetchData() {
		loading = true;
		result = null;
		avgAnnualNominalReturn = null;
		avgAnnualRealReturn = null;
		avgAnnualInflation = null;

		const url = `https://web-production-feab.up.railway.app/investment_performance?ticker=${ticker}&start=${start}&end=${end}&investment_amount=${investment_amount}&monthly_contribution=${monthly_contribution}&country=${country}`;

		try {
			const res = await fetch(url);
			if (!res.ok) throw new Error('API error');
			result = await res.json();

			if (result?.monthly_data?.length) {
				calculateMonthlyFinalValues(result.monthly_data);

				const returns = calculateAverageAnnualReturns(
					result.monthly_data,
					result.investment_amount,
					result.monthly_contribution
				);

				avgAnnualNominalReturn = returns.avgAnnualNominalReturn;
				avgAnnualRealReturn = returns.avgAnnualRealReturn;
				avgAnnualInflation = result.annualized_inflation_percent;

				await tick();
				drawChart(result.monthly_data);
			}
		} catch (e) {
			result = { error: 'Failed to fetch data. Please check inputs or try again later.' };
		} finally {
			loading = false;
		}
	}

	function calculateMonthlyFinalValues(data) {
		let nominalValue = result.investment_amount;
		let realValue = result.investment_amount;

		for (let i = 0; i < data.length; i++) {
			if (i > 0) {
				nominalValue += result.monthly_contribution;
				realValue += result.monthly_contribution;
			}
			nominalValue *= (1 + data[i].nominal_return_percent / 100);
			realValue *= (1 + data[i].real_return_percent / 100);

			data[i].nominal_value = nominalValue;
			data[i].real_value = realValue;
		}
	}

	function drawChart(data) {
		const labels = data.map(item => item.month);
		const nominalValues = data.map(item => item.nominal_value.toFixed(2));
		const realValues = data.map(item => item.real_value.toFixed(2));

		let cashValue = result.investment_amount;
		let nominalCash = result.investment_amount;
		const cashValues = [];
		const nominalCashValues = [];

		data.forEach((item, i) => {
			if (i > 0) {
				cashValue += result.monthly_contribution;
				nominalCash += result.monthly_contribution;
			}

			cashValue *= 1 / (1 + item.inflation_rate_percent / 100);

			cashValues.push(cashValue.toFixed(2));
			nominalCashValues.push(nominalCash.toFixed(2));
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
					},
					{
						label: 'Nominal Cash (No Investment Growth)',
						data: nominalCashValues,
						borderColor: 'gray',
						backgroundColor: 'rgba(128,128,128,0.2)',
						fill: false,
						borderDash: [3, 3],
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
	.stats {
		margin-bottom: 1em;
	}
	.stats strong {
		font-size: 1.5em;
		margin-right: 1.5em;
		display: inline-block;
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
			<label>Popular Assets:</label>
			<select on:change={(e) => ticker = e.target.value}>
				<option value="">-- Choose an asset --</option>
				{#each tickerCategories as category}
					<optgroup label={category.label}>
						{#each category.options as option}
							<option value={option.symbol} title={option.description}>
								{option.name}
							</option>
						{/each}
					</optgroup>
				{/each}
			</select>
		</div>
	</div>
</form>

{#if result?.monthly_data}
	<div class="stats">
		<strong>
			Average Annual Nominal Return: {avgAnnualNominalReturn?.toFixed(2)}%
		</strong>
		<strong>
			Average Annual Real Return: 
			<span style="color: {avgAnnualRealReturn >= 0 ? 'green' : 'red'}">
				{avgAnnualRealReturn?.toFixed(2)}%
			</span>
		</strong>
		<strong>
			Average Annual Inflation: {avgAnnualInflation?.toFixed(2)}%
		</strong>
	</div>
	<h2>Nominal vs Real Value</h2>
	<canvas bind:this={chartCanvas}></canvas>
{/if}

{#if result?.error}
	<p class="error">{result.error}</p>
{/if}
