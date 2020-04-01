<script>
import { onMount, afterUpdate } from 'svelte'
import Header from './Header.svelte'
import Feed from './Feed.svelte'
import NewFeed from './NewFeed.svelte'
import Masonry from 'masonry-layout'

const DOMPARSER = new DOMParser().parseFromString.bind(new DOMParser());

$: feeds = [];
let RSSsites = [];
let sitesList;
let maxNewsToShow = 1;
let message = "Loading news feed...";

const cookiesSites = (async () => {
	const sites = await getCookie('gazeta_online');
    return await sites;
})()

onMount(async () => {
	const sites = await cookiesSites;
	if(sites == null) {
		message = "There is no site loaded yet. Put your favorite news sites below.";
		return
	}

	maxNewsToShow = sites.shift();
	sitesList = sites;

	(async function() {
		for await (let site of sites) {
			const RSSsite = await getRSS(site);
			if (RSSsite == undefined) {
				setRSSList("error", site);
				continue
			;}
			const feed = await getFeed(RSSsite, maxNewsToShow)
			feeds.push(feed);
			setRSSList("sucess", RSSsite);
		}
		feeds = feeds;
	})();
	
})

afterUpdate(async () => {
	const sites = await cookiesSites;
	if(sites == null || sites == "") { return }

	const grid = document.querySelector('.feeds')
	const msnry = new Masonry(grid, {
		itemSelector: '.feed'
	})
	msnry.once('layoutComplete', () => {
		grid.classList.add('load')
	})
	msnry.layout();
})

async function getRSS(site) {
	try {
		var RSSsite;
			
		if (site.includes("rss") || site.includes("feed")) {			
			RSSsite = site;
		} else if (site.includes("reddit.com/")) {
			RSSsite = site + ".rss";
		} else {
			site = !site.endsWith("/") ? site + "/" : site;
			let response = await fetch(site);
			let reshtml = await response.text();
			let doc = DOMPARSER(reshtml, 'text/html')
				
			if (doc.querySelector('link[type="application/rss+xml"]') == null) return
				
			var link = doc.querySelector('link[type="application/rss+xml"]').href;
			RSSsite = link.split(location.origin + "/");
			RSSsite = RSSsite.length == 1 ? RSSsite[0] : site + RSSsite[1];
		}

		return RSSsite;
	} catch (error) {
		setRSSList("error", site);
		throw "Type 1 ++ " + error;
	}
}

async function getFeed(RSSsite, maxNewsToShow) {
	try {
		var date = new Date().getFullYear();
		var articles = [];
		var feeddescription = "";
		var feedicon = "";
		let reshtml = await fetch(RSSsite)
			.then(function(response) {
				if (response.status != 404) {
					return response.text();
				}
			})
		let doc = DOMPARSER(reshtml, "text/xml");		
		let nodes = doc.querySelectorAll('item').length == 0 ? doc.querySelectorAll('entry') : doc.querySelectorAll('item');
		let feedtitle = doc.querySelector('title').textContent;
		let feedlink = doc.querySelector('link').textContent == "" ? doc.querySelector('link').getAttribute("href") : doc.querySelector('link').textContent;
			
		if (doc.querySelector('description') != null) {
			feeddescription = doc.querySelector('description').textContent
		} else if (doc.querySelector('subtitle')) {
			feeddescription = doc.querySelector('subtitle').textContent
		}		

		articles.push(feedtitle);
		articles.push(feeddescription);
		articles.push(feedlink);

		let newsLength = maxNewsToShow > nodes.length ? nodes.length : maxNewsToShow;

		for (var x = 0; x < newsLength; x++) {
				
			var j;
			var title = "";
			var description = "";
			var pubdate = "";
			var categories = [];
			let sourcelink = RSSsite;
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
		setRSSList("error", RSSsite);
		throw "Type 2 ++ " + error;
	}
}

function getCookie(key) {
	var keyValue = document.cookie.match('(^|;) ?' + key + '=([^;]*)(;|$)');
	return keyValue ? keyValue[2].split(",") : null;
}

function setRSSList(status, site) {
	RSSsites.push({
		status: status,
		rss: site
	})
	RSSsites = RSSsites;
}
</script>

<main id="feed">
	
	<Header />

	{#await cookiesSites}
		<img src="img/undraw_newspaper_k72w.svg" alt="" style="width: 300px;margin: 0 auto;display: block;">
		<h2 class="feed-message">{message}</h2>
	{:then sites}
		{#if feeds != ""}
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
		<NewFeed sitesList={sitesList} RSSsites={RSSsites} maxNewsToShow={maxNewsToShow} />
	{:catch error}
		<img src="img/undraw_QA_engineers_dg5p.svg" alt="" style="width: 300px;margin: 0 auto 20px;display: block;">
		<h2 class="feed-message">Ups!, an error occurred, try again in a few minutes</h2>
	{/await}
	
	
</main>

<style>
</style>