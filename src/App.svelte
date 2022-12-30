<script lang="ts">
	import { onMount } from "svelte";
	import { paginate, LightPaginationNav } from "svelte-paginate";
	import Currency from "./components/Currency.svelte";

	let infoAboutCurrency = {};
	let infoKeys: string[] = [];
	let previousInfoAboutCurrency = {};
	let previousInfoKeys: string[] = [];

	// округление числа до X знаков после запятой
	const roundingNumber = (number: number, numberOfDigits: number = 4): number => {
		return Math.round(number * Math.pow(10, numberOfDigits)) / Math.pow(10, numberOfDigits)
	}

	// функция для подсчёта процентного изменения числа
	const percentageСhange = (previousValue: number, currentValue: number): number => {
		if (currentValue >= previousValue) {
			const percentageIncrease: number = ((currentValue - previousValue) / previousValue) * 100;
			return roundingNumber(percentageIncrease, 2)
		} else {
			const percentageDecrease: number = ((previousValue - currentValue) / previousValue) * 100 * (-1);
			return roundingNumber(percentageDecrease, 2)
		}
	};

	

	onMount(async () => {
		const currentResponse: Response = await fetch("https://www.cbr-xml-daily.ru/daily_json.js");
		const currentData: object = await currentResponse.json();

		infoAboutCurrency = { ...currentData["Valute"] };
		infoKeys = Object.keys(infoAboutCurrency);

		const previousResponse: Response = await fetch(currentData["PreviousURL"]);
		const previousData: object = await previousResponse.json();

		previousInfoAboutCurrency = { ...previousData["Valute"] };
		previousInfoKeys = Object.keys(previousInfoAboutCurrency);

		for (const key in infoAboutCurrency) {
			let previousCurrencyValue: number = previousInfoAboutCurrency[key]["Value"];
			let currentCurrencyValue: number = infoAboutCurrency[key]["Value"];

			if (previousInfoAboutCurrency[key]["Nominal"] !== infoAboutCurrency[key]["Nominal"]) {
				previousCurrencyValue /= previousInfoAboutCurrency[key]["Nominal"];
				currentCurrencyValue /= infoAboutCurrency[key]["Nominal"];
				infoAboutCurrency[key]["Changes"] = roundingNumber(currentCurrencyValue - previousCurrencyValue, 4)
				infoAboutCurrency[key]['Percent'] = percentageСhange(previousCurrencyValue, currentCurrencyValue)
			} else {
				infoAboutCurrency[`${key}`]["Changes"] = roundingNumber(currentCurrencyValue - previousCurrencyValue, 4)
				infoAboutCurrency[key]['Percent'] = percentageСhange(previousCurrencyValue, currentCurrencyValue)
			}
		}
	});

	$: items = infoKeys;
	let currentPage: number = 1;
	let pageSize: number = 5;
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
		<Currency currencyInfo={infoAboutCurrency[key]} index={infoKeys.indexOf(key)} />
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
