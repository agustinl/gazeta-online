<script>
import { onMount, tick } from 'svelte'
import Header from './Header.svelte'
import Feed from './Feed.svelte'
import NewFeed from './NewFeed.svelte'
import imagesLoaded from 'imagesloaded'

$: feeds = [];
$: catch_error = false;

let rss_sites = [];
let sites_list;
let max_news_to_show = 1;
let message = "Loading news feed...";

var masonryEvents = ['load', 'resize'];

const DOMPARSER = new DOMParser().parseFromString.bind(new DOMParser());

const store_sites = (async () => {	
	var sites = localStorage.getItem('gazeta_online');
		sites = JSON.parse(sites);
	return await sites;
})()

onMount(async () => {
	const sites = await store_sites;

	// If no sites...
	if(sites == null) {
		message = "There is no site loaded yet. Put your favorite news sites below.";
		return;
	};

	// If sites
	max_news_to_show = sites.max_news_to_show; // remove and set first array item (n° news per feed)
	delete sites.max_news_to_show;
	const sites_rss = Object.values(sites);
	sites_list = Object.keys(sites);

	(async function() {
		for await (let site of sites_rss) {
			const feed = await getFeed(site, max_news_to_show);
			if (feed == undefined) continue // If no feed because rss site failed to fetch
			feeds.push(feed);
			rssListStatus("sucess", site);
		}
		feeds = feeds;

		await tick();
		waitForImages();

		// Fix issue https://github.com/agustinl/gazeta-online/issues/2
		if (feeds == "") catch_error = true;
	})();
	
})

/**
 * Get info from XML
 */
