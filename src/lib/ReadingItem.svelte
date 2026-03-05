<script>
    export let data = {};
</script>

<article>
    <div>
        <img src={data.image} alt="" />
        <h3>{data.title}</h3>
        <p>
            <i>
                By {data.author}
            </i>
        </p>
        <p>{data.description}</p>
    </div>
</article>

<style>
    article {
        display: grid;
        grid-template-columns: subgrid;
        grid-row: span 3;
    }


    img {
        margin-top: 1em;
        max-width: 100%;
    }
</style>
