<script>
    import { createEventDispatcher } from 'svelte';
    
    export let setCountry;

    const dispatch = createEventDispatcher();
    let selected = setCountry[0] || ""

    function getSelectedCountry() {
        dispatch('country', {
            id: selected.id == undefined ? "" : selected.id,
            country: selected.country == undefined ? "" : selected.country
		});
    }

    let countries = [
        {id: `ar`, country: `Argentina`},
        {id: `au`, country: `Australia`},
        {id: `at`, country: `Austria`},
        {id: `be`, country: `Belgium`},
        {id: `br`, country: `Brazil`},
        {id: `bg`, country: `Bulgaria`},
        {id: `ca`, country: `Canada`},
        {id: `cn`, country: `China`},
        {id: `co`, country: `Colombia`},
        {id: `cu`, country: `Cuba`},
        {id: `cz`, country: `Czech Republic`},
        {id: `eg`, country: `Egypt`},
        {id: `fr`, country: `France`},
        {id: `de`, country: `Germany`},
        {id: `gr`, country: `Greece`},
        {id: `hk`, country: `Hong Kong`},
        {id: `hu`, country: `Hungary`},
        {id: `in`, country: `India`},
        {id: `id`, country: `Indonesia`},
        {id: `ie`, country: `Ireland`},
        {id: `il`, country: `Israel`},
        {id: `it`, country: `Italy`},
        {id: `jp`, country: `Japan`},
        {id: `lv`, country: `Latvia`},
        {id: `lt`, country: `Lithuania`},
        {id: `my`, country: `Malaysia`},
        {id: `mx`, country: `Mexico`},
        {id: `ma`, country: `Morocco`},
        {id: `nl`, country: `Netherlands`},
        {id: `nz`, country: `New Zealand`},
        {id: `ng`, country: `Nigeria`},
        {id: `no`, country: `Norway`},
        {id: `ph`, country: `Philippines`},
        {id: `pl`, country: `Poland`},
        {id: `pt`, country: `Portugal`},
        {id: `ro`, country: `Romania`},
        {id: `ru`, country: `Russia`},
        {id: `sa`, country: `Saudi Arabia`},
        {id: `rs`, country: `Serbia`},
        {id: `sg`, country: `Singapore`},
        {id: `sk`, country: `Slovakia`},
        {id: `si`, country: `Slovenia`},
        {id: `za`, country: `South Africa`},
        {id: `kr`, country: `South Korea`},
        {id: `se`, country: `Sweden`},
        {id: `ch`, country: `Switzerland`},
        {id: `tw`, country: `Taiwan`},
        {id: `th`, country: `Thailand`},
        {id: `tr`, country: `Turkey`},
        {id: `ae`, country: `UAE`},
        {id: `ua`, country: `Ukraine`},
        {id: `gb`, country: `United Kingdom`},
        {id: `us`, country: `United States`},
        {id: `ve`, country: `Venezuela`}
    ];
</script>

<select bind:value={selected} on:change={getSelectedCountry}>
    <option value="">Choose a country if you want...</option>
    {#if setCountry[0]}
        <option value={setCountry[0]}>{setCountry[1]}</option>
    {/if}

    {#each countries as country}
		<option value={country}>
			{country.country}
		</option>
	{/each}
</select>
