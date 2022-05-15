<script lang="ts">
  import {onMount} from 'svelte';
  import {createSmartappDebugger} from '@sberdevices/assistant-client';
  import {setTheme} from './themes';
  import {logger} from "./utils";
  // clear spam from sber client

  let assistant;
  let state = {count: 0};

  // Set token for assistant-client from https://developers.sber.ru/studio/settings/emulator
  let token = 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIzNjk4NTAzMzgzMjViZmViZjg2ZGVjYjAzZDkzZjkwZDQ2ZmMxNmNmZGZiNTUyMDAzNGQxOTFkMWEwYTQ4NDQyNTM5YmU5MjcwMDQyNjI5OCIsImF1ZCI6IlZQUyIsImV4cCI6MTY1MjcwMDUyMywiaWF0IjoxNjUyNjE0MTEzLCJpc3MiOiJLRVlNQVNURVIiLCJ0eXBlIjoiQmVhcmVyIiwianRpIjoiZWMyOTRhOGQtNjEyNy00MGQ5LWJkOWEtNWRlN2U1Y2IxNzVhIiwic2lkIjoiYzg3MTE5NDEtZjhhMy00NTQ4LTkyNmYtMDIwYmY4YWYwMDhmIn0.izMvbyuf7CqobW6m44uOIEdEI-91nHwzygpxMMmMIWP44ojevpLcbQQb0YsQ-dR8fVMZ7zkto022ZbUHNrmwq7CSHDNTc17GAqy9KyL9_Ixvb8uuEgIunK9G-GgOQmVIZRFRqz4peSCCY0HKtXKRxeNVuKF5CSkBB0m2Kf1WRxWGbovh-LanyOGqfnk84HQD-Yhg45vg3HYSeU-EpczAb1sChKHXqWBWGt2jG1LNRRY3m3dP1oiEbe0TKlChQrpPobDnajNQUQnTxcIIQt7HtBS7L-DryYZdNX_OXGqZxchQrsKMNDBO-OvxL6l6T0aZP-43hMe8nfLjHClrdqlVei8KzkHbDUVoLiybaoK-3KMgF1svpZf4Z7NcLBfhJwE_xgV9BWTfxOXVv8qaWzjpKSw8_LqFYsjnoZ0PIoH6U78fqhfmDhc60fLMhJCh2hc_OleESjajEixr6FggFLuWJholZ8NkF2ZqfeT-JThgIMGGC02gn_c1Vlq1XL0GORQwVOPfsY7US1y7Pl42XPcezfuVoUj_HHK5SeHx1RtIHp-8_wutmOiQdl7_fv3yI1CP85-0vL3nWk3nX4zuy4xpFdowKcjPqYmiB7FiRIau5GSy1Aph-_4AJWUeV41dCELLcnklNeQyxEyvsMz4m2ajI_-qcFMOyOu1zz03xgeeZNw';

  // Set the name of your SmartApp for activation
  let initPhrase = 'запусти темлейт';

  // Set theme depended on current character;
  let character = 'eva'; // default, before sber client gets state
  $: setTheme(character);
  // Now you can CSS custom properties from https://plasma.sberdevices.ru/current/?path=/docs/colors--default

  onMount(() => {
    function getState() {
      return {}
    }

    const init = () => {
      // Use it for debugging in browser
      return createSmartappDebugger({
        token,
        initPhrase,
        getState,
      });
      // TODO: Use to run it in production mode inside Salute App
      // return createAssistant({getState});
    };
    assistant = init();

    assistant.on('start', () => {
      logger.log('SmartApp started');
    });

    assistant.on('data', event => {
      // Set your action or data hooks
      if (!event.type) {
        // Use invariants to prevent errors on Sber Portal
        return;
      }
      // FIXME Add event handler for closing the app and use "assistant.close()" inside it;

      if (event.type === 'character') {
        character = event.character.id;
      }

      if (event.type === 'smart_app_data') {
        state = event.smart_app_data;
      }
      logger.log('data event:', event);
    });
  });

  function handleClick() {
    assistant.sendData({
      action: {
        action_id: 'click',
        // You can send any data to your hook
        data: {}
      }
    })
  }
</script>

<main>
  <h1>Hello from Dialute!</h1>
  <h2> Count: {state.count} </h2>
  <button on:click={handleClick}>Click Me!</button>
  <p>
    Visit the <a href="https://developer.sberdevices.ru/">Sberdevices Docs</a>
    to learn how to build CanvasApps for Sber Salute.
  </p>
  <p>Read <a href="https://dialute.vercel.app/">Dialute docs</a> for more details about the framework</p>
</main>

<style>
  :global(html) {
    background: black;
  }

  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  button {
    background: #14c07c;
    border: none;
    color: white;
    cursor: pointer;
    border-radius: 20px;
    padding: 20px;
  }


  h1 {
    color: #14c07c;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  h2 {
    color: #14c07c;
    font-weight: 100;
  }

  p {
    color: #f4f4f4;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
