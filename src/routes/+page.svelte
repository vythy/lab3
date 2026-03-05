<script>
    import projects from "$lib/projects.json";
    import Project from "$lib/Project.svelte";
    import reading from "$lib/reading.json";
    import ReadingItem from "$lib/ReadingItem.svelte";
</script>

<h1>Justin Le</h1>

<p>
    In love with the game and music. Current CS student at MIT. Also avid food
    lover.
</p>

<img src="images/electrodog.jpg" alt="electro dawg" />

<h2>Latest Projects</h2>
<div class="projects">
    {#each projects.slice(0, 3) as p}
        <Project data={p} />
    {/each}
</div>

<h2>What I'm Reading</h2>
<div class="reading">
    {#each reading as r}
        <ReadingItem data={r} />
    {/each}
</div>
