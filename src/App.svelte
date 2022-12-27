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
		}
	});

	$: items = infoKeys
	let currentPage = 1
	let pageSize = 5
	$: paginatedItems = paginate({ items, pageSize, currentPage })
</script>

<main>
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
		width: 30%;
		margin: 0 auto;
	}

	:global(.pagination-nav) {
		width: 30%;
		margin: 0 auto;
		margin-top: 30px;
		
		
	}
</style>
