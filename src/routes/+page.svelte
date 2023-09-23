<script lang="ts">
	import { onMount } from 'svelte';
	import { writable } from 'svelte/store';
	import type { Writable } from 'svelte/store';
	import Button from '@smui/button';
	import CircularProgress from '@smui/circular-progress';

	let NV: any;
	let NVI: any;
	let nv: any;

	let queue: Writable<{ url: string }[]> = writable([]);

	onMount(async () => {
		//@ts-ignore
		const { Niivue, NVImage } = await import('@niivue/niivue');
		NV = Niivue;
		NVI = NVImage;

		nv = new Niivue({ isResizeCanvas: true });
		nv.attachTo('canvas');
		nv.setSliceType(nv.sliceTypeAxial);
	});

	async function onClick() {
		queue.update(($queue) => [
			{ url: `/niivue-images/visiblehuman.nii.gz?random=${Math.random()}` }, // Avoid using cached images by adding a random parameter to the get 
			...$queue
		]);
	}

	let k = 0;
    let id = 0;

    function delay(ms: number) {
		return new Promise((resolve) => setTimeout(resolve, ms))
	}

	$: {
		if (NV && NVI && nv && $queue.length > 0) {
			queue.update(($queue) => {
				const command = $queue[0];
				(async () => {
					k++;                    
                    const myid = id++
                    console.time(`${myid}`)
                    console.timeLog(`${myid}`,`1: loading image ${myid}`)
                    // As the queue grows, the button animation will lag.
                    // 
                    // In my opinion, the solution is to move the expensive decompression of images to a web worker.
                    // That would work well if nvimages could be (de)serialized to UInt8Array without recoding.
                    //    
                    // Are there simpler solutions or workaround? I implemented Option 2 below with a delay of 150, 
                    // works for me but it depends on the application.
                    // Just uncomment *one* of the "await" lines to test.s
                    //                
                    // Option 1: should be THE solution according to classical UI programming books?
                    // await delay(0)
                    //
                    // Option 2: much better responsiveness of the button
                    // await delay(500)                    
                    //
					const img = await NVI.loadFromUrl(command);

                    console.timeLog(`${myid}`,`2: loaded image ${myid}`)
					nv.addVolume(img);
                    console.timeLog(`${myid}`,`3: added volume ${myid}`)
                    console.timeEnd
					k--;
				})();
				return $queue.slice(1);
			});
		}
	}
</script>

<h1>Niivue test</h1>
<Button on:click={onClick}>Load another image</Button>

<p>Command queue length: {$queue.length} Awaiting for: {k}</p>

<div style="width:300px;height:300px">
	<canvas id="canvas" style="width:100%;height:100%" />
</div>
