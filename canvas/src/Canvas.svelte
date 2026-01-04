<script>
	import { onMount, onDestroy, setContext } from 'svelte';

	export let width;
	export let height;
	export let contextName = 'canvas';

	const drawFunctions = [];

	let canvas;
	// context (ctx) here is kind of like the "pen" we draw with on canvas
	let ctx;
	let pendingInvalidation = false;
	let frameId;

	function scaleCanvas(canvas, ctx, width, height) {
		const devicePixelRatio = window.devicePixelRatio || 1;

		canvas.width = width * devicePixelRatio;
		canvas.height = height * devicePixelRatio;
		canvas.style.width = width + 'px';
		canvas.style.height = height + 'px';

		ctx.scale(devicePixelRatio, devicePixelRatio);
	}

	function update() {
		if (!ctx) return;

		ctx.clearRect(0, 0, width, height);

		drawFunctions.forEach((fn) => {
			ctx.save();
			fn(ctx);
			ctx.restore();
		});

		pendingInvalidation = false;
	}

	onMount(() => {
		ctx = canvas.getContext('2d');
	});

	onDestroy(() => {
		if (frameId) {
			cancelAnimationFrame(frameId);
		}
	});

	$: setContext(contextName, {
		register(fn) {
			drawFunctions.push(fn);
		},
		deregister(fn) {
			drawFunctions.splice(drawFunctions.indexOf(fn), 1);
		},
		invalidate() {
			if (pendingInvalidation) return;
			pendingInvalidation = true;
			frameId = requestAnimationFrame(update);
		},
	});
	// runes here are svelte syntax meaning "run this line of code whenever something within the line changes"
	$: if (canvas && ctx) scaleCanvas(canvas, ctx, width, height);
</script>

<canvas bind:this={canvas} />
<!-- slot here indicates our canvas component can take children -->
<slot /> 

<style>
</style>
