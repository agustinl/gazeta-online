<script>
    import { createEventDispatcher } from 'svelte';
    
    export let setLanguage;

    const dispatch = createEventDispatcher();
    let selected = setLanguage || ""

    function getSelectedLanguage() {
        dispatch('language', {
			language: selected
		});
    }
</script>

<select bind:value={selected} on:change={getSelectedLanguage}>
    <option value="">Choose a <b>language</b> if you want...</option>
    <option value="ar">ar</option>
    <option value="de">de</option>
    <option value="en">en</option>
    <option value="es">es</option>
    <option value="fr">fr</option>
    <option value="he">he</option>
    <option value="it">it</option>
    <option value="nl">nl</option>
    <option value="no">no</option>
    <option value="pt">pt</option>
    <option value="ru">ru</option>
    <option value="se">se</option>
    <option value="ud">ud</option>
    <option value="zh">zh</option>
</select>

<style>
select {
    margin-top:20px;
}
</style>