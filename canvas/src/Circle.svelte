<script>
	import { getContext, onMount, onDestroy, afterUpdate } from 'svelte';
	import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';

	export let x;
	export let y;
	export let r;
	export let fill;
	export let contextName = 'canvas';

	const { register, deregister, invalidate } = getContext(contextName);

	// tweened x, y, and radius -- for transitions (each of these are "stores", i.e., reactive values)
	const tweenedParams = {
		duration: 1000,
    easing: cubicOut
	};
	const tX = tweened(undefined, tweenedParams);
	const tY = tweened(undefined, tweenedParams);
	const tR = tweened(undefined, tweenedParams);

	// create function specifying how we draw a given circle
	function draw(ctx) {
		ctx.translate($tX, $tY);
		ctx.globalAlpha = 0.8;
		ctx.fillStyle = fill;
		ctx.beginPath();
		ctx.arc(0, 0, $tR, 0, 2 * Math.PI, false);
		ctx.fill();
	}

	onMount(() => {
		register(draw);
		invalidate();
	});

	onDestroy(() => {
		deregister(draw);
	});

	afterUpdate(invalidate);

	$: tX.set(x);
	$: tY.set(y);
	$: tR.set(r);
</script>
