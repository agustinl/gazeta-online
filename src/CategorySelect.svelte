<script>
    import { createEventDispatcher } from 'svelte';
    
    export let setCategory;    

    const dispatch = createEventDispatcher();
    let selected = setCategory[0] || ""

    function getSelectedCategory() {
        dispatch('category', {
            id: selected.id == undefined ? "" : selected.id,
            category: selected.category == undefined ? "" : selected.category
		});
    }

    let categories = [
        {id: `general`, category: `General`},
        {id: `business`, category: `Business`},
        {id: `entertainment`, category: `Entertainment`},
        {id: `health`, category: `Health`},
        {id: `science`, category: `Science`},
        {id: `sports`, category: `Sports`},
        {id: `Technology`, category: `Technology`}
    ]
</script>

<select bind:value={selected} on:change={getSelectedCategory}>
    <option value="">Choose a <b>category</b> if you want...</option>
    {#if setCategory[0]}
        <option value={setCategory[0]}>{setCategory[1]}</option>
    {/if}

    {#each categories as category}
		<option value={category}>
			{category.category}
		</option>
	{/each}
</select>
