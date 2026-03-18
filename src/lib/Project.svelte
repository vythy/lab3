<script>
    export let data = {};
</script>

<article>
    <h2>{data.title}</h2>
    <img src={data.image} alt="" />
    
    <p><b>{data.year} | </b> {data.description}</p>
</article>

<style>
    article {
        display: grid;
        grid-template-rows: subgrid;
        grid-row: span 3;
    }

    h2 {
        margin-top: 1em;
    }

    img {
        max-width: 100%;
    }
</style>
