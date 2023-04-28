<script lang="ts">
  import type { PageData } from "./$types";
  import { invalidate, goto } from "$app/navigation";
  import { page } from "$app/stores";
  import { onMount, afterUpdate } from "svelte";

  export let data: PageData;

  $: isLoading = false;
  $: startDate = new Date(data.chat.created);
  $: history = data.chat.history;

  $: prompt = "";

  async function askQuestion() {
    if (prompt) {
      const data = new URLSearchParams();
      data.append("prompt", prompt);

      const eventSource = new EventSource(
        "/api/chat/" + $page.params.id + "/question?" + data.toString()
      );

      history = [
        ...history,
        {
          type: "human",
          data: {
            content: prompt,
          },
        },
        {
          type: "ai",
          data: {
            content: "",
          },
        },
      ];

      prompt = "";

      eventSource.addEventListener("message", (event) => {
        history[history.length - 1].data.content += event.data;
      });

      eventSource.addEventListener("close", async () => {
        eventSource.close();
        await invalidate("/api/chat/" + $page.params.id);
      });

      eventSource.onerror = async (error) => {
        eventSource.close();
        history[history.length - 1].data.content = "A server error occurred.";
        await invalidate("/api/chat/" + $page.params.id);
      };
    }
  }

  async function handleKeyDown(event: KeyboardEvent) {
    if (event.key === "Enter" && event.ctrlKey) {
      await askQuestion();
    }
  }

  async function createSameSession() {
    const newData = await fetch(
      `/api/chat/?model=${data.chat.params.model_path}&temperature=${data.chat.params.temperature}&top_k=${data.chat.params.top_k}` +
        `&top_p=${data.chat.params.top_p}&max_length=${data.chat.params.max_tokens}&context_window=${data.chat.params.n_ctx}` +
        `&repeat_last_n=${data.chat.params.last_n_tokens_size}&repeat_penalty=${data.chat.params.repeat_penalty}` +
        `&n_threads=${data.chat.params.n_threads}&init_prompt=${data.chat.history[0].data.content}`,

      {
        method: "POST",
        headers: {
          accept: "application/json",
        },
      }
    ).then((response) => response.json());
    await invalidate("/api/chat/");
    await goto("/chat/" + newData);
  }

  document.addEventListener("keydown", async (event) => {
    if (event.key === "n" && event.altKey) {
      await createSameSession();
    }
  });
</script>

<div
  class="max-w-4xl mx-auto h-full max-h-screen relative"
  on:keydown={handleKeyDown}
>
  <div class="flex justify-between items-center">
    <h1 class="text-4xl font-semibold text-white inline-block mr-2">
      Model : {data.chat.params.model_path}
    </h1>
    <div
      class="tooltip tooltip-bottom"
      data-tip="This will create a new chat session with the same parameters."
    >
      <button
        type="button"
        disabled={isLoading}
        class="p-2 px-4 bg-slate-800 hover:bg-slate-700 focus:ring-2 focus:ring-slate-700 focus:ring-offset-2 font-medium rounded-lg mr-2 mt-5 mb-5 inline-block"
        class:loading={isLoading}
        on:click|preventDefault={() => createSameSession()}
      >
        New
      </button>
    </div>
  </div>
  <h4 class="text-lg font-medium mb-5">
    Started on {startDate.toLocaleString("en-US")}
  </h4>
  <div class="overflow-y-auto rounded-xl border border-slate-700 p-4 h-[calc(97vh-14rem)] mb-8">
    <div class="h-max">
      {#each history as question}
        {#if question.type === "human"}
          <div class="chat chat-end my-4">
            <div
              class="bg-gradient-to-t from-slate-800 to-slate-700 text-white p-2 px-4 rounded-xl whitespace-pre-line text-lg"
            >
              {question.data.content}
            </div>
          </div>
        {:else if question.type === "ai"}
          <div class="chat chat-start mt-4 mb-12">
            <div
              class="bg-gradient-to-t from-indigo-700 to-indigo-500 shadow-lg shadow-indigo-600/30 text-white p-2 px-4 rounded-xl whitespace-pre-line text-lg"
            >
              {#if question.data.content === ""}
                <div
                  class="radial-progress animate-spin"
                  style="--value:70; --size:2rem;"
                />
              {/if}
              {question.data.content}
            </div>
          </div>
        {:else if question.type === "system"}
          <div
            class="inline-flex mx-auto text-center font-light text-md text-gray-300 my-4 p-2 px-4"
          >
            {question.data.content}
          </div>
        {/if}
      {/each}
    </div>
  </div>
  <form
    on:submit|preventDefault={askQuestion}
    class="items-center w-full flex justify-center gap-4"
  >
    <input
      autofocus
      name="question"
      class="w-full px-4 p-2.5 rounded-xl text-lg bg-slate-800 focus-visible:ring-0"
      disabled={isLoading}
      placeholder="Ask a question..."
      bind:value={prompt}
    />
    <button
      type="submit"
      disabled={isLoading}
      class="p-2.5 px-4 text-slate-200 bg-slate-800 rounded-xl text-lg focus:ring-2 focus:ring-slate-700 focus:ring-offset-2"
      class:loading={isLoading}
      
    >
      Send
    </button>
  </form>
</div>
