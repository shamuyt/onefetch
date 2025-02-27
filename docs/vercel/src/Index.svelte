<script lang="ts">
  import AsciiPreview from './lib/AsciiPreview.svelte';
  import data from '../../../languages.yaml';
  import type { Languages, Language } from '../../../languages.yaml';
  import { onMount } from 'svelte';
  import { writable, derived } from 'svelte/store';

  let tagName: string;
  let htmlUrl: string;
  let showMenu: boolean;

  const languages: Language[] = Object.entries(data as Languages).map(
    ([name, { type, ascii, colors }]) => ({
      name,
      type,
      ascii,
      colors,
    })
  );

  const languageTypes: string[] = Array.from(
    new Set<string>(Object.values(data as Languages).map(({ type }) => type))
  );

  const filter = writable({
    checkboxes: languageTypes,
  });

  const filteredLanguages = derived(filter, ($filter) => {
    return languages.filter(({ type }) => $filter.checkboxes.includes(type));
  });

  onMount(async () => {
    const response = await fetch(
      'https://api.github.com/repos/o2sh/onefetch/releases/latest'
    );
    const data = await response.json();

    tagName = data.tag_name;
    htmlUrl = data.html_url;
  });
</script>

<header>
  {#if tagName && htmlUrl}
    <div class="banner">
      <small
        >Version {tagName} is available 🎉 Check the
        <a href={htmlUrl}>release notes</a>!!</small>
    </div>
  {/if}
  <h1>Onefetch</h1>
  <p>
    <small>
      <a href="https://github.com/o2sh/onefetch/wiki">wiki</a> |
      <a href="https://github.com/o2sh/onefetch/tree/main/docs/vercel"
        >github</a>
      | Built with ❤ by
      <a href="https://github.com/spenserblack">@spenserblack</a> and
      <a href="https://github.com/o2sh">@o2sh</a></small>
  </p>
</header>
<main>
  <p>
    This page shows you an ASCII preview for all the programming languages
    supported by onefetch. Like the binary, the data is taken from the <a
      href="https://raw.githubusercontent.com/o2sh/onefetch/main/languages.yaml"
      ><code>Languages.yaml</code></a> file and the layout is meant to mimic the
    way the logo would look inside a terminal.
  </p>
  <p>
    Suggestions and PRs are welcome at <a
      href="https://github.com/o2sh/onefetch">github.com/o2sh/onefetch</a>
  </p>
  <div class="title">
    <h3>Languages <small>({$filteredLanguages.length})</small></h3>
    <button class="filter-button" on:click={() => (showMenu = !showMenu)}
      >Filter by type</button>
  </div>

  <div class:hide={!showMenu}>
    <div class="checkbox-group">
      {#each languageTypes as type}
        <label for={type}>
          <input
            id={type}
            type="checkbox"
            value={type}
            bind:group={$filter.checkboxes} />
          {type}
        </label>
      {/each}
    </div>
    <small
      >Note: By default, onefetch will only recognize <strong
        >programming</strong>
      and <strong>markup</strong> types. Use the
      <code>--type</code> flag to configure.</small>
  </div>

  {#each $filteredLanguages as language}
    <AsciiPreview
      name={language.name}
      ansi={language.colors.ansi}
      hex={language.colors.hex}
      ascii={language.ascii}
      chip={language.colors.chip} />
  {/each}
</main>

<style>
  .banner {
    background-color: #e1f6e5;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    text-align: center;
    padding: 0.5rem 0;
  }

  .title {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 3rem;
    margin-bottom: 1.5rem;
  }

  .title h3 {
    margin-bottom: 0;
    margin-top: 0;
  }

  .checkbox-group {
    margin-top: 1.5rem;
  }

  .checkbox-group label {
    width: fit-content;
    text-transform: capitalize;
  }

  .hide {
    display: none;
  }
</style>
