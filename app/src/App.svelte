<script lang="ts">
  import {onMount} from 'svelte';
  import {createSmartappDebugger} from '@sberdevices/assistant-client';
  import {setTheme} from './themes';
  import {logger} from "./utils";
  // clear spam from sber client

  let assistant;
  let state = {count: 0};

  // Set token for assistant-client from https://developers.sber.ru/studio/settings/emulator
  let token = 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIzNjk4NTAzMzgzMjViZmViZjg2ZGVjYjAzZDkzZjkwZDQ2ZmMxNmNmZGZiNTUyMDAzNGQxOTFkMWEwYTQ4NDQyNTM5YmU5MjcwMDQyNjI5OCIsImF1ZCI6IlZQUyIsImV4cCI6MTY1Mjk2MTUwOSwiaWF0IjoxNjUyODc1MDk5LCJpc3MiOiJLRVlNQVNURVIiLCJ0eXBlIjoiQmVhcmVyIiwianRpIjoiYWRlMWRhMTgtMmUxYy00YTJkLTljODgtZDBiZDgxNWI2ZTljIiwic2lkIjoiOTM2ZDFjMmYtMDRjZS00NGVkLWE4YzUtMTliMjZkZTczZDBmIn0.ci8qpt_O4dFyVbe2fbmgfXahtRxNXAApYQUZ9LkvHsfp_MuZHZCvMkEl3W_0fF4NT07GhHKtqea3uONCD_KCE7WA3D-4UjbJ1qtrJ-5607yDZgS3wuLz7x2x8xqv7nMXfgRud9V-9O08Nfi3IXfdhiHcndEJjsuWMO-knvw9fDZ-8OhaHjXBvlsoF3yDgKT4ttMTbqVdCOlRmrxVDUQ5kLpl26PUXBI8zKsRG9pFPY9ZXYbjQgeJBy1aYCboURfzXOtgmEDdhRG9N1Is4Nn4IB8WxlC9Hy4WCWdg4EW-xPDiIMX7x7t6uUOcEJon8baHtOM5ZJhN4h9n-J3_S5vl18_x5CAbwnv-REs8ifAl3A9vXkXveN95v5H9Pf5ZtNr4g2iPEBlQzo5b4A_Wqht8KZwVpv322smB6zLkfuN61hOt-QUz7eYQDPPLY9eAYEi02r7dCtPILe-ys4K92r635uhMQnvIwf0-2JDrFdJOjS6gya6vOzBmLNuCXWKgH14hRHa8hLuQM38FWhCStEgerpf7Q-uLGz_gvf4KoKKTzjydXVIrjY-zzATKxJ1CMp1mC8SdJsQrO55Hll1aXKr4omv80Uz9vngB2krjEc62g0kHLy27RSGSz6TeHmyYHqoa2oDufB8VmMOIK2fGouJ8a2WcsMfzTHMyviFgdK3BmhM';

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
