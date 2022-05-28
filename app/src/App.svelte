<script lang="ts">
  import {onMount} from 'svelte';
  import {createSmartappDebugger, createAssistant} from '@sberdevices/assistant-client';
  import {setTheme} from './themes';
  import {logger} from "./utils";
  // clear spam from sber client

  let assistant;
  let state = {count: 0};

  // Set token for assistant-client from https://developers.sber.ru/studio/settings/emulator
  let token = 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIzNjk4NTAzMzgzMjViZmViZjg2ZGVjYjAzZDkzZjkwZDQ2ZmMxNmNmZGZiNTUyMDAzNGQxOTFkMWEwYTQ4NDQyNTM5YmU5MjcwMDQyNjI5OCIsImF1ZCI6IlZQUyIsImV4cCI6MTY1Mzg1MDc0OSwiaWF0IjoxNjUzNzY0MzM5LCJpc3MiOiJLRVlNQVNURVIiLCJ0eXBlIjoiQmVhcmVyIiwianRpIjoiNTk0NWQ1ZWEtZDU5OS00NmI3LWIzZWItNWQxMmQzNmE3MjQ4Iiwic2lkIjoiZjg2M2RlNDItZWQxNi00NmQ5LTk5MmItYWQ4NjU0MTI0MmJlIn0.OxOsum-_8Z39VjXYttQFddko4j6nc_vEF3YGJeR4CJSRVFpeUFRWFsTH3TInySolmT4nClL03_sUEid0faSJHMiazNFPkOTBVN6M9bJ00u1CyHQOjatI_aoMwOIAPylDhyAU2KN5qAI8nVLVchdZZjJj-fDu4scWwzkKTNaGCRxa8FnParXneghJs0QhV7h74jm99EjY0_bMJfHRhPSBBaSq3OyDq-8r0ZBwOd17Xx00_XTirC2pkQ6LAuKa6WH3NBkCvLbGY2hVOKycMBe_lnGZgIaCEqIfMgjC6A3E8zZ7p8Pb0M6jrqoZgDhTJwrXYiubeWGXpKbSy-YFVPyDxmawmWU2I12820drk3-3MOkkCtYMJ4hgehVpWV_i4pl0Fz_8iu8gNzaL5eJcRzQNkVklO-edUsP-oBH9Uc3BNsLPd63_fEsi4jlnSF-pLymCttdigDI6Z8yMysZLnUYNorKoBXHRxk_DPsXmwCwp8to38BPemZtdiX9X9Ttu3jf8G85mY2u1f3SRh-z6NYsTWeG44ldCresz8z94UwfU1AXSEKP7wz-E4N_QRxVLy5eTgJJCE_VUcMcpY9XCk5-vh92PWFK8iIol96A34ARgMTTeQrx_eeHQpMKpGZXPXif1PypHatk5nzAjpg3JbaJ419yzbDDpywggCKwQuBf1SH0';

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

    const init = (): any => {
      // Use it for debugging in browser
      if (process.env.NODE_ENV === 'production') {
        return createAssistant({getState});
      }
      return createSmartappDebugger({
        token,
        initPhrase,
        getState,
      });
      // TODO: Use to run it in production mode inside Salute App
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