async function getFeed(rss_site, max_news_to_show) {
	
	try {
		var date = new Date().getFullYear();
		var articles = [];
		var feed_description = "";
		var feedicon = "";
		let reshtml = await fetchSites(rss_site);
		let doc = DOMPARSER(reshtml, "text/xml");		
		let nodes = doc.querySelectorAll('item').length == 0 ? doc.querySelectorAll('entry') : doc.querySelectorAll('item');
		let feed_title = doc.querySelector('title').textContent;
		let feed_link = doc.querySelector('link').textContent == "" ? doc.querySelector('link').getAttribute("href") : doc.querySelector('link').textContent;
			
		if (doc.querySelector('description') != null) {
			feed_description = doc.querySelector('description').textContent
		} else if (doc.querySelector('subtitle')) {
			feed_description = doc.querySelector('subtitle').textContent
		}		

		articles.push(feed_title);
		articles.push(feed_description);
		articles.push(feed_link);

		let news_length = max_news_to_show > nodes.length ? nodes.length : max_news_to_show;

		for (var x = 0; x < news_length; x++) {
				
			var j;
			var title = "";
			var description = "";
			var pubdate = "";
			var categories = [];
			let sourcelink = rss_site;
			var link = nodes[x].querySelector('link').textContent == "" ? nodes[x].querySelector('link').getAttribute("href") : nodes[x].querySelector('link').textContent;

			// If exist, set title
			if(nodes[x].querySelector('title') != null) {
				title = nodes[x].querySelector('title').textContent;
			}			
				
			// If exist, set description
			if (nodes[x].querySelector('content') != null && nodes[x].querySelector('content').textContent.trim() != "") {
				description = nodes[x].querySelector('content').textContent
			} else if (nodes[x].getElementsByTagNameNS("*", "encoded").item(0) != null) {
				description = nodes[x].getElementsByTagNameNS("*", "encoded").item(0).textContent;
			} else if (nodes[x].querySelector('description')) {
				description = nodes[x].querySelector('description').textContent;
			}

			// If exist, set published date
			if(nodes[x].querySelector('pubDate') != null) {
				pubdate = nodes[x].querySelector('pubDate') ? nodes[x].querySelector('pubDate').textContent : "";
				pubdate = pubdate.split(date);
				pubdate = pubdate[0] + date;
			}

			// If exist, set categories
			if(nodes[x].querySelectorAll('category') != null && nodes[x].querySelectorAll('category').length > 0) {
					
				var nodeCategories = nodes[x].querySelectorAll('category');
					
				for (j = 0; j < nodeCategories.length; j++) {
					if(nodeCategories[j].textContent != null && nodeCategories[j].textContent != "") {
						categories.push(nodeCategories[j].textContent);
					}					
				}
			}
				
			let article = {
				title: title,
				link: link,
				description: description,
				categories: categories,
				pubdate: pubdate,
				sourcelink: sourcelink
			}

			articles.push(article);
		}

		return articles;
	} catch (error) {
		rssListStatus("error", rss_site);
		/*throw "Type 2 ++ " + error;*/
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

function resizeMasonryItem(item) {
    /* Get the grid object, its row-gap, and the size of its implicit rows */
    var grid = document.getElementsByClassName('feeds')[0],
        rowGap = parseInt(window.getComputedStyle(grid).getPropertyValue('grid-row-gap')),
        rowHeight = parseInt(window.getComputedStyle(grid).getPropertyValue('grid-auto-rows'));

    /*
     * Spanning for any brick = S
     * Grid's row-gap = G
     * Size of grid's implicitly create row-track = R
     * Height of item content = H
     * Net height of the item = H1 = H + G
     * Net height of the implicit row-track = T = G + R
     * S = H1 / T
     */
    var rowSpan = Math.ceil((item.querySelector('.feed-content').getBoundingClientRect().height + rowGap) / (rowHeight + rowGap));

    /* Set the spanning as calculated above (S) */
    item.style.gridRowEnd = 'span ' + rowSpan;
}

/**
 * Apply spanning to all the masonry items
 *
 * Loop through all the items and apply the spanning to them using 
 * `resizeMasonryItem()` function.
 */
function resizeAllMasonryItems() {
    // Get all item class objects in one list
    var allItems = document.getElementsByClassName('feed');

    /*
     * Loop through the above list and execute the spanning function to
     * each list-item (i.e. each masonry item)
     */
    for (var i = 0; i < allItems.length; i++) {
        resizeMasonryItem(allItems[i]);
    }
}

/**
 * Resize the items when all the images inside the masonry grid 
 * finish loading. This will ensure that all the content inside our
 * masonry items is visible.
 */
function waitForImages() {
	var allItems = document.getElementsByClassName('feed');
	for(var i=0;i<allItems.length;i++){
		imagesLoaded(allItems[i], function( instance ) {
			resizeAllMasonryItems()
		});
	}
}

/**
 * Set a list of rss links with status
 * SUCESS : if rss link works
 * ERROR : if rss link doesnt work
 */
function rssListStatus(status, site) {
	rss_sites.push({
		status: status,
		rss: site
	})
	rss_sites = rss_sites;
}

masonryEvents.forEach(function(event) {
  window.addEventListener(event, resizeAllMasonryItems);
});
</script>

<main>	
	<Header />

	{#await store_sites}
		<img src="img/undraw_newspaper_k72w.svg" alt="" style="width: 300px;margin: 0 auto;display: block;">
		<h2 class="feed-message">{message}</h2>
	{:then sites}
		{#if catch_error}
			<img src="img/undraw_QA_engineers_dg5p.svg" alt="" style="width: 300px;margin: 0 auto 20px;display: block;">
			<h2 class="feed-message">Ups!, an error occurred, try again in a few minutes</h2>
		{:else if feeds != ""}
			<div class="feeds">
			{#each feeds as feed}
				<section class="feed">
					<Feed feed={feed} />
				</section>
			{/each}
			</div>
		{:else}
			<img src="img/undraw_newspaper_k72w.svg" alt="" style="width: 300px;margin: 0 auto;display: block;">
			<h2 class="feed-message">{message}</h2>
		{/if}
		<NewFeed sitesList={sites_list} RSSsites={rss_sites} maxNewsToShow={max_news_to_show} />
	{:catch error}
		<img src="img/undraw_QA_engineers_dg5p.svg" alt="" style="width: 300px;margin: 0 auto 20px;display: block;">
		<h2 class="feed-message">Ups!, an error occurred, try again in a few minutes</h2>
	{/await}	
</main>