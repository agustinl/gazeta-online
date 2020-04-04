<script>
export let RSSsites;
export let sitesList;
export let maxNewsToShow;

const DOMPARSER = new DOMParser().parseFromString.bind(new DOMParser());

let store_sites;
let arr_tmp_rss = [];

/**
 * If textarea not empty, convert text to string
 * Then divide it by comma
 * Validate if sites included in array contains "https"
 * @getRss Get RSS link
 * Save in localStorage
 */
async function setNewFeeds() {
	
    if(sitesList == "" || sitesList == null) { return }

    var tmp_sites = sitesList.toString();
    var arr_tmp_sites = new Array();
        arr_tmp_sites = tmp_sites.split(",");        


    (async function() {
		for await (let tmp_site of arr_tmp_sites) {
            if(!tmp_site.startsWith("https://")) {
                tmp_site = tmp_site.startsWith("http") ? tmp_site.replace("http", "https") : "https://" + tmp_site
            }
            var tmp_rss_site = await getRSS(tmp_site);

            if (tmp_rss_site == undefined) tmp_rss_site = tmp_site;
            
            arr_tmp_rss.push(tmp_rss_site);
        }

        store_sites = {"max_news_to_show": maxNewsToShow};
        arr_tmp_sites.forEach((key, i) => store_sites[key] = arr_tmp_rss[i]);
        localStorage.setItem('gazeta_online', JSON.stringify(store_sites));
        location.reload();
    })();
}
/**
 * Get rss link
 * If the page include "rss" or "feed" word, dont do anything
 * If include the word "reddit", add to the site url the extension ".rss"
 * If not, search in DOM site the meta tag "application/rss+xml"
 * If not exist this meta, return with undefined
 */
async function getRSS(site) {
	
	try {
		var tmp_rss_site;
			
		if (site.includes("rss") || site.includes("feed")) {			
			tmp_rss_site = site;
		} else if (site.includes("reddit.com/")) {
			tmp_rss_site = site + ".rss";
		} else {
			site = !site.endsWith("/") ? site + "/" : site;
			let reshtml = await fetchSites(site);

			// If site not exist, or invalid domain, or 404
			if (reshtml == undefined) return;

			let doc = DOMPARSER(reshtml, 'text/html');

			// If not have rss link tag
			if (doc.querySelector('link[type="application/rss+xml"]') == null) {
				rssListStatus("error", site);
				return "";
			}
				
			var link = doc.querySelector('link[type="application/rss+xml"]').href;
			tmp_rss_site = link.split(location.origin + "/");
			tmp_rss_site = tmp_rss_site.length == 1 ? tmp_rss_site[0] : site + tmp_rss_site[1];
        }

		return tmp_rss_site;
	} catch (error) {
		return "";
	}
}
/**
 * Fetch sites and catch if errors
 */
async function fetchSites(site) {	
	const response = await fetch(site).then(function(response) {
		if (response.ok) {
			return response.text()
		} else {
		console.log('Fetch problem with ' + site + " :: " + response.status);
		}
	})
	.catch(function(error) {
		console.log('Fetch problem with ' + site + " :: " + error.message);
	})

	return response;
}
</script>

<div id="new-feed">
    <div>
        <textarea name="newfeed" id="newfeed" bind:value={sitesList} placeholder="Put your favorite websites separated by a comma (eg. https://www.theverge.com/, https://www.wired.com/)"></textarea>
        {#if RSSsites != ""}
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
        {/if}
        <div>
            <p>I want to see <input type="number" id="news-amount" name="news-amount" bind:value={maxNewsToShow} min="1" max="100"> news per feed</p>
            <button on:click={setNewFeeds}>New Feed!</button>
        </div>
    </div>    
</div>

<footer>
    <h4>by <a href="https://agustinl.dev/" target="_blank" rel="noopener">agustínl</a> — <a href="https://github.com/agustinl/gazeta-online" target="_blank" rel="noopener">GitHub</a></h4>
    <h5>Font by <a href="https://rsms.me/inter/" target="_blank" rel="noopener">@rsms</a>, Icons by <a href="https://github.com/tabler/tabler-icons" target="_blank" rel="noopener">tabler-icons</a> and Ilustration by <a href="https://undraw.co/" target="_blank" rel="noopener">unDraw</a></h5>
</footer>
