<script>
  import { groupBy, props, slice } from "ramda";
  import {
    isInIgnored,
    HTMLERRORS,
    getHtmlHintIssues,
    getCodeErrorRules,
    getCodeErrorsByFile
  } from "../utils/utils.js";
  import { fade, fly } from "svelte/transition";
  import { ignoredUrls$ } from "../stores.js";
  import { createEventDispatcher } from "svelte";
  import Icon from "./Icon.svelte";

  export let errors = [];
  export let codeIssues = [];
  $: allErrors = errors.concat(getCodeErrorsByFile(codeIssues));
  $: htmlHintIssues = getHtmlHintIssues(errors);
  const dispatch = createEventDispatcher();
  const viewSource = (url, location, key) => {
    console.log(url, location);

    if (htmlHintIssues.indexOf(key) >= 0) {
      dispatch("viewSource", {
        url,
        location
      });
    } else {
      const snippet = codeIssues.filter(
        x => x.file === url && x.line === location
      )[0].snippet;

      dispatch("viewCode", `// ..........\n${snippet}\n// ..........`);
    }
  };

  $: ERRORS =
    codeIssues && codeIssues.length > 0
      ? (getCodeErrorRules(codeIssues) || []).concat(HTMLERRORS)
      : HTMLERRORS;

  let hiddenRows = {};
  const viewRule = k => {
    window.open();
  };
  const hideShow = key =>
    (hiddenRows[key] = key in hiddenRows ? !hiddenRows[key] : true);
</script>

{#each allErrors as url}
  <div class="mb-3">
    <span class="font-bold mr-2">
      <Icon
        on:click={() => hideShow(url.url)}
        cssClass="inline-block cursor-pointer">
        {#if !hiddenRows[url]}
          <path d="M19 9l-7 7-7-7" />
        {:else}
          <path d="M9 5l7 7-7 7" />
        {/if}
      </Icon>
      Issues found on:
    </span>
    <a class="inline-block align-baseline link" target="_blank" href={url.url}>
      {url.url}
    </a>
  </div>
  {#if !hiddenRows[url.url]}
    <table
      class="table-auto mb-8"
      in:fade={{ y: 100, duration: 400 }}
      out:fade={{ y: -100, duration: 200 }}>
      <thead>
        <tr>
          <th class="w-2/12 px-4 py-2">
            Issues ({Object.keys(url.errors).length})
          </th>
          <th class="w-10/12 px-4 py-2">Locations</th>
        </tr>
      </thead>
      <tbody>
        {#each Object.keys(url.errors) as key}
          <tr>
            <td
              class="whitespace-no-wrap break-all w-2/12 border px-4 py-2
              break-all">
              <Icon
                title={ERRORS.indexOf(key) >= 0 ? 'Error' : 'Warning'}
                cssClass={`inline-block cursor-pointer ${ERRORS.indexOf(key) >= 0 ? 'text-red-600' : 'text-orange-600'}`}>
                {#if ERRORS.indexOf(key) >= 0}
                  <path d="M6 18L18 6M6 6l12 12" />
                {:else}
                  <path d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                {/if}
              </Icon>
              <a
                class="inline-block align-baseline link"
                target="_blank"
                href={htmlHintIssues.indexOf(key) >= 0 ? `https://htmlhint.com/docs/user-guide/rules/${key}` : `https://github.com/nvhoanganh/urlchecker/blob/master/sswcodeauditor/rules/${key}.md`}>
                {key}
              </a>

            </td>
            <td class="w-10/12 border px-4 py-2 break-all">
              <div class="flex flex-wrap">
                {#each slice(0, 49, url.errors[key]) as item}
                  <div
                    class="text-xs mr-2 my-1 uppercase tracking-wider border
                    px-2 border-red-600 hover:bg-red-600 hover:text-white
                    cursor-default whitespace-no-wrap">
                    <a
                      on:click={() => viewSource(url.url, item, key)}
                      href="javascript:void(0)"
                      title="View source">
                      {item}
                    </a>
                  </div>
                {/each}
              </div>
              {#if url.errors[key].length > 50}
                <div
                  class="text-xs mr-2 my-1 tracking-wider px-2 cursor-default">
                  {url.errors[key].length - 50} more..
                </div>
              {/if}
            </td>
          </tr>
        {/each}
      </tbody>
    </table>
  {/if}
{/each}
