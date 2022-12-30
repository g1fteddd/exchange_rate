# Курс валюты

## Документация

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

Далее идёт цикл и в пропсах мы передаём информацию о валюте и индекс. Индекс нужен для стилизации.

```javascript
{#each paginatedItems as key}
	<Currency currencyInfo={infoAboutCurrency[key]} index={infoKeys.indexOf(key)} />
{/each}
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
