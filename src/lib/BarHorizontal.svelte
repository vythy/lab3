<script>
    import * as d3 from 'd3';

    export let data = [];
    export let title = "";

    let width = 680;
    let height = 240;

    let margin = { top: 40, right: 110, bottom: 80, left: 60 };
    let innerWidth = width - margin.left - margin.right;
    let innerHeight = height - margin.top - margin.bottom;

    let xAxis;
    let yAxis;

    $: xMax = d3.max(data, d => d.value) || 1;

    $: xScale = d3.scaleLinear()
        .domain([0, xMax])
        .range([0, innerWidth]);

    $: yScale = d3.scaleBand()
        .domain(data.map(d => d.label))
        .range([0, innerHeight])
        .padding(0.2);

    $: colorScale = d3.scaleOrdinal()
        .domain(["svelte", "css", "js", "html"])
        .range(["#c7d9e5", "#8fb3cc", "#cfe7b3", "#9fd27a"]);

    $: maxBar = d3.greatest(data, d => d.value);

    $: if (xAxis && yAxis) {
        d3.select(xAxis).call(
            d3.axisBottom(xScale)
                .ticks(7)
                .tickFormat(d3.format("d"))
        );

        d3.select(yAxis).call(d3.axisLeft(yScale));
    }

    $: if (xAxis && yAxis) {
        d3.select(xAxis).call(
            d3.axisBottom(xScale)
            .ticks(Math.min(d3.max(data, d => d.value), 10))
        );
        d3.select(yAxis).call(d3.axisLeft(yScale));
    }
</script>

<div class="container">
    <svg viewBox="0 0 {width} {height}">
        <text
            x={margin.left + innerWidth / 2}
            y={28}
            text-anchor="middle"
            class="chart-title">
            {title}
        </text>

        <g transform="translate({margin.left}, {margin.top})">
            {#each data as d}
                <rect
                    x={0}
                    y={yScale(d.label)}
                    width={xScale(d.value)}
                    height={yScale.bandwidth()}
                    fill={colorScale(d.label)}
                />
            {/each}
            {#if maxBar}
                <rect
                    x={0}
                    y={yScale(maxBar.label)}
                    width={xScale(maxBar.value)}
                    height={yScale.bandwidth()}
                    fill="none"
                    stroke="black"
                    stroke-width="2"
                />
                <!-- <line
                    x1={xScale(maxBar.value) / 2}
                    x2={xScale(maxBar.value) / 2}
                    y1={yScale(maxBar.label) + yScale.bandwidth()}
                    y2={yScale(maxBar.label) + yScale.bandwidth() + 20}
                    stroke="currentcolor"
                    stroke-width="1"
                /> -->

                <text
                    x={xScale(maxBar.value) + 52}
                    y={yScale(maxBar.label) + yScale.bandwidth()/2}
                    text-anchor="middle"
                    class="annotation">
                    Most lines of code
                </text>
            {/if}
        </g>

        <g
            transform="translate({margin.left}, {margin.top + innerHeight})"
            bind:this={xAxis}>
        </g>

        <g
            transform="translate({margin.left}, {margin.top})"
            bind:this={yAxis}>
        </g>

        <text
            x={margin.left + innerWidth / 2}
            y={height - 18}
            text-anchor="start"
            class="axis-label">
            Lines of Code
        </text>

        <text
            x={-(margin.top + innerHeight / 2)}
            y={-margin.left + 48}
            text-anchor="middle"
            transform="rotate(-90)"
            class="axis-label">
            Programming Language
        </text>
    </svg>

    <ul class="legend">
        {#each data as d}
            <li style="--color: {colorScale(d.label)}">
                <span class="swatch"></span>
                {d.label} <em>({d.value})</em>
            </li>
        {/each}
    </ul>
</div>

<style>
    .container {
        display: flex;
        align-items: center;
        gap: 0.5rem;
        margin-top: 1.5rem;
    }

    svg {
        max-width: 100%;
        height: auto;
        display: block;
        overflow: visible;
        font-family: sans-serif;
    }

    .chart-title {
        font-size: 1rem;
        font-weight: 700;
        fill: currentColor;
    }

    .axis-label {
        font-size: 0.8rem;
        fill: #777;
    }

    .annotation {
        font-size: 0.75rem;
        fill: currentColor;
        font-style: italic;
    }

    .legend {
        list-style: none;
        padding: 0;
        margin: 0;
        display: grid;
        gap: 0.4rem;
        font-size: 0.9rem;
    }

    .legend li {
        display: flex;
        align-items: center;
        gap: 0.5rem;
    }

    .swatch {
        display: inline-block;
        width: 0.9rem;
        height: 0.9rem;
        background-color: var(--color);
        flex: 0 0 auto;
    }

    em {
        font-style: italic;
    }
</style>