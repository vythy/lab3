<script>
    import { base } from "$app/paths";
    import { page } from "$app/stores";
    let pages = [
        { url: "/", title: "About" },
        { url: "/projects", title: "Projects" },
        { url: "/cv", title: "Resume" },
        { url: "/contact", title: "Contact" },
        { url: "/meta", title: "Meta"},
        { url: "https://github.com/vythy", title: "Github" },
    ];
    let colorScheme = "light dark";
    let localStorage = globalThis.localStorage ?? {};
    if (localStorage.colorScheme) {
        colorScheme = localStorage.colorScheme;
    }
    let root = globalThis.document?.documentElement;
    $: root?.style.setProperty("color-scheme", colorScheme);
    $: localStorage.colorScheme = colorScheme;
</script>

<label class="color-scheme-switch">
    Theme:
    <select bind:value={colorScheme}>
        <option value="light dark">Automatic</option>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
    </select>
</label>
<nav>
    {#each pages as p}
        <a
            href={p.url.startsWith("https") ? p.url : base + p.url}
            class:current={p.url === "/"
                ? $page.url.pathname === (base + "/")
                : $page.url.pathname.startsWith(base + p.url)}
            target={p.url.startsWith("http") ? "_blank" : null}
        >
            {p.title}
        </a>
    {/each}
</nav>
<slot />

<style>
    nav {
        --border-color: oklch(50% 10% 200 / 40%);
        display: flex;
        border-bottom: 2px solid var(--border-color);
    }

    a {
        flex: 1;
        text-decoration: none;
        color: inherit;
        text-align: center;
        padding: 0.5em;
        margin-top: 0.5em;
        border-bottom-width: 1px;
        border-bottom-style: solid;
        border-bottom-color: oklch(80% 3% 200);
    }

    a:hover {
        border-bottom-width: 0.4em;
        border-bottom-color: var(--color-accent);
        background-color: oklch(from var(--color-accent) 95% 5% h);
    }

    .current {
        border-bottom-width: 0.4em;
        border-bottom-color: var(--border-color);
        padding-bottom: 0.5em;
    }

    .color-scheme-switch {
        position: absolute;
        display: inline-flex;
        top: 1rem;
        right: 1rem;
        gap: 4px;
        font-size: 80%;
    }

</style>
