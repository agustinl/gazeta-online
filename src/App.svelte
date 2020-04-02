<script>
import { onMount, afterUpdate } from 'svelte'
import Header from './Header.svelte'
import Feed from './Feed.svelte'
import NewFeed from './NewFeed.svelte'
import Masonry from 'masonry-layout'

const DOMPARSER = new DOMParser().parseFromString.bind(new DOMParser());

$: feeds = [];
$: catch_error = false;
let rss_sites = [];
let sites_list;
let max_news_to_show = 1;
let message = "Loading news feed...";

const sites_cookies = (async () => {
	const sites = await getCookie('gazeta_online');
    return await sites;
})()

onMount(async () => {
	const sites = await sites_cookies;
	
	// If no sites...
	if(sites == null) {
		message = "There is no site loaded yet. Put your favorite news sites below.";
		return;
	};

	// If sites
	max_news_to_show = sites.shift(); // remove and set first array item (n° news per feed)
	sites_list = sites; // array without first item

	(async function() {
		for await (let site of sites) {
			const rss_site = await getRSS(site);
			if (rss_site == undefined) continue // If no rss_site, jump to next site
			const feed = await getFeed(rss_site, max_news_to_show);
			feeds.push(feed);
			rssListStatus("sucess", rss_site);
		}
		feeds = feeds;

		// Fix issue https://github.com/agustinl/gazeta-online/issues/2
		if (feeds == "") catch_error = true;
	})();
	
})

afterUpdate(async () => {
	window.addEventListener("DOMSubtreeModified", function () {
		var elem = document.querySelector('.feeds');
		if (elem) {
			var msnry = new Masonry(elem, {
				// options
				itemSelector: '.feed'
			});
		}
	});	
})

async function getRSS(site) {
	try {
		var rss_site;
			
		if (site.includes("rss") || site.includes("feed")) {			
			rss_site = site;
		} else if (site.includes("reddit.com/")) {
			rss_site = site + ".rss";
		} else {
			site = !site.endsWith("/") ? site + "/" : site;
			let reshtml = await fetchSites(site);

			// If site not exist, or invalid domain, or 404
			if (reshtml == undefined) return;

			let doc = DOMPARSER(reshtml, 'text/html');

			// If not have rss link tag
			if (doc.querySelector('link[type="application/rss+xml"]') == null) {
				rssListStatus("error", site);
				return undefined;
			}
				
			var link = doc.querySelector('link[type="application/rss+xml"]').href;
			rss_site = link.split(location.origin + "/");
			rss_site = rss_site.length == 1 ? rss_site[0] : site + rss_site[1];
		}

		return rss_site;
	} catch (error) {
		rssListStatus("error", site);
		/*throw "Type 1 ++ " + error;*/
	}
}

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
		rssListStatus("error", site);
	})

	return response;

}

function getCookie(key) {
	var keyValue = document.cookie.match('(^|;) ?' + key + '=([^;]*)(;|$)');
	return keyValue ? keyValue[2].split(",") : null;
}

function rssListStatus(status, site) {
	rss_sites.push({
		status: status,
		rss: site
	})
	rss_sites = rss_sites;
}
</script>

<main id="feed">
	
	<Header />

	{#await sites_cookies}
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

<style>
</style>