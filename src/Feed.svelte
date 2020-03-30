<script>
    import Categories from './Categories.svelte';

    export let feed;
</script>

<div class="feed-header">
    <h2>
        <svg xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-rss" width="24" height="24" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" fill="none" stroke-linecap="round" stroke-linejoin="round">
            <path stroke="none" d="M0 0h24v24H0z"/>
            <circle cx="5" cy="19" r="1" />
            <path d="M4 4a16 16 0 0 1 16 16" />
            <path d="M4 11a9 9 0 0 1 9 9" />
        </svg>
        <a href={feed[2]} target="_blank" rel="noopener">{feed[0]}</a></h2>
    <hr>
    <p>{feed[1]}</p>
</div>
{#each feed as article, i}
    {#if i > 2}
        <article>
            {#if article.categories != ""}
                <Categories categories={article.categories} />
            {/if}
            <h3><a href="{article.link}" target="_blank" rel="noopener">{@html article.title}</a></h3>
            <div class="article-info">
                <p>{@html article.description}</p>
            </div>
            {#if article.pubdate != ""}
                <time>{article.pubdate}</time>
            {/if}
        </article>
    {/if}
{/each}

<style>

</style>