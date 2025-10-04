<script>
  import Button from "$lib/components/ui/button/button.svelte";
  import Input from "$lib/components/ui/input/input.svelte";
  import Card from "$lib/components/ui/card/card.svelte";
  import { cn } from "$lib/utils.js";

  let msgs = [];
  let inputText = "";

  const ws = new WebSocket("ws://127.0.0.1:3000");

  ws.onopen = () => {
    ws.send("A new client has connected.");
    ws.onmessage = (ev) => {
      const message = typeof ev.data === "string" ? ev.data : "";
      msgs = [...msgs, { text: message, sender: "them" }];
    };
  };

  function send() {
    if (!inputText.trim()) return;
    ws.send(inputText);
    msgs = [...msgs, { text: inputText, sender: "me" }];
    inputText = "";
  }

  let chatContainer;
  $: chatContainer?.scrollTo({ top: chatContainer.scrollHeight, behavior: "smooth" });

  // Reference to inner input element
  let inputRef;
</script>

<Card class="w-full max-w-4xl h-[80vh] mx-auto mt-10 p-6 flex flex-col gap-4 shadow-xl rounded-3xl bg-white">
  <!-- Messages -->
  <div bind:this={chatContainer} class="flex-1 flex flex-col gap-3 overflow-y-auto p-2">
    {#each msgs as msg}
      <div class={cn(
        "px-6 py-4 rounded-2xl max-w-[75%] break-words text-lg",
        msg.sender === "me" ? "bg-blue-500 text-white self-end" : "bg-gray-200 text-gray-800 self-start"
      )}>
        {msg.text}
      </div>
    {/each}
  </div>

  <!-- Input + Send -->
  <div class="flex gap-3">
    <Input
      bind:this={inputRef}       <!-- reference to inner input -->
      bind:value={inputText}
      placeholder="Type a message..."
      class="flex-1"
      on:keydown={(e) => {
        if (e.key === "Enter") send(); 
      }}
    />
    <Button on:click={send} variant="default">
      Send
    </Button>
  </div>
</Card>
