<script>
  import { base } from '$app/paths';
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import BarHorizontal from '$lib/BarHorizontal.svelte';
  import {
    computePosition,
    autoPlacement,
    offset,
  } from '@floating-ui/dom';

  let width = 1000, height = 600;
  let margin = { top: 40, right: 150, bottom: 60, left: 80 };

  let locData = [];
  let grouped = [];
  let wrangled = [];
  let commits = [];

  onMount(async () => {
    locData = await d3.csv(`${base}/loc.csv`, row => ({
        ...row,
        line: Number(row.line),
        length: Number(row.length),
        depth: Number(row.depth),
        date: new Date(row.date + "T00:00" + row.timezone),
	      datetime: new Date(row.datetime)
    }));
  
    // grouped = d3.rollups(
    //   locData,
    //   v => v.length,
    //   d => d.type
    // );
    
    // wrangled = grouped
    //   .map(([language, count]) => ({ 
    //     label: language, 
    //     value: count 
    //   }))
    //   .sort((a, b) => b.value - a.value);

    commits = d3.groups(locData, d => d.commit).map(([commit, lines]) => {
      let first = lines[0];
      let {author, date, time, timezone, datetime} = first;
      let ret = {
        id: commit,
        url: "https://github.com/vis-society/lab-7/commit/" + commit,
        author, date, time, timezone, datetime,
        hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
        totalLines: lines.length,
        lines: lines
      };
      return ret;
    });
    commits = d3.sort(commits, d => -d.totalLines);
  })
  let usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left
  };
  usableArea.width = usableArea.right - usableArea.left;
  usableArea.height = usableArea.bottom - usableArea.top;
  $: [minDate, maxDate] = d3.extent(commits.map(d => d.date));
  $: maxDatePlusOne = new Date(maxDate);
  $: maxDatePlusOne.setDate(maxDatePlusOne.getDate() + 1);

  $: xScale = d3.scaleTime()
                .domain([minDate, maxDatePlusOne])
                .range([usableArea.left, usableArea.right])
                .nice();

  $: yScale = d3.scaleLinear()
                .domain([24, 0])
                .range([usableArea.bottom, usableArea.top]);

  $: {
    if (commits.length > 0) {
      const [minLines, maxLines] = d3.extent(commits, d => d.totalLines);
      window.rScale = d3.scaleSqrt()
        .domain([minLines, maxLines])
        .range([3, 15]);
    }
  }

  let xAxis, yAxis, yAxisGridlines;
  $: {
    d3.select(xAxis).call(d3.axisBottom(xScale));
    d3.select(yAxis).call(d3.axisLeft(yScale).tickFormat(d => String(d % 24).padStart(2, "0") + ":00"));
    d3.select(yAxisGridlines).call(
      d3.axisLeft(yScale)
        .tickFormat("")
        .tickSize(-usableArea.width)
    );
  }

  let hoveredIndex = -1;
  $: hoveredCommit = commits[hoveredIndex] ?? hoveredCommit ?? {};

  let tooltipPosition = {x: 0, y: 0};
  let commitTooltip;

  let clickedCommits = [];
  async function dotInteraction(index, evt) {
    let hoveredDot = evt.target;
    if (evt.type === "mouseenter") {
      hoveredIndex = index;
      tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
        strategy: "fixed",
        middleware: [
          offset(5),
          autoPlacement()
        ],
      });
    } else if (evt.type === "mouseleave") {
      hoveredIndex = -1;
    } else if (evt.type === "click") {
      let commit = commits[index]
      if (!clickedCommits.includes(commit)) {
        // Add the commit to the clickedCommits array
        clickedCommits = [...clickedCommits, commit];
      }
      else {
          // Remove the commit from the array
          clickedCommits = clickedCommits.filter(c => c !== commit);
      }
    }
  }
  $: selectedLines = (clickedCommits.length > 0 ? clickedCommits.flatMap(d => d.lines) : locData);
  $: selectedCounts = d3.rollup(
      selectedLines,
      v => v.length,
      d => d.type
  );
  $: allTypes = Array.from(new Set(locData.map(d => d.type)));
  $: wrangled = allTypes.map(type =>  ({ label: String(type), value: selectedCounts.get(type) || 0 }));
</script>
<h1>Meta</h1>
<p>Meta page to visualize project data.</p>
<h3>Commits by time of day</h3>
<svg viewBox="0 0 {width} {height}">
	<g class="dots">
  {#each commits as commit, index }
    <circle
      on:mouseenter={evt => dotInteraction(index, evt)}
      on:mouseleave={evt => dotInteraction(index, evt)}
      on:click={ evt => dotInteraction(index, evt) }
      class:selected={ clickedCommits.includes(commit) }
      cx={ xScale(commit.datetime) }
      cy={ yScale(commit.hourFrac) }
      r={ rScale(commit.totalLines) }
      fill="steelblue"
    />
  {/each}
  </g>
  <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
  <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
  <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
</svg>
<dl class="info tooltip" bind:this={commitTooltip} hidden={hoveredIndex === -1} style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px">
	<dt>COMMIT</dt>
	<dd><a href="{ hoveredCommit.url }" target="_blank">{ hoveredCommit.id }</a></dd>

	<dt>DATE</dt>
	<dd>{ hoveredCommit.datetime?.toLocaleString("en", {dateStyle: "full"}) }</dd>

  <dt>TIME</dt>
	<dd>{ hoveredCommit.datetime?.toLocaleString("en", {timeStyle: "short"}) }</dd>

	<dt>AUTHOR</dt>
	<dd>{ hoveredCommit.author }</dd>

	<dt>LINES</dt>
	<dd>{ hoveredCommit.totalLines }</dd>
</dl>
<BarHorizontal data={wrangled} title={clickedCommits.length > 0 ? "Lines of Code: Selected commits" : "Lines of Code: Website Breakdown"}/>
<style>
	svg {
		overflow: visible;
	}
  .gridlines {
    stroke-opacity: .2;
  }

  .dots {
    fill-opacity: 50%;
  }

  circle {
    transition: 200ms;
    &:hover {
      fill:darkgreen;
    }
  }

  dl.info {
    display: grid;
    grid-template-columns: auto 1fr;
    margin: 0;
    transition-duration: 500ms;
    transition-property: opacity, visibility;

    &[hidden]:not(:hover, :focus-within) {
      opacity: 0;
      visibility: hidden;
    }
  }

  dl.info dt {
    font-weight: lighter;
  }

  .tooltip {
    position: fixed;
    top: 1em;
    left: 1em;
    background-color: oklch(100% 0% 0 / 80%);
    box-shadow: 2px 2px lightgrey;
    border-radius: 4px;
    backdrop-filter: blur(4px);
    padding: 1em 1em;
  }

  .selected {
    fill: var(--color-accent);
  }

  h3 {
    text-align: center;
  }
</style>