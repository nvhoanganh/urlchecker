<script>
  import LoadingCirle from "../components/LoadingCirle.svelte";
  import { createEventDispatcher } from "svelte";
  import Icon from "./Icon.svelte";
  export let show;
  export let header;
  export let loading;
  export let full = false;
  export let mainAction;

  const dispatch = createEventDispatcher();
  const action = () => dispatch("action");

  const dismiss = () => {
    dispatch("dismiss");
    show = false;
  };

  const handleKeydown = event => event.key === "Escape" && dismiss();
</script>

<style>
  .modal {
    transition: opacity 0.2s ease;
  }
  .fullheight {
    height: 90vh;
  }
</style>

<!--Modal-->
<div
  class:opacity-0={!show}
  class:pointer-events-none={!show}
  class="modal opacity-0 pointer-events-none fixed w-full h-full top-0 left-0
  flex items-center justify-center">
  <div
    class="modal-overlay absolute w-full h-full bg-gray-900 opacity-50"
    on:click={dismiss} />

  <div
    class={`modal-container bg-white w-11/12 md:${full ? 'max-w-6xl' : 'max-w-xl'} mx-auto rounded
    shadow-lg z-50 overflow-y-auto`}>
    <div
      class="modal-close absolute top-0 right-0 cursor-pointer flex flex-col
      items-center mt-4 mr-4 text-white text-sm z-50">
      <Icon on:click={dismiss} cssClass="fill-current text-white">
        <path
          d="M14.53 4.53l-1.06-1.06L9 7.94 4.53 3.47 3.47 4.53 7.94 9l-4.47 4.47
          1.06 1.06L9 10.06l4.47 4.47 1.06-1.06L10.06 9z" />
      </Icon>
      <span class="text-sm">(Esc)</span>
    </div>

    <!-- Add margin if you want to see some of the overlay behind the modal-->
    <div
      class="modal-content py-4 text-left px-6"
      class:fullheight={full === true}>
      <!--Title-->
      <div class="flex justify-between items-center pb-3">
        <p class="text-2xl font-bold">{header}</p>
        <div class="modal-close cursor-pointer z-50">
          <Icon on:click={dismiss}>
            <path
              d="M14.53 4.53l-1.06-1.06L9 7.94 4.53 3.47 3.47 4.53 7.94 9l-4.47
              4.47 1.06 1.06L9 10.06l4.47 4.47 1.06-1.06L10.06 9z" />
          </Icon>
        </div>
      </div>

      <!--Body-->
      <div class="py-5">
        <slot />
      </div>

      <!--Footer-->
      <div class="flex justify-end pt-2 mb-3">
        {#if mainAction}
          <button
            on:click={action}
            type="button"
            class="bgred hover:bg-red-800 text-white font-semibold py-2 px-4
            border hover:border-transparent rounded">
            {mainAction}
            {#if loading}
              <LoadingCirle />
            {/if}
          </button>
        {/if}
        <button
          on:click={dismiss}
          type="button"
          class="bgdark hover:bg-grey-800 font-semibold ml-1 text-white
          hover:text-white py-2 px-4 border hover:border-transparent rounded">
          Cancel
        </button>
      </div>

    </div>
  </div>
</div>

<svelte:window on:keydown={handleKeydown} />
