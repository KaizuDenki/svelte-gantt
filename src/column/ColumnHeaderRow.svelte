<script lang="ts">
    import { createEventDispatcher, getContext } from 'svelte';

    const dispatch = createEventDispatcher();

    import type { SvelteGanttDateAdapter } from '../utils/date';
    import { startOf, getDuration } from '../utils/date';

    const { from, to, width } = getContext('dimensions');
    const { dateAdapter }: { dateAdapter: SvelteGanttDateAdapter } = getContext('options');

    export let header;
    export let baseWidth;
    export let baseDuration;

    export let columnWidth;
    $: {
        header.duration = getDuration(header.unit, header.offset);
        const duration = header.duration;
        const ratio = duration / baseDuration;
        columnWidth = baseWidth * ratio;
    }

    export let columnCount;
    $: {
        columnCount = Math.ceil($width / columnWidth);
        if(!isFinite(columnCount)){
            console.error('columnCount is not finite');
            columnCount = 0;
        }
    }

    let _headers = [];
    $: {
        const headers = [];
        let headerTime = startOf($from, header.unit);

        let headerDuration = header.duration;
        let calculatedWidth = columnWidth;
        let d = new Date($from).getDate();
        let m = new Date($from).getMonth();
        for (let i = 0; i < columnCount; i++) {
            m++;
            let nextMonth = new Date(new Date($from).setMonth(m));
            if (header.unit === "month") {
                let e = new Date(nextMonth.setDate(0)).getDate();
                let headerDurationDay = i > 0 ? e : e - d + 1
                headerDuration = headerDurationDay * 24 * 60 * 60 * 1000;
                calculatedWidth = (columnWidth / 30) * headerDurationDay;

                headerTime = startOf(nextMonth.valueOf(), header.unit);
                console.log(new Date(headerTime))
            }
            headers.push({
                width: Math.min(calculatedWidth, $width),
                label: dateAdapter.format(headerTime, header.format),
                from: headerTime,
                to: headerTime + headerDuration,
                //to: headerTime + header.duration,
                unit: header.unit,
            });
            if (header.unit !== "month") {
                headerTime += headerDuration;
            }
            //headerTime += header.duration;
        }
        _headers = headers;
    }
</script>

<div class="column-header-row">
    {#each _headers as _header}
        <div class="column-header-cell" class:sticky={header.sticky} style="width:{_header.width}px" on:click="{() => dispatch('dateSelected', { from: _header.from, to: _header.to, unit: _header.unit })}">
            <div class="column-header-cell-label">{_header.label || 'N/A'}</div>
        </div>
    {/each}
</div>
<style>
    .column-header-row {
        box-sizing: border-box;
        white-space: nowrap;
        height: 32px;
    }

    .column-header-cell {
        display: inline-block;
        height: 100%;
        box-sizing: border-box;
        text-overflow: clip;
        /* vertical-align: top; */
        text-align: center;

        display: inline-flex;
        justify-content: center;
        align-items: center;
        font-size: 1em;    
        font-size: 14px;
        font-weight: 300;
        transition: background 0.2s;

        cursor: pointer;     
        user-select: none;

        border-right: #efefef 1px solid;
        border-bottom: #efefef 1px solid;
    }

    .column-header-cell:hover {
        background: #f9f9f9;
    }

    .column-header-cell.sticky > .column-header-cell-label {
        position: sticky;
        left: 1rem;
    }
</style>