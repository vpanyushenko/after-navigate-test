<script context="module">
  export const load = async ({ url, fetch, session }) => {
    console.log("Running load");

    const search = url.searchParams.get("search");
    let api = `https://pokeapi.co/api/v2/pokemon`;

    if (search) {
      api += `?search=${search}`;
    }

    const fake_data = await fetch(api).then((res) => res.json());
    return {
      props: {
        data: fake_data,
      },
    };
  };
</script>

<script>
  import { session } from "$app/stores";
  import { browser } from "$app/env";
  import { afterNavigate, invalidate } from "$app/navigation";

  export let data;

  function addQueryParam(event) {
    const value = event.target.value;
    console.log("Adding search param:", value);

    if (browser) {
      const searchParams = new URLSearchParams(window.location.search);

      if (value === "" || value === null || value === undefined) {
        searchParams.delete("search");
      } else {
        searchParams.set("search", value);
      }

      //I suspect this might be while the afterNavigate is happening.

      //The reason the state is being replaced instead of pushed is because this is common for applying filters
      //and it's really annoying to use the back button to go through the history of filters rather than just going
      //back to the previous page.

      //Should this be triggering load to rerun since the URL changed?
      history.replaceState(null, "", "?" + searchParams.toString());

      //Update the session to trigger load to rerun
      $session.timestamp = Date.now();

      //Can also use invalidate to trigger load to rerun but that used to be glitchy so i tend to use the session timestamp hacky way.
      //invalidate("https://pokeapi.co/api/v2/pokemon");
    }
  }

  afterNavigate(() => {
    //This never fires. Is this being used correctly?
    console.log("URL Changed!");
  });
</script>

<h1>Select an option</h1>
<p>
  When an option is selected, it will add a query param which should trigger <code
    >afterNavigate</code
  >
</p>

<select on:change={addQueryParam}>
  <option>Option 1</option>
  <option>Option 2</option>
</select>
