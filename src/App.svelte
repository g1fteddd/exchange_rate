<script lang="ts">
	import { onMount } from "svelte";
	import { paginate, LightPaginationNav } from "svelte-paginate";
	import Currency from "./components/Currency.svelte";

	let info = {};
	let infoKeys: string[] = [];
	let previousInfo = {};
	let previousInfoKeys: string[] = [];

	onMount(async () => {
		const currentResponse: Response = await fetch("https://www.cbr-xml-daily.ru/daily_json.js");
		const currentData = await currentResponse.json();

		info = {...currentData['Valute']}
		infoKeys = Object.keys(info)


		const previousResponse: Response = await fetch(currentData['PreviousURL']);
		const previousData = await previousResponse.json();

		previousInfo = {...previousData['Valute']}
		previousInfoKeys = Object.keys(previousInfo)


		for (const key in info) {
			let previousCurrency = previousInfo[key]['Value']
			let currentCurrency = info[key]['Value']

			if (previousInfo[key]['Nominal'] !== info[key]['Nominal']) {
				previousCurrency /= previousInfo[key]['Nominal']
				currentCurrency /= info[key]['Nominal']
				info[key]['Changes'] = Math.round((currentCurrency - previousCurrency) * 10000) / 10000
			} else {
				info[`${key}`]["Changes"] = Math.round((currentCurrency - previousCurrency) * 10000) / 10000
			}

			
			
		}
	});

	

	$: items = infoKeys;
	let currentPage = 1;
	let pageSize = 5;
	$: paginatedItems = paginate({ items, pageSize, currentPage });
</script>

<main>
	<div class="columns">
		<div class="column_charcode">Код</div>
		<div class="column_nominal">Номинал</div>
		<div class="column_currency">Валюта</div>
		<div class="column_value">Курс ЦБ</div>
		<div class="column_changes">Изменения</div>
		<div class="column_percent">%</div>
	</div>
	{#each paginatedItems as key}
		<Currency currencyInfo={info[key]} id={infoKeys.indexOf(key)} />
	{/each}
</main>

<LightPaginationNav
	totalItems={items.length}
	{pageSize}
	{currentPage}
	showStepOptions={true}
	on:setPage={(e) => (currentPage = e.detail.page)}
/>

<style>
	main {
		width: 80%;
		margin: 0 auto;
	}

	.columns {
		display: flex;
		justify-content: space-around;
		color: #8d96b2;
		height: 30px;
		font-size: 14px;
		text-align: center;
		border-bottom: 1px solid rgba(137, 137, 137, 0.2);
	}

	.column_charcode {
		width: 55px;
	}

	.column_nominal {
		width: 71px;
	}

	.column_currency {
		width: 146px;
	}

	.column_value {
		width: 146px;
	}

	.column_changes {
		width: 146px;
	}

	.column_percent {
		width: 146px;
	}

	:global(.pagination-nav) {
		position: absolute;
		bottom: 30%;
		left: 50%;
		transform: translate(-50%, -30%);
	}
</style>
