<script lang="ts">
  import type { LedgerDirectory, LedgerFile } from "$lib/utils";
  import _ from "lodash";
  import { createEventDispatcher } from "svelte";

  export let files: Array<LedgerDirectory | LedgerFile>;
  export let path: string;
  export let selectedFileName: string;
  export let hasUnsavedChanges: boolean;
  export let root: boolean = true;

  const dispatch = createEventDispatcher();

  function fileName(path: string) {
    return _.last(path.split("/"));
  }

  function join(paths: string[]) {
    return _.filter(paths, (p) => !_.isEmpty(p)).join("/");
  }

  function isOpen(file: LedgerDirectory | LedgerFile) {
    const fullPath = join([path, file.name]);
    return selectedFileName?.startsWith(fullPath);
  }
</script>

<ul class={root && "du-menu du-menu-sm w-full p-0"}>
  {#each files as file}
    {#if file.type != "directory"}
      <li>
        <a
          on:click={() => dispatch("select", file)}
          class={file.name == selectedFileName ? "du-active" : ""}
        >
          <span class="icon is-small">
            <i class="fa-regular fa-file-lines" />
          </span>
          <span title={fileName(file.name)} class="truncate">{fileName(file.name)}</span>
          {#if file.name == selectedFileName && hasUnsavedChanges}
            <span class="ml-1 tag is-danger">unsaved</span>
          {/if}
        </a>
      </li>
    {:else}
      <li>
        <details open={isOpen(file)}>
          <summary>
            <span class="icon is-small">
              <i class="fa-regular fa-folder" />
            </span>
            <span title={file.name} class="truncate">{file.name}</span>
          </summary>
          <svelte:self
            path={join([path, file.name])}
            on:select={(e) => dispatch("select", e.detail)}
            root={false}
            files={file.children}
            {selectedFileName}
            {hasUnsavedChanges}
          />
        </details>
      </li>
    {/if}
  {/each}
</ul>
