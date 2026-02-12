<script lang="ts">
	let qrCodeUrl = $state<string | null>(null);
	let qrCodeError = $state<string | null>(null);
	let qrCodeSuccess = $state(false);
</script>

<div class="mx-auto w-screen max-w-6xl items-center p-5 pb-16">
	<h1 class="mt-16 text-center text-3xl font-extrabold">
		discord/twitch/kick/snapchat age verifier
	</h1>
	<p class="text-center">
		age verifies your account automatically as an adult on any website using k-id
	</p>
	<p class="mt-4 text-center text-white/50">
		made by <a class="underline" href="https://kibty.town" target="_blank">xyzeva</a> and
		<a class="underline" href="https://github.com/Dziurwa14" target="_blank">Dziurwa</a>, greetz to
		<a class="underline" href="https://amplitudes.me/" target="_blank">amplitudes</a> (for previous work)
	</p>

	<p class="mt-8 text-red-500">this script doesn't work anymore and has been temporarily disabled while we're looking into a fix.</p>

	<h2 class="mt-8 text-2xl font-bold">how does this work</h2>
	<p>
		k-id, the age verification provider discord uses doesn't store or send your face to the server.
		instead, it sends a bunch of metadata about your face and general process details. while this is
		good for your privacy <span class="text-white/50"
			>(well, considering some other providers send actual videos of your face to their servers)</span
		>, its also bad for them, because we can just send legitimate looking metadata to their servers
		and they have no way to tell its not legitimate.
		<br />
		while this was easy in the past, k-id's partner for face verification (faceassure) has made this significantly
		harder to achieve after
		<a href="https://github.com/amplitudesxd/discord-k-id-verifier" class="underline"
			>amplitudes k-id verifier</a
		>
		was released,
		<span class="text-white/50">(which doesn't work anymore because of it.)</span>
		<br />
		<br />
		with discord's decision of making the age verification requirement global, we decided to look into
		it again to see if we can bypass the new checks.
	</p>
	<h3 class="mt-8 text-xl font-bold">step 1: encrypted_payload and auth_tag</h3>

	<p>
		the first thing we noticed that the old implementation doesn't send when comparing a legitimate
		request payload with a generated one, is its missing <code class="bg-white/20 text-blue-400"
			>encrypted_payload</code
		>, <code class="bg-white/20 text-blue-400">auth_tag</code>,
		<code class="bg-white/20 text-blue-400">timestamp</code>
		and
		<code class="bg-white/20 text-blue-400">iv</code> in the body.
		<br />
		<br />
		looking at the code, this appears to be a simple AES-GCM cipher with the key being
		<code class="bg-white/20"
			><span class="text-orange-300">nonce</span> + <span class="text-orange-300">timestamp</span> +
			<span class="text-orange-300">transaction_id</span></code
		>, derived using HKDF (sha256). we can easily replicate this and also create the missing
		parameters in our generated output.
	</p>
	<h3 class="mt-8 text-xl font-bold">step 2: prediction data</h3>
	<p>
		heres where it kind of gets tricky, even after perfectly replicating the encryption, our
		verification attempt still doesn't succeed, so they must also be doing checks on the actual
		payload.
		<br />
		<br />
		after some trial and error, we narrowed the checked part to the prediction arrays, which are
		<code class="bg-white/20 text-blue-400">outputs</code>,
		<code class="bg-white/20 text-blue-400">primaryOutputs</code>
		and <code class="bg-white/20 text-blue-400">raws</code>.
		<br />
		<br />
		turns out, both <code class="bg-white/20 text-blue-400">outputs</code> and
		<code class="bg-white/20 text-blue-400">primaryOutputs</code>
		are generated from <code class="bg-white/20 text-blue-400">raws</code>. basically, the raw
		numbers are mapped to age outputs, and then the outliers get removed with z-score (once for
		<code class="bg-white/20 text-blue-400">primaryOutputs</code>
		and twice for <code class="bg-white/20 text-blue-400">outputs</code>).
		<br />
		<br />
		there is also some other differences:
	</p>
	<ul
		class="list-inside list-disc
"
	>
		<li>
			<code class="bg-white/20 text-blue-400">xScaledShiftAmt</code> and
			<code class="bg-white/20 text-blue-400">yScaledShiftAmt</code> in predictions are not random but
			rather can be one of two values
		</li>
		<li>
			it is checked that the media name (camera) matches one of your media devices in the array of
			devices
		</li>
		<li>it is checked if the states completion times match the state timeline</li>
	</ul>

	<p>
		<br />
		with all of that done,
		<span class="font-extrabold">we can officially verify our age as an adult.</span> all of this
		code is open source and available
		<a
			href="https://github.com/xyzeva/k-id-age-verifier"
			target="_blank"
			class="font-extrabold underline"
			rel="noreferer noopener">on github</a
		>, so you can actually see how we do this exactly.
	</p>
</div>
