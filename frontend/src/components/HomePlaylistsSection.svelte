<script>
  // icons
  import IconListMusic from '@lucide/svelte/icons/list-music';
  import IconListPlus from '@lucide/svelte/icons/list-plus';
  import IconCircleQuestionMark from '@lucide/svelte/icons/circle-question-mark';
  import IconArrowRight from '@lucide/svelte/icons/arrow-right';
  import IconChevronLeft from '@lucide/svelte/icons/chevron-left';
  import IconChevronRight from '@lucide/svelte/icons/chevron-right';
  import IconSpeaker from '@lucide/svelte/icons/speaker';
  import IconPlus from '@lucide/svelte/icons/plus';
  import IconX from '@lucide/svelte/icons/x';
  import IconListVideo from '@lucide/svelte/icons/list-video';

  // ui components
  import { Modal } from '@skeletonlabs/skeleton-svelte';
  import { toaster } from '../lib/toaster';

  // utils
  import { formatDate } from '../lib/utils';

  // modal state
  let newName = $state('');
  let newDescription = $state('');

  $effect(() => {
    if (!$showPlaylistModal) {
      newName = '';
      newDescription = '';
    }
  });

  function closeModal() {
    showPlaylistModal.set(false);
  }

  // playlists state/events
  import {
    playlists,
    playlistsLoading,
    showPlaylistModal,
    creatingPlaylist
  } from '../stores/state.js';
  import { layoutEvents } from '../stores/events.js';

  function createPlaylist() {
    layoutEvents.set({
      type: 'create_playlist',
      name: newName,
      description: newDescription
    });
  }
</script>

<section class="mx-auto w-full max-w-5xl">
  <div class="flex flex-row justify-between">
    <h2 class="flex flex-row gap-2 items-center text-xl sm:text-2xl font-bold">
      <IconListMusic />
      Preset Playlists
    </h2>

    <Modal
      open={$showPlaylistModal}
      onOpenChange={(e) => ($showPlaylistModal = e.open)}
      triggerBase="btn preset-filled-tertiary-500 items-center"
      contentBase="card bg-surface-100-900 p-4 space-y-4 shadow-xl max-w-screen-lg"
      backdropClasses="backdrop-blur-sm"
    >
      {#snippet trigger()}<IconPlus /> New{/snippet}
      {#snippet content()}
        <header class="flex flex-row gap-2 items-center relative">
          <IconListPlus />
          <h2 class="text-xl font-semibold">New Playlist</h2>
          <button
            class="absolute top-0 right-0 btn btn-ghost btn-xs"
            onclick={closeModal}
            aria-label="Close">&times;</button
          >
        </header>
        <article class="flex flex-col gap-2">
          <div class="form-control">
            <label class="label" for="playlist-name">
              <span class="label-text">Name</span>
            </label>
            <input
              id="playlist-name"
              class="input input-bordered w-full"
              bind:value={newName}
              maxlength="40"
              required
              placeholder="Playlist name..."
            />
          </div>
          <div class="form-control">
            <label class="label" for="playlist-desc">
              <span class="label-text">Description</span>
            </label>
            <textarea
              id="playlist-desc"
              class="textarea textarea-bordered w-full"
              bind:value={newDescription}
              maxlength="100"
              placeholder="Description (optional)"
            ></textarea>
          </div>
        </article>
        <footer class="flex justify-end gap-4">
          <button
            type="button"
            class="btn preset-tonal"
            onclick={closeModal}
            disabled={$creatingPlaylist}>Cancel</button
          >
          <button
            type="button"
            class="btn preset-filled"
            onclick={createPlaylist}
            disabled={$creatingPlaylist || !newName}
            >{$creatingPlaylist ? 'Creating...' : 'Create'}
          </button>
        </footer>
      {/snippet}
    </Modal>
  </div>
  <div class="pt-2">
    {#if playlists.length == 0}
      <div class="flex flex-row gap-2 items-center justify-center">
        <IconCircleQuestionMark />
        <p class="text-sm opacity-60 text-center">
          No preset playlists currently exist. Maybe try creating one?
        </p>
      </div>
    {/if}

    {#if $playlistsLoading}
      <div class="text-center p-4">Loading...</div>
    {:else}
      <div class="pt-4 pb-1">
        <div class="flex flex-row items-center">
          <div class="relative w-0 h-0 flex flex-row items-center">
            <button
              type="button"
              class="btn absolute left-[-10px] preset-filled-tertiary-500 p-2.5 rounded-full"
              onclick={() => {
                document
                  .querySelector('#playlist-row')
                  .scrollBy({ left: -200, behavior: 'smooth' });
              }}><IconChevronLeft /></button
            >
          </div>

          <div
            class="px-4 flex flex-row gap-5 overflow-x-auto overflow-y-hidden scroll-smooth snap-x snap-mandatory w-full whitespace-nowrap py-3"
            id="playlist-row"
          >
            {#each $playlists as playlist}
              <div
                class="card w-68 min-w-68 shadow-xl bg-base-100 flex flex-col justify-between cursor-pointer hover:shadow-2xl transition-all preset-filled-surface-100-900 py-3 px-6 gap-4 snap-center"
              >
                <div class="card-body flex-1 flex flex-col gap-2">
                  <div class="flex items-center mb-2 gap-2">
                    <span class="bg-primary/10 rounded-full">
                      <IconListVideo class="w-7 h-7 text-primary" />
                    </span>
                    <h2 class="card-title text-md font-semibold truncate flex-1">
                      {playlist.name}
                    </h2>
                  </div>
                  <p class="text-base-content/80 text-sm line-clamp-2">
                    {playlist.description != '~' ? playlist.description : 'No description'}
                  </p>
                  <div
                    class="flex flex-row flex-wrap items-center gap-1 mt-2 text-sm text-base-content/70"
                  >
                    <span class="flex items-center gap-1">
                      <IconSpeaker class="w-4 h-4" />
                      {playlist.size} presets •
                    </span>
                    <span>Updated {formatDate(playlist.last_updated)}</span>
                  </div>
                </div>
                <div class="flex flex-row justify-end">
                  <button
                    type="button"
                    class="btn preset-filled w-fit"
                    onclick={() =>
                      toaster.info({
                        title: 'Coming soon!',
                        description: 'When implemented, you will be able to open this playlist.'
                      })}
                  >
                    <span>Open</span>
                    <IconArrowRight size={18} />
                  </button>
                </div>
              </div>
            {/each}
          </div>
          <div class="relative w-0 h-0">
            <button
              type="button"
              class="btn absolute right-[-10px] p-2.5 rounded-full preset-filled-tertiary-500"
              onclick={() => {
                document.querySelector('#playlist-row').scrollBy({ left: 200, behavior: 'smooth' });
              }}><IconChevronRight /></button
            >
          </div>
        </div>
      </div>
    {/if}
  </div>
</section>
