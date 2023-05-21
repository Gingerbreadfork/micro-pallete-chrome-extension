<script>
  import { writable, derived } from "svelte/store";
  import websites from "./lib/websites.json";

  let searchTerm = writable("");
  let actionURL = writable("");
  let actionText = writable("visit");
  let selectionMode = writable("website");
  let searchInput;

  const actions = {
    "-s": "searchUrl",
    "-l": "loginUrl",
    "-v": "baseUrl",
    search: "searchUrl",
    login: "loginUrl",
    visit: "baseUrl",
    // Add more actions as needed
  };

  const filteredWebsites = derived(searchTerm, ($searchTerm) => {
    const [websiteName] = $searchTerm.split(" ");
    return websiteName.length > 0
      ? websites.filter(
          (website) =>
            website.websiteName &&
            typeof website.websiteName === "string" &&
            website.websiteName
              .toLowerCase()
              .startsWith(websiteName.toLowerCase()) &&
            website.websiteName.toLowerCase() !== websiteName.toLowerCase()
        )
      : [];
  });

  const actionsList = derived(searchTerm, ($searchTerm) => {
    const [websiteName, actionName] = $searchTerm.split(" ");
    const website = websites.find(
      (website) =>
        website.websiteName &&
        website.websiteName.toLowerCase() === websiteName.toLowerCase()
    );
    if (
      website &&
      (!actionName ||
        !(
          actionName in actions ||
          Object.values(actions).includes(`${actionName}Url`)
        ))
    ) {
      let actionList = Object.keys(website)
        .filter((key) => key.includes("Url") && key !== "baseUrl")
        .map((key) => key.replace("Url", ""));
      if (website.baseUrl && !actionList.includes("visit"))
        actionList.push("visit");
      return actionList;
    }
    return [];
  });

  const handleKeyDown = (e) => {
    if (e.key === "Enter" && $actionURL) {
      window.open($actionURL, "_blank");
    }
  };

  const selectWebsite = (website) => {
    searchTerm.set(website.websiteName + " ");
    selectionMode.set("action"); // Move on to action selection mode
  };

  searchTerm.subscribe(($searchTerm) => {
    const [websiteName = "", action = "", ...queryParts] =
      $searchTerm.split(" ");
    const query = queryParts.join(" ");

    const website = websites.find(
      (website) =>
        website.websiteName &&
        website.websiteName.toLowerCase() === websiteName.toLowerCase()
    );
    const actionKey = actions[action] || `${action}Url`;
    const actionRequiresInput = ["search", "-s"].includes(action);

    if (website && actionKey in website) {
      if (actionRequiresInput && !query.trim()) {
        actionURL.set("");
      } else {
        actionURL.set(
          query ? `${website[actionKey]}${query}` : website[actionKey]
        );
        actionText.set(`${websiteName} ${action}`);
      }
    } else {
      actionURL.set("");
    }
  });

  const selectAction = (action) => {
    searchTerm.update((value) => {
      const [website] = value.split(" ");
      return `${website} ${action} `;
    });
  };
</script>

<div class="">
  <div class="bg-neutral-100 rounded-xl shadow-md overflow-hidden">
    <div class="md:flex">
      <div class="md:flex-shrink-0">
        <!-- svelte-ignore a11y-img-redundant-alt -->
        <img
          class="h-48 w-full object-cover md:h-full md:w-48"
          src="https://source.unsplash.com/random/500x500?purple"
          alt="Random Image"
        />
      </div>
      <div class="p-8">
        <div
          class="uppercase tracking-wide text-sm text-indigo-500 font-semibold"
        >
          Command
        </div>
        <input
          bind:this={searchInput}
          type="text"
          bind:value={$searchTerm}
          class="block mt-2 w-full placeholder-gray-400/70 dark:placeholder-gray-500 rounded-lg border border-gray-200 bg-white px-5 py-2.5 text-gray-700 focus:border-indigo-400 focus:outline-none focus:ring focus:ring-indigo-300 focus:ring-opacity-40 dark:border-gray-600 dark:bg-gray-900 dark:text-gray-300 dark:focus:border-indigo-300"
          autofocus
        />

        {#if $filteredWebsites.length > 0}
          {#each $filteredWebsites as website (website.websiteName)}
            <!-- svelte-ignore a11y-click-events-have-key-events -->
            <div
              on:click={() => selectWebsite(website)}
              class="bg-purple-100 p-2 mt-2 rounded-lg hover:bg-purple-200 cursor-pointer"
            >
              {website.websiteName}
            </div>
          {/each}
        {/if}

        {#if $actionsList.length > 0}
          <div class="mt-4">
            <div
              class="uppercase tracking-wide text-sm text-indigo-500 font-semibold"
            >
              Actions
            </div>
            {#each $actionsList as action}
              <div
                on:mousedown={() => selectAction(action)}
                class="bg-indigo-100 p-2 mt-2 rounded-lg hover:bg-indigo-200 cursor-pointer"
              >
                {action}
              </div>
            {/each}
          </div>
        {/if}

        {#if $actionURL}
          <div class="mt-6">
            <a
              href={$actionURL}
              target="_blank"
              class="mt-4 bg-indigo-500 text-white px-4 py-2 rounded-lg hover:bg-indigo-600"
              >OK</a
            >
          </div>
        {/if}
      </div>
    </div>
  </div>
</div>
