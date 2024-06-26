<script lang="ts">
  import PlayerDisplay from "../components/PlayerDisplay.svelte";
  import { lobbyStore } from "../services/lobbyService";
  import {
    sendMessage,
    addEventHandler,
    removeEventHandler,
  } from "../services/websocketService";
  import { displayInGame } from "../services/navigationService";
  import { onDestroy } from "svelte";

  let handler = addEventHandler("start_game", {
    onSuccess: (e) => {
      console.log("start: ", e);
      displayInGame();
    },
  });

  function startGame() {
    sendMessage({
      type: "start_game",
      error: false,
    });
  }

  function getPlayerDisplayName(player: string): string {
    let out = player;
    if (player === $lobbyStore.you) out += " (you)";
    if (player === $lobbyStore.host) out += " (Host)";
    return out;
  }

  function getPlayerColor(player: string): string {
    let color = "slate";
    if (player === $lobbyStore.you) color = "blue";
    if (player === $lobbyStore.host) color = "yellow";
    return color;
  }

  onDestroy(() => {
    removeEventHandler(handler);
  });

  function shareRoomCodeLink() {
    const roomCodeLink =
      window.location.protocol +
      "//" +
      window.location.host +
      window.location.pathname +
      "?c=" +
      encodeURIComponent($lobbyStore.roomCode);
    navigator.clipboard.writeText(roomCodeLink);
  }
</script>

<div class="md:fixed md:top-5 md_left-7">
  <div class="mb-4">
    <span class="font-semibold"> </span>
    <button on:click={shareRoomCodeLink}>
      Room Code '{$lobbyStore.roomCode}'
    </button>
  </div>

  <div>
    <PlayerDisplay
      playerNames={$lobbyStore.players.map((player) => {
        return {
          name: getPlayerDisplayName(player),
          color: getPlayerColor(player),
        };
      })}
    />
  </div>
</div>

<div class="flex justify-center items-center">
  <div class="flex flex-col">
    <h2 class="text-center font-bold tracking-wide mb-2 text-slate-700">
      Settings
    </h2>
    <div class="card bg-gray-50 px-8 py-5 border-2 border-slate-300 rounded-lg">
      <input
        disabled={true}
        type="number"
        class="w-full"
        placeholder="Number of Rounds"
      />

      <p class="mt-2">Settings are currently not supported...</p>
    </div>

    <button
      on:click={startGame}
      class="w-full blue mt-4 md:w-32 float-right ml-auto"
      disabled={$lobbyStore.you !== $lobbyStore.host}
    >
      Start Game</button
    >
  </div>
</div>

<style>
  .card {
    max-width: 600px;
    min-width: 300px;
    max-height: 800px;
    min-height: 300px;
    width: 40vw;
    height: 70vh;
  }
</style>
