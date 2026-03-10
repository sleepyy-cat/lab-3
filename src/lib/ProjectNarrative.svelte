<script>
    import projects from "$lib/projects.json";
    import Scrolly from "svelte-scrolly";
    let scrollyProgress = 0;
    let sorted_projects = projects.sort((a, b) => a.year - b.year);
    let progressPerProject = 100 / sorted_projects.length;
    $: activeProjectIdx = Math.min(sorted_projects.length-1, Math.floor(scrollyProgress / progressPerProject));
</script>
<Scrolly bind:progress={ scrollyProgress }>
	{#each sorted_projects as p}
    <section class=step>
        <div class=step-content>
            <h3>{p.title}</h3>
            <p>{p.story}</p>
        </div>
    </section>
    {/each}
	<svelte:fragment slot="viz">
        <div class="scrolly-wrapper">
            <div class="project-detail">
                <h3>{sorted_projects[activeProjectIdx].year}</h3>
                <img src="{sorted_projects[activeProjectIdx].image}" alt="">
            </div>
        </div>
	</svelte:fragment>
</Scrolly>
<style>
    .scrolly-wrapper {
        width: 100%;
        position: relative;
        left: 50%;
        transform: translateX(-50%);
    }
    .project-detail {
        padding: 2rem;
        width: 100%;
    }
    .step {
        min-height: 80vh;
        padding: 2em;
    }
    .step-content {
        border-left: solid oklch(26.826% 0.1459 284.5);
        padding: 1.5rem 2rem;
    }
    .scrolly-wrapper > .project-detail > img {
        width: 70%;
    }
</style>