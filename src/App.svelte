<script>
	import { onMount, afterUpdate } from 'svelte'
	import Header from './Header.svelte'
	import Feed from './Feed.svelte'
	import NewFeed from './NewFeed.svelte'
	import Masonry from 'masonry-layout'

	const DOMPARSER = new DOMParser().parseFromString.bind(new DOMParser());

	$: feeds = [];
	let sites = "";
	let listOfRSSsites = [];
	let message = "Loading news feed...";

	onMount(async () => {
		sites = getCookie('gazeta_online');

		if(sites == null) {
			message = "There is no site loaded yet. Put your favorite news sites below.";
			return
		} else {
			await createFeeds(sites);
		}
	})

	afterUpdate(() => {
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

	async function createFeeds(sites) {	
		for (var i = 1; i < sites.length; i++) {
			getRSS(sites[i])
			.then(rsslink => {
				if (rsslink == undefined) { return }
				getNews(rsslink, sites[0]) /* sites[0], first element in array, number of news amount */
				.then(data => {				
					listOfRSSsites.push({
						status: "sucess",
						rss: rsslink
					});
					listOfRSSsites = listOfRSSsites;
				})
			})
		}
	}

	async function getNews(rsslink, maxNewsToShow) {
		try {
			var date = new Date().getFullYear();
			var articles = [];
			var feeddescription = "";
			var feedicon = "";
			let reshtml = await fetch(rsslink)
			.then(function(response) {
				if (response.status != 404) {
					return response.text();
				}
			})
			let doc = DOMPARSER(reshtml, "text/xml");		
			let nodes = doc.querySelectorAll('item').length == 0 ? doc.querySelectorAll('entry') : doc.querySelectorAll('item');		

			let feedtitle = doc.querySelector('title').textContent;
			
			if (doc.querySelector('description') != null) {
				feeddescription = doc.querySelector('description').textContent
			} else if (doc.querySelector('subtitle')) {
				feeddescription = doc.querySelector('subtitle').textContent
			}

			let feedlink = doc.querySelector('link').textContent == "" ? doc.querySelector('link').getAttribute("href") : doc.querySelector('link').textContent;

			articles.push(feedtitle);
			articles.push(feeddescription);
			articles.push(feedlink);

			for (var x = 0; x < maxNewsToShow; x++) {
				
				var j;
				var title = "";
				var description = "";
				var pubdate = "";
				var categories = [];
				let sourcelink = rsslink;
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

			feeds.push(articles);
			feeds = feeds;

			return feeds;
		} catch (e) {
			listOfRSSsites.push({
				status: "error",
				rss: rsslink
			});
			listOfRSSsites = listOfRSSsites;
			console.error(e);
			return
		}
	}

	async function getRSS(site) {
		try {
			var rsslink;
			
			if (site.includes("rss")) {			
				rsslink = site;
			} else if (site.includes("reddit.com/")) {
				rsslink = site + ".rss";
			} else {
				site = !site.endsWith("/") ? site + "/" : site;
				let response = await fetch(site);
				let reshtml = await response.text();
				let doc = DOMPARSER(reshtml, 'text/html')
				
				if(doc.querySelector('link[type="application/rss+xml"]') == null) {
					return
				}
				
				var feedUrl = doc.querySelector('link[type="application/rss+xml"]').href;
				rsslink = feedUrl.split(location.origin + "/");
				rsslink = rsslink.length == 1 ? rsslink[0] : site + rsslink[1];
			}

			return rsslink;
		} catch (e) {
			listOfRSSsites.push({
				status: "error",
				rss: site
			});
			listOfRSSsites = listOfRSSsites;
			console.error(e);
			return
		}
	}

	function getCookie(key) {
		var keyValue = document.cookie.match('(^|;) ?' + key + '=([^;]*)(;|$)');
		return keyValue ? keyValue[2].split(",") : null;
	}
</script>

<main id="feed">
	
	<Header />

	<div class="feeds">
		{#each feeds as feed}
			<section class="feed">
				<Feed feed={feed} />
			</section>
		{:else}
			<img src="img/undraw_newspaper_k72w.svg" alt="" style="width: 300px;margin: 0 auto;display: block;">
			<h2 class="feed-message">{message}</h2>
		{/each}
	</div>
	
	<NewFeed listOfRSSsites={listOfRSSsites} />
	
</main>

<style>
</style>