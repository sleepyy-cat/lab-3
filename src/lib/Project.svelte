<script>
    export let data = {};
</script>
<article>
    <h2>{data.title}</h2>
    <img src={data.image} alt="">
    <p>{data.description}</p>
</article>
<style>
    article {
        display: grid;
        grid-template-rows: subgrid;
        grid-row: span 3;
    }

    article h2 {
        margin: 0;
    }

    img {
        max-width: 100%;
    }
</style>