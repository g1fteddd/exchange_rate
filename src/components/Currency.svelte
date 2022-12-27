<script lang="ts">
    export let currencyInfo;

</script>

<div class="currency">
    <div class="charcode">{currencyInfo['CharCode']}</div>
    <div class="nominal">{currencyInfo['Nominal']}</div>
    <div class="name">{currencyInfo['Name']}</div>
    <div class="value">{currencyInfo['Value']}</div>
    <div class="changes">{currencyInfo['Changes'] > 0 ? '+' : ''}{currencyInfo['Changes']}</div>   
    <div class="percent">+0.37%</div> 
    
</div>

<style>
    .currency {
        display: flex;
        justify-content: space-around;
        height: 80px;
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
</style>