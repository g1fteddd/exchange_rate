<script lang="ts">
    export let currencyInfo: object;
    export let index: number;
</script>

<div class="currency {index % 2 !== 0 ? 'gray' : ''}">
    <div class="charcode">
        <p>{currencyInfo["CharCode"]}</p>
    </div>
    <div class="nominal">
        <p>{currencyInfo["Nominal"]}</p>
    </div>
    <div class="name">
        <p>{currencyInfo["Name"]}</p>
    </div>
    <div class="value">
        <p>{currencyInfo["Value"]}</p>
    </div>
    <div class="changes {currencyInfo['Changes'] >= 0 ? 'green' : 'red'}">
        <p>
            {currencyInfo["Changes"] > 0 ? `+${currencyInfo["Changes"]}` : `${currencyInfo["Changes"]}`}
        </p>
    </div>
    <div class="percent {currencyInfo['Changes'] >= 0 ? 'green' : 'red'}">
        <p>
            {currencyInfo["Percent"] >= 0 ? `+${currencyInfo["Percent"]}` : `${currencyInfo["Percent"]}`}%
        </p>
    </div>
</div>

<style>
    .currency {
        display: flex;
        justify-content: space-around;
        font-size: 14px;
        text-align: center;
    }

    .charcode {
        width: 55px;
        color: #8d96b2;
    }

    .nominal {
        width: 71px;
        color: #8d96b2;
    }

    .name {
        width: 146px;
        font-weight: 500;
    }

    .value {
        width: 146px;
    }

    .changes {
        width: 146px;
    }

    .percent {
        width: 146px;
    }

    .green {
        color: #28bc00;
    }

    .red {
        color: #ff564e;
    }

    .gray {
        background-color: #f9fafb;
    }

    .currency:hover {
        background-color: #ebecee;
        transition: 0.3s;
    }
</style>
