<script>
  import { base } from '$app/paths';
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import BarHorizontal from '$lib/BarHorizontal.svelte';

  let locData = [];
  let wrangled = [];

  onMount(async () => {
    locData = await d3.csv(`${base}/loc.csv`, row => ({
        ...row,
        line: Number(row.line),
        length: Number(row.length),
        depth: Number(row.depth)
    }));
  
    const grouped = d3.rollups(
      locData,
      v => v.length,
      d => d.type
    );
    
    wrangled = grouped
      .map(([language, count]) => ({ 
        label: language, 
        value: count 
      }))
      .sort((a, b) => b.value - a.value);
  })
</script>
<h1>Meta</h1>
<p>Meta page to visualize project data.</p>
<BarHorizontal data={wrangled} />