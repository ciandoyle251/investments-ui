<script>
	let ticker = 'AAPL';
	let start = '2020-01-01';
	let end = '2024-12-31';
	let investment_amount = 1000;
	let monthly_contribution = 50;
	let country = 'US';
	let result = null;
	let loading = false;

	async function fetchData() {
		loading = true;
		result = null;

		const url = `https://web-production-feab.up.railway.app/investment_performance?ticker=${ticker}&start=${start}&end=${end}&investment_amount=${investment_amount}&monthly_contribution=${monthly_contribution}&country=${country}`;
		try {
			const res = await fetch(url);
			if (!res.ok) throw new Error('API error');
			result = await res.json();
		} catch (e) {
			result = { error: 'Failed to fetch data. Please check inputs or try again later.' };
		} finally {
			loading = false;
		}
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

{#if result}
	<h2>Result</h2>
	<pre>{JSON.stringify(result, null, 2)}</pre>
{/if}
