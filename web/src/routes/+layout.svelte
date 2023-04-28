<script lang="ts">
  import "../app.css";
  import type { LayoutData } from "./$types";
  import { invalidate, goto } from "$app/navigation";
  import { page } from "$app/stores";
  export let data: LayoutData;

  $: deleteConfirm = false;
  $: id = $page.params.id || "";
  async function deleteChat(chatID: string) {
    const response = await fetch("/api/chat/" + chatID, { method: "DELETE" });
    if (response.status === 200) {
      toggleDeleteConfirm();
      await goto("/");
      await invalidate("/api/chat/");
    } else {
      console.error("Error " + response.status + ": " + response.statusText);
    }
  }

  function toggleDeleteConfirm() {
    deleteConfirm = !deleteConfirm;
  }

  function timeSince(datestring: string) {
    const date = new Date(datestring);
    const seconds = Math.floor((Date.now() - date.getTime()) / 1000);

    let interval = seconds / 31536000;

    if (interval > 1) {
      return Math.floor(interval) + " years";
    }
    interval = seconds / 2592000;
    if (interval > 1) {
      return Math.floor(interval) + " months";
    }
    interval = seconds / 86400;
    if (interval > 1) {
      return Math.floor(interval) + " days";
    }
    interval = seconds / 3600;
    if (interval > 1) {
      return Math.floor(interval) + " hours";
    }
    interval = seconds / 60;
    if (interval > 1) {
      return Math.floor(interval) + " minutes";
    }
    return Math.floor(seconds) + " seconds";
  }

  function truncate(str: string, n: number) {
    return str.length > n ? str.slice(0, n - 1) + "..." : str;
  }
</script>

<aside
  id="default-sidebar"
  class="fixed top-0 left-0 z-40 w-80 h-screen transition-transform bg-slate-950/80 -translate-x-full sm:translate-x-0"
  aria-label="Sidebar"
>
  <div class="h-full px-3 py-4 overflow-y-auto">
    <ul class="space-y-6">
      <li class="mb-4 w-full flex flex-col items-center">
        <a href="/" class="bg-slate-800 p-4 border border-slate-700/70 rounded-lg w-full font-medium text-center focus:ring-2 focus:ring-slate-700 focus:ring-offset-2">Home page</a>
      </li>
      {#each data.chats as chat}
        <li>
          <a
            href={"/chat/" + chat.id}
            class={`flex items-center p-2 text-base font-normal rounded-lg relative ${id === chat.id ? "bg-gradient-to-t from-slate-800/80 to-slate-700" : "bg-gradient-to-t from-slate-900/80 to-slate-800"}`}
          >
            <div class="flex flex-col w-full">
              <div class="flex items-center justify-between w-full gap-4 h-8">
                <div>
                  <span class="font-medium text-slate-200">{chat.model}</span>
                  <span class="ml-1">•</span>
                  <span class="ml-1">{timeSince(chat.created) + " ago"}</span>
                </div>
                {#if $page.params.id === chat.id}
                  {#if deleteConfirm}
                    <button
                      name="confirm-delete"
                      class="btn btn-ghost btn-sm"
                      on:click|preventDefault={() => deleteChat(chat.id)}
                    >
                      <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M4.5 12.75l6 6 9-13.5" />
                      </svg>                    
                    </button>
                    <button
                      name="cancel-delete"
                      class="btn btn-ghost btn-sm"
                      on:click|preventDefault={toggleDeleteConfirm}
                    >
                      <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
                      </svg>                    
                    </button>
                  {:else}
                    <button
                      class="btn btn-ghost btn-sm"
                      on:click|preventDefault={toggleDeleteConfirm}
                    >
                      <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-5 h-5">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M14.74 9l-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 01-2.244 2.077H8.084a2.25 2.25 0 01-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 00-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 013.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 00-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 00-7.5 0" />
                      </svg>                    
                    </button>
                  {/if}
                {/if}
              </div>
              <p class="font-regular mt-1 text-sm">{truncate(chat.subtitle, 100)}</p>
            </div>
          </a>
        </li>
      {/each}
    </ul>
  </div>
</aside>

<div class="p-4 sm:ml-80 bg-slate-900 h-full relative">
  <slot />
</div>
