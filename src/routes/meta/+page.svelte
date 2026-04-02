<script>
    import { base } from "$app/paths";
    import { onMount } from "svelte";
    import * as d3 from "d3";
    import BarHorizontal from "$lib/BarHorizontal.svelte";
    import {
        computePosition,
        autoPlacement,
        offset,
    } from '@floating-ui/dom';


    let locData = [];
    let languageData = [];
    let commits = [];
    let clickedCommits = [];

    let commitTooltip;
    let tooltipPosition = {x: 0, y: 0};
    let cursor = {x: 0, y: 0};
    let hoveredIndex = -1;
    
    let margin = { top: 0, right: 0, bottom: 0, left: 0};
    let width = 1000, height = 600;

    let usableArea = {
        top: margin.top,
        right: width - margin.right,
        bottom: height - margin.bottom,
        left: margin.left
    };
    usableArea.width = usableArea.right - usableArea.left;
    usableArea.height = usableArea.bottom - usableArea.top;

    let xAxis, yAxis, yAxisGridlines;
    
    async function dotInteraction (index, evt) {
        let hoveredDot = evt.target;
        if (evt.type === "mouseenter") {
            hoveredIndex = index;
            cursor = {x: evt.x, y: evt.y};
            tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
                strategy: "fixed", 
                middleware: [
                    offset(5),
                    autoPlacement() 
                ],
            });        }
        else if (evt.type === "mouseleave") {
            hoveredIndex = -1
        }
        else if (evt.type === "click") {
            let commit = commits[index]
            if (!clickedCommits.includes(commit)) {
                clickedCommits = [...clickedCommits, commit];
            }
            else {
                clickedCommits = clickedCommits.filter(c => c !== commit);
            }
        }
    }

    onMount(async () => {
        locData = await d3.csv(`${base}/loc.csv`, row => ({
            ...row,
            line: Number(row.line),
            depth: Number(row.depth),
            length: Number(row.length),
            date: new Date(row.date + "T00:00" + row.timezone),
            datetime: new Date(row.datetime)
        }));
        
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

        languageData = d3.rollups(
            locData,
            v => v.length,
            d => d.type
        )
            .map(([label, value]) => ({ label, value }))
    });

    $: [minDate, maxDate] = d3.extent(commits.map(d => d.date));
    $: maxDatePlusOne = new Date(maxDate);
    $: maxDatePlusOne.setDate(maxDatePlusOne.getDate() + 1);

    $: xScale = d3.scaleTime()
                .domain([minDate, maxDatePlusOne])
                .range([0, width])
                .nice();

    $: yScale = d3.scaleLinear()
                .domain([24, 0])
                .range([height, 0]);

    $: rScale = d3.scaleSqrt()
                .domain(d3.extent(commits.map(l => l.totalLines)))
                .range([5, 30])

    $: tScale = d3.scaleLinear()
                .domain(d3.extent(commits.map(l => l.totalLines)))
                .range([0.5, 1])

    $: hoveredCommit = commits[hoveredIndex] ?? hoveredCommit ?? {};

    $: selectedLines = (clickedCommits.length > 0 ? clickedCommits.flatMap(d => d.lines) : locData);
    $: selectedCounts = d3.rollup(
        selectedLines,
        v => v.length,
        d => d.type
    );
    $: allTypes = Array.from(new Set(locData.map(d => d.type)));
    $: barData = allTypes.map(type =>  ({ label: String(type), value: selectedCounts.get(type) || 0 }));

    
    $: {
	    d3.select(xAxis).call(d3.axisBottom(xScale));
	    d3.select(yAxis).call(d3.axisLeft(yScale));
        d3.select(yAxis).call(d3.axisLeft(yScale).tickFormat(d => String(d % 24).padStart(2, "0") + ":00"));
        d3.select(yAxisGridlines).call(
            d3.axisLeft(yScale)
            .tickFormat("")
            .tickSize(-usableArea.width)
	    );
    }
</script>

<svelte:head>
    <title>Meta</title>
</svelte:head>

<h1>Meta</h1>

<h2>Commits by Time of Day</h2>

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
                fill-opacity={ tScale(commit.totalLines) }
                fill="steelblue"
            />
        {/each}
    </g>
    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
</svg>
<dl class="info tooltip" hidden={hoveredIndex === -1}  style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px" bind:this={commitTooltip}>
    <dt>Commit</dt>
    <dd><a href="{ hoveredCommit.url }" target="_blank">{ hoveredCommit.id }</a></dd>

    <dt>Date</dt>
    <dd>{ hoveredCommit.datetime?.toLocaleString("en", {dateStyle: "full"}) }</dd>

    <dt>Time</dt>
    <dd>{ hoveredCommit.time?.toLocaleString("en", {dateStyle: "full"})}</dd>

    <dt>Author</dt>
    <dd>{ hoveredCommit.author }</dd>

    <dt>Lines</dt>
    <dd>{ hoveredCommit.totalLines }</dd>

</dl>
<BarHorizontal data={barData} title = {clickedCommits.length > 0 ? "Lines of Code: Selected commits" : "Lines of Code: Website Breakdown"} />

<style>
	svg {
		overflow: visible;
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
        gap: 0.3em 0.8em;
        margin: 0;
        padding: 0.6em 0.8em;
        background: rgba(230, 225, 225, 0.9);
        border-radius: 0.4em;
        box-shadow: 0 6px 18px rgba(0, 0, 0, 0.12);
        backdrop-filter: blur(4px);
        color: black;
        font-size: 0.9rem;

        transition-duration: 500ms;
        transition-property: opacity, visibility;

        &[hidden]:not(:hover, :focus-within) {
            opacity: 0;
            visibility: hidden;
        }
    }

    dl.info dt, 
    dl.info dd {
        margin: 0;
        font-size: inherit;
    }

    dl.info dt {
        opacity: 0.6;
        font-weight: 600;
        text-transform: uppercase;
    }

    .tooltip {
        position: fixed;
    }

    .selected {
        fill: var(--color-accent);
    }

    h2 {
        margin-bottom: 1.5em;
        text-align: center;
    }
</style>


<BarHorizontal data={languageData} />