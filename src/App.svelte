<script>
    import { onMount, beforeUpdate } from 'svelte';
    import Header from './Header.svelte'
    import Article from './Article.svelte'
    import NewFeed from './NewFeed.svelte'
    import Footer from './Footer.svelte'

    let sites;
    let storedSites;
    let showingNewsMessage = "";
    $: feedMessage = "There is no site loaded yet. Put your favorite news sites below."    
    $: articles = [];

    onMount(async () => {
        if (sites == null) return;

        if(sites.category[0] || sites.country[0]) {
            var url = `https://newsapi.org/v2/top-headlines?pageSize=${sites.max_news_to_show}&country=${sites.country[0]}&category=${sites.category[0]}&apiKey=API_KEY`
        } else if(sites.news_list) {
            var url = `https://newsapi.org/v2/everything?pageSize=${sites.max_news_to_show}&domains=${sites.news_list}&language=${sites.language[0]}&apiKey=API_KEY`
        } else if (sites.language[0] && !sites.news_list) {
            feedMessage = 'To choose a language, it is necessary to place your news sources';
            return;
        }
        
        var req = new Request(url);        
        
        feedMessage = `<div class="sk-cube-grid">
        <div class="sk-cube sk-cube1"></div>
        <div class="sk-cube sk-cube2"></div>
        <div class="sk-cube sk-cube3"></div>
        <div class="sk-cube sk-cube4"></div>
        <div class="sk-cube sk-cube5"></div>
        <div class="sk-cube sk-cube6"></div>
        <div class="sk-cube sk-cube7"></div>
        <div class="sk-cube sk-cube8"></div>
        <div class="sk-cube sk-cube9"></div>
        </div> Loading news...`;

        articles = await fetch(req).then(function(response) {
            return response.json()
        })
        .catch(function(error) {
            console.log('Hubo un problema con la petición Fetch:' + error.message);
        });

        if(articles.status == "ok" && articles.totalResults > 0) {
            articles = articles.articles;
            showingNewsMessage = setShowingNewsMessage(sites.category[1], sites.country[1], sites.max_news_to_show, sites.news_list, sites.language[1]);
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
    
    function setShowingNewsMessage(category, country, max_news_to_show, news, language) {

        var message;

        if(category && country) {
            message = `Showing <strong>${country}</strong> news about <strong>${category}</strong>`
        } else if (category) {
            message = `Showing <strong>${category}</strong> news`
        } else if (country) {            
            message = `Showing news from <strong>${country}</strong>`
        } else if (news && language) {
            message = `Showing news from <strong>${news}</strong> in <strong>${language}</strong>`
        } else if (news) {
            message = `Showing news from <strong>${news}</strong>`
        } else {
            message = `Showing news in <strong>${language}</strong>`
        }
        return message;
    }
</script>

<Header />

{#if showingNewsMessage}
    <div class="watching-news-from">
        <h3>{@html showingNewsMessage}</h3>
    </div>
{/if}

<main>

    {#if articles.length > 0}
        {#each articles as article, i}
            <Article id={i} news={article}  />
        {/each}
    {:else}
        <div class="feed-messages">
            {@html feedMessage}
        </div>
    {/if}

</main>

<NewFeed sitesList={sites} />

<Footer />

<style>
main {
    display: flex;
    flex-wrap:wrap;
}

.feed-messages {
    width:100%;
    text-align:center;
    font-weight:600;
    padding: 100px 20px 0;
}

.watching-news-from {
    padding: 50px 20px;
    text-align:center;
}
</style>