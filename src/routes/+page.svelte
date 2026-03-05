<script>
    import projects from "$lib/projects.json";
    import Project from "$lib/Project.svelte";
    import reading from "$lib/reading.json";
    import ReadingItem from "$lib/ReadingItem.svelte";
    import { onMount } from "svelte";

    let githubData = null; // This will eventually hold our Github stats
    let loading = true; // This will be true *until* the fetch's promise resolves to a value
    let error = null; // If the API call resulted in an error, it will go into this variable

    async function retrieveGithubData() {
        try {
            console.log("Page has been mounted!");
            let response = await fetch("https://api.github.com/users/vythy");
            console.log(response);
            githubData = await response.json();
            console.log(githubData);
        } catch (err) {
            error = err;
        }
        loading = false;
    }

    onMount(async () => {
        retrieveGithubData();
    });
</script>

<div>
    <h1>Justin Le</h1>

    <p>
        In love with the game and music. Current CS student at MIT. Also avid food
        lover.
    </p>
</div>

<img src="images/electrodog.jpg" alt="electro dawg" />

<div class="github-stats">
    {#if loading}
        <p>Loading...</p>
    {:else if error}
        <p>Something went wrong: {error.message}</p>
    {:else}
        <section>
            <h2>My GitHub Stats</h2>
            <dl>
                <dt>Followers</dt>
                <dd>{githubData.followers}</dd>
                <dt>Following</dt>
                <dd>{githubData.following}</dd>
                <dt>Public Repositories</dt>
                <dd>{githubData.public_repos}</dd>
            </dl>
        </section>
    {/if}
</div>

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

<style>
    dl {
        display: grid;
        grid-template-columns: repeat(4, 1fr);
    }

    dt {
        grid-row: 1;
    }

    dd {
        grid-row: 2;
    }

    .github-stats {
        border: 2px solid;
        border-color: var(--color-accent);
        margin: 2rem;
    }
</style>