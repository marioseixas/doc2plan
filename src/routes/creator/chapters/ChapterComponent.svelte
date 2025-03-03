<script lang="ts">
    import { openaiStore } from '../../../stores/openai';
    import { get } from 'svelte/store';
    import Topics from './Topics.svelte';
    import { extractKeyTopics, parseKeyTopics as aiParseKeyTopics } from '$lib/openai';
    import Fa from 'svelte-fa';
    import { faSpinner } from '@fortawesome/free-solid-svg-icons';
    import { createEventDispatcher } from 'svelte';
    import { sendErrorToast } from '$lib/alertToast';

    export let chapter: Chapter;

    const withAI = get(openaiStore).assistantId !== '';

    let loading = false;
    async function findKeyTopics() {
        loading = true;
        try {
            chapter.keyTopics = await extractKeyTopics(chapter.name);
        } catch (error: unknown) {
            sendErrorToast(`extract key topics: ${error}`);
        } finally {
            loading = false;
        }
    }

    function parseKeyTopics() {
        if (!chapter.keyTopics) {
            sendErrorToast('No key topics to parse');
            return;
        }

        const topics = aiParseKeyTopics(chapter.keyTopics);
        updateTopics(topics);
    }

    const dispatch = createEventDispatcher();
    function updateTopics(topics: Topic[]) {
        dispatch('updateTopics', {id: chapter.id, topics});
    }
</script>

<section>
    {#if withAI}
        <textarea bind:value={chapter.keyTopics} class="textarea" placeholder="Key topics"></textarea>
        <button class="btn btn-sm variant-filled-secondary" on:click={findKeyTopics} disabled={loading}>
            {#if loading}
            <Fa icon={faSpinner} class="animate-spin"/>
            {:else}
            Extract key topics
            {/if}
        </button>

        {#if chapter.keyTopics && chapter.keyTopics.length > 0}
            <!-- button parse to topics -->
            <button class="btn btn-sm variant-filled" on:click={parseKeyTopics} disabled={loading}>
                Create topics from key topics
            </button>
        {/if}
    {/if}

    <div class="flex mt-4">
        {#if chapter}
            <Topics topics={chapter.topics} chapter_id={chapter.id} withAI={withAI}/>
        {/if}
    </div>
</section>
