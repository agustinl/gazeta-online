<script>
    import CountrySelect from './CountrySelect.svelte'
    import CategorySelect from './CategorySelect.svelte'
    import LanguageSelect from './LanguageSelect.svelte'

    export let sitesList;

    let newsList;
    let newsNumber = 1;
    let storeCountry = [];
    let storeCategory = [];
    let storeLanguage = [];

    if(sitesList != null) {
        newsList        = sitesList.news_list;
        newsNumber      = sitesList.max_news_to_show;
        storeCountry    = sitesList.country;
        storeCategory   = sitesList.category;
        storeLanguage   = sitesList.language;
    }

    function setNewsList(e) {
        e.preventDefault();
        
        var store_sites = {max_news_to_show: newsNumber, news_list: newsList, country: storeCountry, category: storeCategory, language:storeLanguage};
        localStorage.setItem('gazeta_online', JSON.stringify(store_sites));
        location.reload();
    }

    function countrySelected(event) {
        storeCountry = [event.detail.id, event.detail.country];
    }

    function categorySelected(event) {
        storeCategory = [event.detail.id, event.detail.category];
    }

    function languageSelected(event) {
        storeLanguage = [event.detail.id, event.detail.language];
    }

    function reset() {
        newsList        = "";
        newsNumber      = 1;
        storeCountry    = "";
        storeCategory   = "";
        storeLanguage   = "";
    }
</script>

<section id="new-feed">

    <div class="new-feed-box">
        <!-- <input type="text"> -->
        <textarea name="news-list" id="news-list" bind:value={newsList} placeholder="Your favorite news sources separated by commas (eg. theverge.com,wired.com)"></textarea>
        <LanguageSelect on:language={languageSelected} setLanguage={storeLanguage} />
        <div class="separator">
            <span></span> <p>or</p> <span></span>
        </div>
        <CountrySelect on:country={countrySelected} setCountry={storeCountry} />
        <CategorySelect on:category={categorySelected} setCategory={storeCategory} />
        <p>I want to see <input type="number" id="news-number" name="news-number" bind:value={newsNumber} min="1" max="100"> news per feed</p>
        
        <div class="btns">
            <button class="btn-reset" on:click={reset}>Reset</button>
            <button class="btn" on:click={setNewsList}>Set new feed</button>        
        </div>
    </div>

</section>

<style>
#new-feed {
    display:flex;
    align-items:center;
    justify-content:center;
    padding: 100px 0;
}

.new-feed-box {
    width: 30vw;
    display:flex;
    flex-direction:column;
}

.new-feed-box .btns {
    display:flex;
    justify-content:flex-end;
}

.btn-reset {
    border:none;
    background:none;
    color: #808080;
    margin: 10px 30px;
    font-size:14px;
    text-transform:uppercase;
    font-weight:bold;
}

.new-feed-box p {
    font-size:14px;
    margin:0;
}

textarea {
    width: 100%;
    height: 100px;
    resize: vertical;
    padding: 10px 5px;
}

#news-number {
    padding: 5px;
}

.separator {
    display:flex;
    justify-content:space-between;
    align-items:center;
    margin-bottom:20px;
}

.separator span {
    flex-grow:1;
    height:1px;
    background:#000;
}

.separator p {
    padding:0px 10px;
}

@media only screen and (max-width:1700px) {
    .new-feed-box {
        width: 50vw;
    }
}

@media only screen and (max-width:680px) {
    .new-feed-box {
        width: 95vw;
    }
}
</style>