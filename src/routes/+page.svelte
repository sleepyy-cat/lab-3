<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";
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