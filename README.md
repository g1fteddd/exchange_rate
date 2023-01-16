# Курс валюты

## Задание
Необходимо реализовать интерфейс для получения актуальной информации о курсе валют с выводом подробной информации.

Адрес куда необходимо делать запрос на получение курса валют:

https://www.cbr-xml-daily.ru/daily_json.js

Так же необходимо реализовать пагинацию для валют. ( на одной странице 5 валют )

Дизайн на свой выбор.

## Документация

Интерфейс для полученных данных:
```javascript
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
  [key: string]: ICurrency;
};
```

Функция округления числа используется, потому что мы работаем с вещественными числами:
```javascript
const roundingNumber = (number: number, numberOfDigits: number = 4): number => {
  return Math.round(number * Math.pow(10, numberOfDigits)) / Math.pow(10, numberOfDigits)
}
```

В функции для подсчёта процентного изменения числа происходит проверка на то, какое число больше, исходя из этого мы применяем разные формулы подсчёта:

```javascript
const percentageСhange = (previousValue: number, currentValue: number): number => {
  if (currentValue >= previousValue) {
    const percentageIncrease: number = ((currentValue - previousValue) / previousValue) * 100;
    return roundingNumber(percentageIncrease, 2)
  } else {
    const percentageDecrease: number = ((previousValue - currentValue) / previousValue) * 100 * (-1);
    return roundingNumber(percentageDecrease, 2)
  }
};
```

Загружаем актуальную информацию о курсе валют. Также загружаем предыдущую информацию о курсе валют, это нужно для того, чтобы правильно рассчитать изменение, потому что номинал может измениться и тогда расчёт будет неправильным. Далее идёт пробег по объекту и добавление новых ключей - Changes и Percent, изменение и процентное изменение соответственно.

```javascript
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
```

Для стилизации таблицы я использовал Bootstrap:

```javascript
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
```

Для пагинации я использовал пакет svelte-paginate. Переменные которые нужны для пагинации:

```javascript
$: items = infoKeys;
let currentPage: number = 1;
let pageSize: number = 5;
$: paginatedItems = paginate({ items, pageSize, currentPage });
```

Компонент для пагинации:

```javascript
<LightPaginationNav
  totalItems={items.length}
  {pageSize}
  {currentPage}
  showStepOptions={true}
  on:setPage={(e) => (currentPage = e.detail.page)}
/>
```
