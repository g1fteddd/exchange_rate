<script lang="ts">
    export let name: string;
</script>

<div class="currency">
    <p>{name}</p>
</div>

<style>
    :global(body) {
        background-color: gray;
    }
    .currency {
        background-color: red;
        width: 40%;
        height: 200px;
    }
</style>