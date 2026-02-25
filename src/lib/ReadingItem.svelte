<script>
    export let data = {};
</script>
<article>
    <img src={data.image} alt="">
    <div class="info">
        <h3>{data.title}</h3>
        <p><i>{data.author}</i></p>
        <p>{data.description}</p>
    </div>
</article>
<style>
    article {
        display: grid;
        grid-template-columns: subgrid;
        grid-column: span 3;
    }

    article h3 {
        margin: 0;
    }

    img {
        max-width: 50%;
    }
</style>