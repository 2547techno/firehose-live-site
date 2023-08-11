<script lang="ts">
    import { onMount } from "svelte";
    const firehoseStatusUrl = new URL(import.meta.env.PUBLIC_FIREHOSE_STATUS_URL)

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
</script>

<div id="app">
    { #if !loading }
        { #if error }
            <span class="error">There was an error connecting with the status service</span>
        { :else }
            { #each statuses as status}
                <div class="status-card">
                    <div class="name">
                        {status.name}

                        <span class={status.error ? "indicator error": "indicator online"}>⬤</span>
                    </div>
                    <div class="data">
                        { #if status.health}
                            Uptime: {Math.round(status.health.uptimeMs/1000)}s
                            <br>
                            Channels: {status.health.channels}
                            <br>
                            Messages/s: {status.health.messagesPerSecond}
                            <br>
                            Total Messages: {status.health.totalMessages}
                        { :else }
                            <span class="error">Cannot get status</span>
                        { /if }
                    </div>
                </div>
            {/each }
        { /if }
    { :else }
        Loading...
    { /if }
</div>


<style scoped lang="scss">
    #app {
        min-height: 150px;
    }

    .status-card {
        padding: 20px;
        background-color: #00000041;
        border-radius: 10px;

        .data {
            color: rgb(170, 170, 170);
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