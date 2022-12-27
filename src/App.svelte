<script lang="ts">
	import { onMount } from "svelte";
	import { paginate, DarkPaginationNav } from 'svelte-paginate'
	import Currency from "./components/Currency.svelte";

	let info = {};
	let infoKeys: string[] = [];

	onMount(async () => {
		const response: Response = await fetch(
			"https://www.cbr-xml-daily.ru/daily_json.js"
		);
		const data = await response.json();
		const currency = data["Valute"];
		for (const key in currency) {
			info[`${key}`] = currency[key];
			infoKeys = [...infoKeys, key];
			info[`${key}`]['Changes'] = Math.round((info[`${key}`]['Value'] - info[`${key}`]['Previous']) * 10000) / 10000
		}
	});

	$: items = infoKeys
	let currentPage = 1
	let pageSize = 5
	$: paginatedItems = paginate({ items, pageSize, currentPage })
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
		<Currency currencyInfo={info[key]}/>
	{/each}
</main>

<DarkPaginationNav
  totalItems="{items.length}"
  pageSize="{pageSize}"
  currentPage="{currentPage}"
  limit="{1}"
  showStepOptions="{true}"
  on:setPage="{(e) => currentPage = e.detail.page}"
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
		height: 45px;
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
		width: 30%;
		margin: 0 auto;
		margin-top: 30px;
		
		
	}


</style>
