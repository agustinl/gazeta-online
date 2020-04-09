<script>
    import { createEventDispatcher } from 'svelte';
    
    export let setLanguage;

    const dispatch = createEventDispatcher();
    let selected = setLanguage[0] || ""

    function getSelectedLanguage() {
        dispatch('language', {
            id: selected.id == undefined ? "" : selected.id,
            language: selected.language == undefined ? "" : selected.language
		});
    }

    let languages = [
        {id: `ar`, language: `Arabic`},
        {id: `de`, language: `German`},
        {id: `en`, language: `English`},
        {id: `es`, language: `Spanish`},
        {id: `fr`, language: `French`},
        {id: `he`, language: `Hebrew`},
        {id: `it`, language: `Italian`},
        {id: `nl`, language: `Dutch`},
        {id: `no`, language: `Norwegian`},
        {id: `pt`, language: `Portuguese`},
        {id: `ru`, language: `Russian`},
        {id: `se`, language: `Sami`},
        /*{id: `ud`, language: `E`},*/
        {id: `zh`, language: `Chinese`}
    ]
</script>

<select bind:value={selected} on:change={getSelectedLanguage}>
    <option value="">Choose a <b>language</b> if you want...</option>
    {#if setLanguage[0]}
        <option value={setLanguage[0]}>{setLanguage[1]}</option>
    {/if}

    {#each languages as language}
		<option value={language}>
			{language.language}
		</option>
	{/each}
</select>