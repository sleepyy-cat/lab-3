<script>
    import * as d3 from 'd3';

    export let width = 1000, height = 300;
    export let margin = { top: 40, right: 150, bottom: 60, left: 80 };
    let innerWidth = width - margin.left - margin.right;
    let innerHeight = height - margin.top - margin.bottom;
    export let data = [];

    let usableArea = {
        top: margin.top,
        right: width - margin.right,
        bottom: height - margin.bottom,
        left: margin.left
    }

    $: xScale = d3.scaleTime()
        .domain(d3.extent(data, d => d.date))
        .range([usableArea.left, usableArea.right]);

    $: yScale = d3.scaleLinear()
        .domain([0, d3.max(data, d => d.count)])
        .range([usableArea.bottom, usableArea.top])
        .nice();

    $: line = d3.line()
        .x(d => xScale(d.date))
        .y(d => yScale(d.count))
        .curve(d3.curveBumpX);

    let xAxis, yAxis;
    $: if (xAxis && yAxis) {
      d3.select(xAxis).call(d3.axisBottom(xScale).tickFormat(d3.timeFormat("%b %d")));
      d3.select(yAxis).call(d3.axisLeft(yScale));
    }

    let hoveredDay = null;
    $: dayRegions = (() => {
        // if there's no data, there are no regions!
        if (data.length === 0) return [];
        return data.map((d, i, arr) => {
            // get the previous date, if it exists
            const prev = arr[i - 1];
            // get the next date, if it exists
            const next = arr[i + 1];
            // define the left side of the region, which might be the edge of the axis if this is the first date
            const left = prev ? new Date((d.date.getTime() + prev.date.getTime()) / 2) : d.date;
            // define the right side of the region, which might be the edge of the axis if this is the last date
            const right = next ? new Date((d.date.getTime() + next.date.getTime()) / 2) : d.date;
            
            return {
                date: d.date,
                weekday: d.date.toLocaleString("en", { weekday: "long" }), // e.g. "Monday"
                x: xScale(left),
                width: xScale(right) - xScale(left),
            };
        });
    })();
</script>
<h3>Lines Edited {hoveredDay ? `on ${hoveredDay}s` : "by Day"}</h3>
<svg viewBox="0 0 {width} {height}">
    <path
        d={line(data)}
        fill="none"
        stroke="steelblue"
        stroke-width="2"
    />
    {#each data as d}
        {@const isHighlighted = d.date.toLocaleString("en", { weekday: "long" }) === hoveredDay}
        <circle
            cx={xScale(d.date)}
            cy={yScale(d.count)}
            r={isHighlighted ? 5 : 3}
            fill={isHighlighted? "var(--color-accent)" : "steelblue"}
        />
        {#if isHighlighted}
            <text
                x={xScale(d.date)}
                y={usableArea.top + 15}
                text-anchor="middle"
                font-size="12"
                fill="var(--color-accent)"
            >
                {Math.round(d.count)}
            </text>
        {/if}
    {/each}
    <g transform="translate({0}, {margin.top + innerHeight})" bind:this={xAxis} />
    <g transform="translate({margin.left}, {0})" bind:this={yAxis} />
    <text
        x={usableArea.left + (usableArea.right - usableArea.left) / 2}
        y={height - 15}
        text-anchor="middle"
        class="axis-label">
        Date
    </text>

    <!-- y-axis label -->
    <text
        x={-(usableArea.top + (usableArea.bottom - usableArea.top) / 2)}
        y={25}
        text-anchor="middle"
        transform="rotate(-90)"
        class="axis-label">
        Number of Lines Edited
    </text>

    {#each dayRegions as region}
        {#if region.weekday === hoveredDay}
            <rect
                x={region.x}
                y={usableArea.top}
                width={region.width}
                height={usableArea.bottom - usableArea.top}
                fill="var(--color-accent)"
                opacity="0.2"
            />
        {/if}
    {/each}

    {#each dayRegions as region}
        <rect
            x={region.x}
            y={usableArea.top}
            width={region.width}
            height={usableArea.bottom - usableArea.top}
            fill="transparent"
            on:mouseenter={() => hoveredDay = region.weekday}
            on:mouseleave={() => hoveredDay = null}
        />
    {/each}
</svg>
<style>
    svg {
        overflow: visible;
    }
    h3 {
        text-align: center;
    }
    .axis-label {
        font-weight: bold;
        font-size: 0.8em;
        fill: currentColor;
    }
</style>