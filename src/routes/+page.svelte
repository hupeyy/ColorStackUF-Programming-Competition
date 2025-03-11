<script>
  import { onMount } from 'svelte';
  import { spring } from 'svelte/motion';
  import { Button } from '$lib/components/ui/button';

  let gridSize = 64;
  let tileSize = 64;
  let mounted = false;
  let registering = false;

  let tiles = [];
  let mousePosition = spring({ x: 0, y: 0 });
  let palette = [
    "0BA7C2",
    "8D80AD",
    "157145",
    "DEF6CA",
    "DE9A2D"
  ]
  
  onMount(() => {
    mounted = true;
    window.addEventListener('mousemove', handleMouseMove);

    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
    };
  });

  const handleMouseMove = (event) => {
    mousePosition.set({ x: event.clientX, y: event.clientY });
  }
</script>

<style lang="postcss">
  .grid {
    @apply fixed w-screen h-screen top-[-25vh] left-[-25vw] gap-0;
  }

  .tile {
    box-sizing: border-box;
    background-color: rgb(245, 124, 32);
    animation: wave-effect 1s ease-out forwards;
    animation-play-state: running;
    transform-origin: center;
    border: black 2px solid;
  }
</style>


<svelte:head>
  <style>
    body, html {
      overflow: hidden;
    }
  </style>
</svelte:head>

{#if mounted}
<div class="w-screen h-screen">
  <div 
    class="grid"
    style="
      grid-template-columns: repeat({gridSize}, 1fr); 
      grid-template-rows: repeat({gridSize}, 1fr);
      transform: translate(
        {-($mousePosition.x - window.innerWidth / 2) / 20}px, 
        {-($mousePosition.y - window.innerHeight / 2) / 20}px
      );
    "
  >
    {#each Array(Math.floor(gridSize ** 2)) as _, i}
      <div 
        bind:this="{tiles[i]}"
        class="tile"
        style="height:{tileSize}px; width:{tileSize}px; background-color:#{palette[Math.floor(Math.random() * palette.length)]}"
      >
      </div>
    {/each}
  </div>

  <div class="fixed bg-white p-4 rounded-xl left-1/2 top-1/2 transform -translate-x-1/2 -translate-y-1/2 flex flex-col">
    <div class="bg-white p-2 shadow-lg text-center text-6xl ">
        ColorCoding
      <div class="text-xl font-light text-center">
        Presented by ColorStack UF
      </div>
    </div>
    <div>
      <div class="flex flex-row gap-4 text-center mt-4 rounded-lg shadow-lg text-black text-2xl">
        <Button 
          class="bg-green-500 grow text-lg"
          on:click={() => {
            window.location.href = '/lobby';
          }}
        >
          Sign In 
        </Button>
        <Button 
          class="bg-blue-500 grow text-lg"
          on:click={() => {
            registering = true; 
          }}
        >
          Sign Up 
        </Button>
      </div>
    </div>
  </div>
  {#if registering}
    <div class="fixed left-1/2 top-1/2 transform -translate-x-1/2 -translate-y-1/2 bg-white p-4 rounded-xl shadow-lg">
      <div class="text-center text-2xl font-bold">
        Register
      </div>
      
    </div>
  {/if}
  
</div>
{:else}
  <div>Loading...</div>
{/if}
