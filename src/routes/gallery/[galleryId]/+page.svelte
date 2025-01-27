<script lang="ts">
	import { page } from '$app/state';
    import type { ImageDBResult, LoadResult } from '$lib/types';
	import type { PageData } from './$types';
	import { MAX_IMAGE_WIDTH } from './constants';
	import Gallery from './Gallery.svelte';
    import UploadForm from './UploadForm.svelte';
    import { SvelteToast } from '@zerodevx/svelte-toast';

    interface Props extends LoadResult {
		data: PageData;
	}
    const columnOneFilter = (item: any, index: number) => index === 0 || !(index % 2);
    const columnTwoFilter = (item: any, index: number) => (index % 2);
    let { data }: Props = $props();
    let options = {};
    let lastRetrieval = $state(data.lastImageLoadedEpoch);
    let columnOne = $state(data.galleryImages.filter(columnOneFilter));
    let columnTwo = $state(data.galleryImages.filter(columnTwoFilter));
    const retrieveNewImages = async () => {
        const response = await fetch(`/rest/${page.params.galleryId}/${lastRetrieval}`); 
        const data: LoadResult = await response.json();
        if (data.lastImageLoadedEpoch > lastRetrieval) {
            lastRetrieval = data.lastImageLoadedEpoch;
            let columnOneNewEntries: ImageDBResult[] = []; 
            let columnTwoNewEntries: ImageDBResult[] = [];

            if (data.galleryImages.length > 1) {
                columnOneNewEntries = data.galleryImages.filter(columnOneFilter);
                columnTwoNewEntries = data.galleryImages.filter(columnTwoFilter);
            } 
            else if (columnTwo.length > columnOne.length) {
                columnOneNewEntries = data.galleryImages;
            }
            else {
                columnTwoNewEntries = data.galleryImages
            }

            columnOne.unshift(...columnOneNewEntries);
            columnTwo.unshift(...columnTwoNewEntries);
        }
    };

    $effect(() => {
        let timeoutId:number = -1;
        const setTimeoutFn = () => {
            retrieveNewImages()
                .then(() => {
                    timeoutId = setTimeout(setTimeoutFn, 7000);
                });
        };
        timeoutId = setTimeout(setTimeoutFn, 7000);

        return () => {
            clearTimeout(timeoutId);
        };
    })
</script>
<SvelteToast {options} />
<div class="main" style:--max-image-width={`${MAX_IMAGE_WIDTH}px`}>
    <div class="container">
        <UploadForm galleryName={data.galleryName} retrieveNewImages={retrieveNewImages} isValidToLoad={data?.isValidToLoad}/>
    
        {#if data?.galleryImages }
            <Gallery columnOne={columnOne} columnTwo={columnTwo} />
        {/if}
    </div>
</div>

<style>
    :global(body) {
        background-color: var(--clr-surface-a20);
        overflow:hidden;
    }

    .main {
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .container {
        display: flex;
        flex-direction: column;
        min-width: min(90vw, 922px);
    }

    :root {
        /** Base colors */
        --clr-dark-a0: #000000;
        --clr-light-a0: #ffffff;

        /** Dark theme primary colors */
        --clr-primary-a0: #f2c200;
        --clr-primary-a10: #f6c939;
        --clr-primary-a20: #f9cf57;
        --clr-primary-a30: #fcd671;
        --clr-primary-a40: #fedc89;
        --clr-primary-a50: #ffe3a1;

        /** Dark theme surface colors */
        --clr-surface-a0: #140d32;
        --clr-surface-a10: #2b2446;
        --clr-surface-a20: #433b5b;
        --clr-surface-a30: #5b5471;
        --clr-surface-a40: #746e87;
        --clr-surface-a50: #8f899e;

        /** Dark theme tonal surface colors */
        --clr-surface-tonal-a0: #2b1c34;
        --clr-surface-tonal-a10: #403148;
        --clr-surface-tonal-a20: #55485c;
        --clr-surface-tonal-a30: #6b5f72;
        --clr-surface-tonal-a40: #827888;
        --clr-surface-tonal-a50: #9a919f;
    }
</style>