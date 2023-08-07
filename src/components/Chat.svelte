<script lang="ts">
    import { onDestroy, onMount } from "svelte";

    const firehoseDemoUrl = new URL(import.meta.env.PUBLIC_FIREHOSE_DEMO_URL)

    export let height: string = "auto";
    export let width: string = "auto";

    type Message = {
        emotes: string
        message: string
        channel: string
        from: string
        userColor: string
        alternateBgc: boolean
    }

    let messages: Message[] = []
    let ws: WebSocket | undefined;
    let alternateBgc = false;
    const MAX_MESSEAGES = 15;

    onMount(() => {
        ws = new WebSocket(firehoseDemoUrl)

        ws.addEventListener("open", () => {
            messages.unshift({
                emotes: "",
                message: "Connected to service!",
                channel: "System",
                from: "",
                userColor: "",
                alternateBgc: false
            })
            
            messages = messages;
        })

        ws.addEventListener("message", (e) => {
            const message: Message = {
                ...JSON.parse(e.data),
                alternateBgc
            };
            alternateBgc = !alternateBgc
            if (messages.length >= MAX_MESSEAGES) {
                messages.splice(messages.length-1,1);
            }
            messages.unshift(message)
            messages = messages;
        })
    })

    onDestroy(() => {
        ws?.close()
        ws = undefined;
        messages = [];
    })
</script>

<div class="chat" style="height: {height}; width: {width}">
    { #each messages as message}
    <div class={ message.alternateBgc ? "alternate-bg message" : "message" }>
        <span class="channel">#{ message.channel }</span>
        <strong style="color: { message.userColor }">{ message.from }</strong>: { message.message }
    </div>
    {/each}
</div>

<style scoped>
.chat {
    border: var(--white) solid 2px;
    border-radius: 10px;
    padding: 20px;
    display: flex;
    flex-direction: column-reverse;
    overflow: hidden;
}

.message {
    padding: 7px 5px 7px 5px;
}

.alternate-bg {
    background-color: #ffffff08;
}

.channel {
    color: rgb(160, 160, 160);
}
</style>