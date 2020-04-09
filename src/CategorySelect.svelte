<script>
    import { createEventDispatcher } from 'svelte';
    
    export let setCategory;

    const dispatch = createEventDispatcher();
    let selected = setCategory || ""

    function getSelectedCategory() {
        dispatch('category', {
			category: selected
		});
    }
</script>

<select bind:value={selected} on:change={getSelectedCategory}>
    <option value="">Choose a <b>category</b> if you want...</option>
    <option value="general">General</option>
    <option value="business">Business</option>
    <option value="entertainment">Entertainment</option>
    <option value="health">Health</option>
    <option value="science">Science</option>
    <option value="sports">Sports</option>
    <option value="Technology">Technology</option>
</select>

<style>
select {
    margin-top:20px;
}
</style>