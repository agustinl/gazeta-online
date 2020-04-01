<script>
export let RSSsites;
export let sitesList;
export let maxNewsToShow;

$: arr_new_feed = [];

function setNewFeeds() {
    if(sitesList == "" || sitesList == null) { return }

    sitesList = sitesList.toString();
    
    var arr_tmp_feed = new Array();
        arr_tmp_feed = sitesList.split(",");

    arr_new_feed.push(maxNewsToShow);

    arr_tmp_feed.forEach(site => {
        site = site.indexOf("http") < 0 ? "http://" + site : site;
        arr_new_feed.push(site);
    })

    arr_new_feed = arr_new_feed;

    setCookie('gazeta_online', arr_new_feed);
}

function setCookie(key, value) {
    var expires = new Date();
    expires.setTime(expires.getTime() + (10 * 365 * 24 * 60 * 60));
    document.cookie = key + '=' + value + ';expires=' + expires.toUTCString();
    location.reload();
}
</script>

<div id="new-feed">
    <div>
        <textarea name="newfeed" id="newfeed" bind:value={sitesList} placeholder="Put your favorite websites separated by a comma (eg. https://www.theverge.com/, https://www.wired.com/)"></textarea>
        <div>
            <p>I want to see <input type="number" id="news-amount" name="news-amount" bind:value={maxNewsToShow} min="1" max="100"> news per feed</p>
            <button on:click={setNewFeeds}>New Feed!</button>
        </div>
    </div>
    {#if RSSsites != ""}
    <div>
        <ul class="feeds-list">
            {#each RSSsites as site}
                {#if site.status == "error"}
                    <li class={site.status}>
                        <svg xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-alert-circle" width="24" height="24" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" fill="none" stroke-linecap="round" stroke-linejoin="round">
                        <path stroke="none" d="M0 0h24v24H0z"/>
                        <circle cx="12" cy="12" r="9" />
                        <line x1="12" y1="8" x2="12" y2="12" />
                        <line x1="12" y1="16" x2="12.01" y2="16" />
                        </svg> <a href={site.rss} target="_blank">{site.rss}</a>
                    </li>
                {:else}
                    <li class={site.status}><svg xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-check" width="24" height="24" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" fill="none" stroke-linecap="round" stroke-linejoin="round">
                        <path stroke="none" d="M0 0h24v24H0z"/>
                        <polyline points="20 7 10 17 5 12" />
                        </svg> <a href={site.rss} target="_blank">{site.rss}</a>
                    </li>
                {/if}
            {/each}
        </ul>
    </div>
    {/if}
</div>

<footer>
    <h4>by <a href="https://agustinl.dev/" target="_blank" rel="noopener">agustínl</a></h4>
    <h5>Font by <a href="https://rsms.me/inter/" target="_blank" rel="noopener">@rsms</a>, Icons by <a href="https://github.com/tabler/tabler-icons" target="_blank" rel="noopener">tabler-icons</a> and Ilustration by <a href="https://undraw.co/" target="_blank" rel="noopener">unDraw</a></h5>
</footer>
