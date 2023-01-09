<script>
    export let showModal = false;
</script>

{#if showModal}
    <div class="w-full h-full fixed top-0 left-0 bg-black/50 z-50" on:click|self>
        <div class="p-7 rounded-xl max-w-lg mx-auto my-80 text-center bg-white">
            <slot />
        </div>
    </div>
{/if}

<style>
</style>