<script lang="ts">
    import { onMount } from "svelte";
    import dayjs from "dayjs";
    import relativeTime from "dayjs/plugin/relativeTime";
    const firehoseStatusUrl = new URL(import.meta.env.PUBLIC_FIREHOSE_STATUS_URL)

    dayjs.extend(relativeTime)

    type HealthData = {
        uptimeMs: number;
        channels: number;
        clients: number;
        messagesPerSecond: number;
        totalMessages: number;
    };

    type NodeStatus = {
        name: string;
        error?: boolean;
        health?: HealthData;
    };


    let loading = true;
    let error = false;
    let statuses: NodeStatus[] = [];

    onMount(async () => {
        loading = true;
        const res = await fetch(firehoseStatusUrl);
        if (!res.ok) {
            loading = false;
            error = true;
        }
        const json = await res.json();
        statuses = json;
        loading = false;
    })

    function prettifyTotalMessages(totalMessages: number) {
        let num = totalMessages;
        let word = "";
        if (totalMessages >= 1_000_000_000) {
            num = totalMessages / 1_000_000_000;
            word = "Billion";
        } else if (totalMessages >= 1_000_000) {
            num = totalMessages / 1_000_000;
            word = "Million";
        }
        
        return `${Number(num.toPrecision(3)).toLocaleString()} ${word}`;
    }
</script>

<div id="app">
    { #if !loading }
        { #if error }
            <span class="error">There was an error connecting with the status service</span>
        { :else }
            <div class="status-cards">
                { #each statuses as status}
                    <div class="status-card">
                        <div class="name">
                            {status.name}

                            <span class={status.error ? "indicator error": "indicator online"}>⬤</span>
                        </div>
                        <div class="data">
                            { #if status.health}
                                Uptime: {dayjs().from(dayjs(new Date().getTime()-status.health.uptimeMs), true)}
                                <br>
                                Channels: {status.health.channels}
                                <br>
                                Messages/s: {status.health.messagesPerSecond}
                                <br>
                                Total Messages: {prettifyTotalMessages(status.health.totalMessages)}
                            { :else }
                                <span class="error">Cannot get status</span>
                            { /if }
                        </div>
                    </div>
                {/each }
            </div>
        { /if }
    { :else }
        Loading...
    { /if }
</div>


<style scoped lang="scss">
    #app {
        min-height: 150px;
    }

    .status-cards {
        max-width: 70vw;
        display: grid;
        grid-template-columns: repeat(auto-fit, 260px);
        gap: 10px;

        .status-card {
            min-width: 80px;
            min-height: 80px;
            padding: 20px;
            background-color: #00000041;
            border-radius: 10px;

            .data {
                color: rgb(170, 170, 170);
            }
        }
    }

    @keyframes pulse {
        25%   { opacity:1; }
        50%  { opacity:0; }
        75% { opacity:1; }
    }

    .error {
        color: red;
    }

    .online {
        color: green;
        animation: pulse 3s infinite;
    }
</style>