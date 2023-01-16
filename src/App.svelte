<script lang="ts">
    import { onMount } from "svelte";
    import { paginate, LightPaginationNav } from "svelte-paginate";

	interface ICurrency {
		Changes: number;
		CharCode: string;
		ID: string;
		Name: string;
		Nominal: number;
		NumCode: string;
		Percent: number;
		Previous: number;
		Value: number;
	};

	interface IGroupCurrency {
		[key: string]: ICurrency
	}


    let infoAboutCurrency: IGroupCurrency = {};
    let infoKeys: string[] = [];
    let previousInfoAboutCurrency: IGroupCurrency = {};
    let previousInfoKeys: string[] = [];

    // округление числа до X знаков после запятой
    const roundingNumber = (number: number, numberOfDigits: number = 4): number => {
        return Math.round(number * Math.pow(10, numberOfDigits)) / Math.pow(10, numberOfDigits);
    };

    // функция для подсчёта процентного изменения числа
    const percentageСhange = (previousValue: number, currentValue: number): number => {
        if (currentValue >= previousValue) {
            const percentageIncrease: number = ((currentValue - previousValue) / previousValue) * 100;
            return roundingNumber(percentageIncrease, 2);
        } else {
            const percentageDecrease: number = ((previousValue - currentValue) / previousValue) * 100 * -1;
            return roundingNumber(percentageDecrease, 2);
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
                infoAboutCurrency[key]["Changes"] = roundingNumber(currentCurrencyValue - previousCurrencyValue, 4);
                infoAboutCurrency[key]["Percent"] = percentageСhange(previousCurrencyValue, currentCurrencyValue);
            } else {
                infoAboutCurrency[key]["Changes"] = roundingNumber(currentCurrencyValue - previousCurrencyValue, 4);
                infoAboutCurrency[key]["Percent"] = percentageСhange(previousCurrencyValue, currentCurrencyValue);
            }
        }
    });

    $: items = infoKeys;
    let currentPage: number = 1;
    let pageSize: number = 5;
    $: paginatedItems = paginate({ items, pageSize, currentPage });

	$: console.log(infoAboutCurrency)
</script>

<main>
    <table class="table table-striped table-hover">
        <thead>
            <tr>
                <th class="text-center" scope="col">Код</th>
                <th class="text-center" scope="col">Номинал</th>
                <th class="text-center" scope="col">Валюта</th>
                <th class="text-center" scope="col">Курс ЦБ</th>
                <th class="text-center" scope="col">Изменения</th>
                <th class="text-center" scope="col">%</th>
            </tr>
        </thead>
        <tbody>
            {#each paginatedItems as key}
                <tr>
                    <td class="text-center">{infoAboutCurrency[key]["CharCode"]}</td>
                    <td class="text-center">{infoAboutCurrency[key]["Nominal"]}</td>
                    <td class="text-center name">{infoAboutCurrency[key]["Name"]}</td>
                    <td class="text-center">{infoAboutCurrency[key]["Value"]}</td>
                    <td class="text-center">
                        <p class={infoAboutCurrency[key]["Changes"] >= 0 ? "green" : "red"}>
                            {infoAboutCurrency[key]["Changes"] >= 0 ? `+${infoAboutCurrency[key]["Changes"]}` : `${infoAboutCurrency[key]["Changes"]}`}
                        </p>
                    </td>
                    <td class="text-center">
                        <p class={infoAboutCurrency[key]["Changes"] >= 0 ? "green" : "red"}>
                            {infoAboutCurrency[key]["Percent"] >= 0 ? `+${infoAboutCurrency[key]["Percent"]}` : `${infoAboutCurrency[key]["Percent"]}`}%
                        </p>
                    </td>
                </tr>
            {/each}
        </tbody>
    </table>
</main>

<LightPaginationNav
    totalItems={items.length}
    {pageSize}
    {currentPage}
    showStepOptions={true}
    on:setPage={(e) => (currentPage = e.detail.page)}
/>

<style>
    :global(.pagination-nav) {
        position: absolute;
        bottom: 30%;
        left: 50%;
        transform: translate(-50%, -30%);
    }

    .green {
        color: #28bc00;
    }

    .red {
        color: #ff564e;
    }
</style>
