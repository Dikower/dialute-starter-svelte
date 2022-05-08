<script lang="ts">
  import {onMount} from 'svelte';
  import {createSmartappDebugger} from '@sberdevices/assistant-client';
  import {setTheme} from './themes';
  import {logger} from "./utils";
  // clear spam from sber client

  let assistant;
  let state = {count: 0};

  // Set token for assistant-client from https://developers.sber.ru/studio/settings/emulator
  let token = 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIzNjk4NTAzMzgzMjViZmViZjg2ZGVjYjAzZDkzZjkwZDQ2ZmMxNmNmZGZiNTUyMDAzNGQxOTFkMWEwYTQ4NDQyNTM5YmU5MjcwMDQyNjI5OCIsImF1ZCI6IlZQUyIsImV4cCI6MTY1MjA5MTUyMywiaWF0IjoxNjUyMDA1MTEzLCJpc3MiOiJLRVlNQVNURVIiLCJ0eXBlIjoiQmVhcmVyIiwianRpIjoiNmM3ZmQ3NWQtMmQ5YS00MTBiLTlkNDQtZjc4YTVmMGE5ODFhIiwic2lkIjoiMTIyZmNkOTMtOGIzNS00ZWY3LWIwNWEtY2MzZWIxZDJiZTBiIn0.Y9AMQ0-9QFYeDpBJxZT4tk_lBVEfkSfHF6RXMLX6GNOkGQ1H7P1Wf7HmyjEeq5cB7AxmjzUR_FXXVvgu7Xgipa6wYE7O5W2bpxDIAYdgbMmjc8XySn5KpLLcsffBEfFCAqIvjfFE96kJp2LKW48maRutqUrONyQVSF6ULwD-_-pX5s14dCRanfME7DeREU2etH_6LE35H957dYAE8d0sJtWFG3P-f5KLcIKFojV8rorgbf6bOBGU_KfK0OxWgNEImF_2AuouEBf_wVhkaia9rD8Qw0l1OSuV74OwUn9G1i42NOABl_DZ_hQS5DTVhzlVPdojJcLvV08Z6eAFOQpSHVq0bEp1U97_FwczVJyi8q-_YdxlcF1R7ktbNs3XfFoW3vvQ0RkNsi-knx2rZc-fKR-FcJQhPb-AUZRIuctK6AADTf7dO4LocmN9vYAnCq9Mgtj54w9m2uBEWZBwVRcbLZhxmMR74MJDDuFkreVsg3Y3eSN20ePMMQrs-URTBIlRY2GFeNm0s_Hn_ZiQFigY3tyYv-AQZqHxgnCzz6netGoacbEBmj4Mmqa-c8AP18DbKsrsnfMGXJZzZ_zgvljr6DVbN3y3RKGPyhCaMfnnI7ycguHFOIW-ptLdsyr5P0DEAJVhKlVcsHG7qChJMMJlkDkjXEWgqbpoWlcZuPt2zZA';

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
    assistant.sendData({action: {action_id: 'click'}})
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
  <p>Dialute docs comming soon...</p>
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
