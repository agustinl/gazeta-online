<script>
    import { onMount, beforeUpdate, afterUpdate } from 'svelte';
    import Header from './Header.svelte'
    import Article from './Article.svelte'
    import NewFeed from './NewFeed.svelte'

    let sites;
    let storedSites;
    $: feedMessage = "There is no site loaded yet. Put your favorite news sites below."
    $: articles = [];

    onMount(async () => {
        if (sites == null) return;

        if(sites.category || sites.country) {
            var url = `https://newsapi.org/v2/top-headlines?pageSize=${sites.max_news_to_show}&country=${sites.country}&category=${sites.category}&apiKey=b3408247a56444d5b1f87b48f5e69165`
        } else {
            var url = `https://newsapi.org/v2/everything?pageSize=${sites.max_news_to_show}&domains=${sites.news_list}&language=${sites.language}&apiKey=b3408247a56444d5b1f87b48f5e69165`

        }
        
        var req = new Request(url);
        
        /*articles = await fetch(req).then(function(response) {
            return response.json()
        })
        .catch(function(error) {
            console.log('Hubo un problema con la petición Fetch:' + error.message);
        });*/

        if(articles.status == "ok" && articles.totalResults > 0) {
            articles = articles.articles;
        } else if (articles.status == "ok" && articles.totalResults === 0) {
            feedMessage = `We were unable to load your news feed. Maybe the sites you placed are not available. Try others, there is a lot`;
        } else if (articles.status == "error") {
            feedMessage = articles.message
        }        
	});

    beforeUpdate(() => {
        sites = localStorage.getItem('gazeta_online');
        sites = JSON.parse(sites);
    });
</script>

<Header />

<main>

    {#if articles.length > 0}
        {#each articles as article, i}
            <Article id={i} news={article}  />
        {/each}
    {:else}
        <div class="feed-messages">
            {feedMessage}
        </div>
    {/if}

</main>

<NewFeed sitesList={sites} />

<style>
main {
    display: flex;
    flex-wrap:wrap;
}

.feed-messages {
    width:100%;
    text-align:center;
    font-weight:600;
}
</style>