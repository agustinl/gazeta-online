<script>
    import moment from 'moment'
</script>

<header>
    <div class="title">
        <h1>Gazeta online</h1>
    </div>
    <div class="sub-title">
        <h2>Free news reader. All in one place.</h2>
        <time>{moment().format('dddd, MMMM Do, YYYY')}</time>
    </div>
    <div class="nav">
        <a href="#new-feed" class="btn">New Feed</a>
    </div>
</header>

<style>
header {
    display:flex;
    align-items:center;
    justify-content:center;

    padding: 100px 0;
}

.sub-title {
    padding: 0 20px;
    margin: 0 20px;
    border-left: solid 1px #000;
}

h1 {
    font-weight:800;
    margin: 10px 0;
}

h2 {
    font-weight:300;
}

time {    
    color: #888;
}

@media only screen and (max-width: 820px) {
    header {
        flex-wrap:wrap;
    }

    .sub-title {
        margin: 20px;
    }
}

@media only screen and (max-width: 700px) {
    header {
        flex-direction:column;
        padding: 50px 0;
    }

    .sub-title {
        padding: 0;
        margin: 30px 0;
        border:none;
    }

    h1 {
        margin:0;
    }
}

@media only screen and (max-width: 400px) {
    header {
        text-align:centeR;
    }
}
</style>