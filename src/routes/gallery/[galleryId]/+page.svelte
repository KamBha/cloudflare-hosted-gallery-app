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
<svelte:head>
    <title>{data.galleryName}</title>
</svelte:head>
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
        margin: 0;
        padding: 0;
    }

    .main {
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .container {
        display: flex;
        flex-direction: column;
        min-width: 100%;
    }

    :root {

/** Base colors */
--clr-dark-a0: rgb(117, 126, 115);
--clr-light-a0: #ffffff;

/** Dark theme primary colors */
--clr-primary-a0: #ff9f93;
--clr-primary-a10: rgb(117, 126, 115);
--clr-primary-a20: #ffb5aa;
--clr-primary-a30: #ffbfb6;
--clr-primary-a40: #ffcac2;
--clr-primary-a50: #ffd5ce;

/** Dark theme surface colors */
--clr-surface-a0: #295242;
--clr-surface-a10: #3f6455;
--clr-surface-a20: rgb(117, 126, 115);
--clr-surface-a30: #6c887c;
--clr-surface-a40: #849b91;
--clr-surface-a50: #9baea6;

/** Dark theme tonal surface colors */
--clr-surface-tonal-a0: #415a49;
--clr-surface-tonal-a10: rgb(117, 126, 115);
--clr-surface-tonal-a20: #687c6e;
--clr-surface-tonal-a30: #7c8e81;
--clr-surface-tonal-a40: #91a095;
--clr-surface-tonal-a50: #a6b2a9;

    }
</style>