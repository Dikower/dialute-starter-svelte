<script lang="ts">
  import {onMount} from 'svelte'
  import {createSmartappDebugger} from '@sberdevices/assistant-client'
  import {logger} from './logging'; // Use custom logger to clean up assistant-client`s spam
  import {setTheme} from './themes';

  let assistant;
  let state = {};
  let token = '';       // Set token for assistant-client from https://developers.sber.ru/studio/settings/emulator
  let initPhrase = '';  // Set the name of your SmartApp for activation

  let character = 'eva';
  $: setTheme(character);  // Set theme depended on current character;
  // Now you can CSS custom properties from https://plasma.sberdevices.ru/current/?path=/docs/colors--default

  onMount(() => {
    function getState() {
      logger.log('State was get');
      const state = {}
      return state;
    }

    const init = () => {
      // Use it for debugging in browser
      return createSmartappDebugger({
        token,
        initPhrase,
        getState,
        settings: {debugging: false}
      })
      // TODO: Use to run it in production mode inside Salute App
      // return createAssistant({getState});
    }
    assistant = init();

    assistant.on('start', () => {
      logger.log('SmartApp started',);
    });

    assistant.on('data', (event) => {  // Set your action or data hooks
      if (!event.type) {  // Use invariants to prevent errors on Sber Portal
        return;
      }
      // FIXME Add event handler for closing the app and use "assistant.close()" inside it;

      if (event.type === 'character') {
        character = event.character.id;
      }
      logger.log('Data', event);
    });
  })

</script>

<main>
  <h1>Hello from Dialute!</h1>
  <p>Visit the <a href="https://developer.sberdevices.ru/">Sberdevices Docs</a>
    to learn how to build CanvasApps for Sber Salute.</p>
  <p>Dialute docs comming soon...</p>
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #14c07c;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>