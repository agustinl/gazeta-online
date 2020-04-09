<script>
    import moment from 'moment'

    export let id;
    export let news;

    var { content } = news;
    if(content != null && content != "") {
        content.includes("[+") ? content = content.split("[+")[0] : content;
    }

    const { source_id, name } = news.source;
</script>

<article class="news" id={id}>

    <div class="news-head">
        <h4> <svg xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-rss" width="18" height="24" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" fill="none" stroke-linecap="round" stroke-linejoin="round">
                <path stroke="none" d="M0 0h24v24H0z"/>
                <circle cx="5" cy="19" r="1" />
                <path d="M4 4a16 16 0 0 1 16 16" />
                <path d="M4 11a9 9 0 0 1 9 9" />
            </svg>{name}</h4>
        <pubDate>{moment(`${news.publishedAt}`).format('MMMM Do, YYYY')}</pubDate>
    </div>

    <h3>{news.title}</h3>
    {#if news.description}
        <p class="short-description">{news.description}</p>
    {/if}

    <img src={news.urlToImage} alt={name}>

    {#if news.content}
        <p>{content} <a href={news.url} target="_blank" rel="noopener">Read more</a></p>
    {/if}
        

</article>

<style>
h3 {
    font-weight: 800;
    margin: 0px 0 20px;
}

h4 {
    font-weight: 600;
    line-height:1;
    display:flex;
    align-items:center;
}

p {
    font-size: 16px;
}

p.short-description {
    font-size: 14px;
    color: #888;
    font-weight: 400;
}

.news-head {
    display:flex;
    align-items:center;
    justify-content:space-between;
    margin-bottom: 20px;
}

svg {
    margin-right: 5px;
}

article {
    width: 18%;
    margin: 1%;
    padding: 20px 20px 10px;
    border: solid 1px #e2e2e2;
    box-shadow: 0 1px 2px 0 rgba(0,0,0,.1);
    background: #FFF;
    border-radius: 3px;
    margin-bottom: 20px;
}

article img {
    display: block;
    max-width: 100%;
    height: auto;
    margin: 0 auto 10px;
}

article time {
    font-size: 14px;
    color: #000;
    font-weight: 500;
    text-align: right;
    display: block;
    width: 100%;
}

pubDate {    
    font-size: 14px;
    color: #888;
    font-weight: 300;
}

@media only screen and (max-width: 1680px) {
    article {
        width: 23%;
    }
}

@media only screen and (max-width: 1280px) {
    textarea {
        width: 500px;
    }
    article {
        width: 31%;
    }
}

@media only screen and (max-width: 980px) {
    article {
        width: 48%;
    }
}

@media only screen and (max-width: 680px) {
    article {
        width: 100%;
    }
}
</style>