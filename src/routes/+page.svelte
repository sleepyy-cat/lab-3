<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";
  import { onMount } from "svelte";
  let githubData = null;
  let loading = true;
  let error = null;
  onMount(async () => {
    try {
      console.log("Page has been mounted!")
      let response = await fetch("https://api.github.com/users/sleepyy-cat");
      console.log(response);
      githubData = await response.json();
      console.log(githubData);
    } catch (err) {
      error = err;
    }
    loading = false;
  })
</script>
<h1>Hanna Chen</h1>
<h2>Hi, I'm Hanna!</h2>
<p>
    I'm a senior at MIT double majoring in math and computer science. I've played piano and violin 
    (mostly classical music) for more than a decade. I also really enjoy listening to music--classical, pop artists like 
    Taylor Swift and Gracie Abrams, and just about anything else as well. In my free time, in addition to music, I also like 
    hanging out with my friends and playing games like DnD and Celeste.
</p>
<!-- svelte-ignore a11y_img_redundant_alt -->
<img src="images/profilepic.jpg" alt="Picture of me in front of the Eiffel Tower" style="max-width: 30%; height: auto;">
{#if loading}
    <p>Loading...</p>
{:else if error}
    <p>Something went wrong: {error.message}</p>
{:else}
    <section>
        <h2>My GitHub Stats</h2>
        <dl>
            <dt>FOLLOWERS</dt>
            <dd>{githubData.followers}</dd>
            <dt>FOLLOWING</dt>
            <dd>{githubData.following}</dd>
            <dt>PUBLIC REPOS</dt>
            <dd>{githubData.public_repos}</dd>
        </dl>
    </section>
{/if}
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
    grid-template-columns: repeat(3, 1fr);
  }

  dt {
      font-size: medium;
      grid-row: 1;
  }

  dd {
      font-size: xx-large;
      grid-row: 2;
      margin-left: 0;
  }
</style>