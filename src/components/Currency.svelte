<script lang="ts">
    export let currencyInfo;
</script>

<div class="currency">
    <div class="name">{currencyInfo['Name']}</div>
    <div class="charcode">Букв. код: <span>{currencyInfo['CharCode']}</span></div>
    <div class="numcode">Цифр. код: <span>{currencyInfo['NumCode']}</span></div>
    <div class="nominal">Единиц: <span>{currencyInfo['Nominal']}</span></div>
    <div class="value {currencyInfo['Previous'] > currencyInfo['Value'] ? 'green' : 'red'}">Курс: <span>{currencyInfo['Value']}</span></div>
    

</div>

<style>
    :global(body) {
        background-color: gray;
    }
    .currency {
        background-color: purple;
        color: white;
        border-radius: 10px;
        text-align: center;
        display: flex;
        justify-content: space-around;
        align-items: top;
        flex-wrap: wrap;
        margin-top: 20px;
    }

    span {
        font-weight: 700;
    }

    .name {
        width: 100%;
    }

    .charcode {
        width: 50%;
    }

    .numcode {
        width: 50%;
    }

    .nominal {
        width: 50%;
    }

    .value {
        width: 50%;
    }

    .red {
        background-color: red;
    }

    .green {
        background-color: green;
    }

</style>