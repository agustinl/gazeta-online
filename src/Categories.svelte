<script>
    export let categories;
</script>

<div class="categories">
    {#each categories as categorie, i}
        <mark>{categorie}</mark>
    {/each}
</div>

<style>
</style>