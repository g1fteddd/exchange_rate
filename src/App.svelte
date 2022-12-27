<script lang="ts">
	import { onMount } from "svelte";
	import Currency from "./components/Currency.svelte";

	let info = {};
	let infoKeys = [];

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
</script>

<main>
	{#each infoKeys as key}
		<Currency currencyInfo={info[key]}/>
	{/each}
</main>

<style>
	main {
		width: 20%;
		margin: 0 auto;
	}
</style>
