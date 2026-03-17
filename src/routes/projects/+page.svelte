<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "$lib/ProjectNarrative.svelte";
  import Bar from '$lib/Bar.svelte';
  let years = projects.map(proj => proj.year)
  let range = Math.max(...years) - Math.min(...years);
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let rawData = [];
  let wrangled = [];
  let wrangledPercentages = [];

  $: barData = d3.rollups(projects, v => v.length, d => d.year)
    .map(([year, count]) => ({ label: String(year), value: count })) 
    .sort((a, b) => a.label - b.label);

  onMount(async () => {
      rawData = await d3.json('/lab6_example.json');
      let totalLines = d3.sum(rawData, d => d.lines);
      wrangled = d3.rollups(
        rawData,
        v => d3.sum(v, d => d.lines),
        d => d.language
      );
      wrangledPercentages = d3.rollups(
        rawData,
        v => (d3.sum(v, d => d.lines) / totalLines).toFixed(2) * 100 + '%',
        d => d.language
      );
  });
</script>
<svelte:head>
  <title>Projects</title>
</svelte:head>
<h1>{projects.length} Projects over {range} Years</h1>
<div>
  <Bar data={barData} />
</div>
<br>
<p>Scroll down to see my a timeline of my projects and how they've contributed to my professional and personal life</p>

<ProjectNarrative />

<p class="outro">Thanks for scrolling through my project story! Feel free to explore all of the projects at your leisure below.</p>
<div class="projects">
    {#each projects as p}
    <Project data={p} />
    {/each}
</div>
<section>
    <h2>Data wrangling result</h2>
    <pre>{JSON.stringify(wrangled, null, 2)}</pre>
    <h2>Data wrangling percentages result</h2>
    <pre>{JSON.stringify(wrangledPercentages, null, 2)}</pre>
</section>
<style>
  .outro {
    margin-bottom: 30px;
  }
</style>