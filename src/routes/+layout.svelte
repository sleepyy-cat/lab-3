<script>
    import { base } from "$app/paths";
    import { page } from "$app/stores";
    let pages = [
        {url: "/", title: "About"},
        {url: "/projects", title: "Projects"},
        {url: "/resume", title: "Resume"},
        {url: "/contact", title: "Contact"},
        {url: "https://github.com/sleepyy-cat", title: "Github"},
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
    <select bind:value={ colorScheme }>
        <option value="light dark">Automatic</option>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
    </select>
</label>
<nav>
    {#each pages as p}
    <a href={p.url.startsWith('https') ? p.url : base + p.url} 
    class:current={p.url === "/"? $page.url.pathname === (base + "/") : $page.url.pathname.startsWith(base + p.url)}
    target={p.url.startsWith("http") ? "_blank": null}>
    {p.title}
    </a>
    {/each}
</nav>
<style>
    nav {
    --border-color: oklch(50% 10% 200 / 40%);
    display: flex;
    margin-bottom: 10px;
    border-bottom: 2px solid var(--border-color);
    }

    nav a {
    flex: 1;
    text-decoration: none;
    color: inherit;
    text-align: center;
    padding: 0.5em;
    border-bottom: 2px solid var(--border-color);
    }

    nav a.current {
    padding-bottom: 0.2em;
    border-bottom: 4px solid var(--border-color); 
    }

    nav a:hover {
    border-bottom: 0.4em solid var(--color-accent);
    background-color: color-mix(in oklch, var(--color-accent), canvas 85%);
    padding-bottom: 0.2em;
    }
</style>
<slot />