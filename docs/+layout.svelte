<script>
	import '@evidence-dev/tailwind/fonts.css';
	import '../app.css';
	import { EvidenceDefaultLayout } from '@evidence-dev/core-components';
	export let data;
</script>
<svelte:head>
	<script>
		// CHANGE ME
		const __REPORT_ROOT__ = `/docs`;

		function normalizeUrl(url) {
			if (url && url.startsWith('/') && !url.startsWith(__REPORT_ROOT__)) {
				url = __REPORT_ROOT__ + url;
			}
			return url;
		}

		function processManifest(manifest) {
			const { renderedFiles } = manifest;
			if (!renderedFiles) return manifest;
			for (const schema in renderedFiles) {
				const files = renderedFiles[schema];
				files.forEach((url, index) => {
					if (url.startsWith('static')) {
						url = url.substring(6);
					}
					files[index] = normalizeUrl(url);
				});
			}
			return manifest;
		}

		// Save the original fetch function
		const originalFetch = window.fetch;

		 function fetchProxy(...args) {
			let url = args[0];
			const isManifest = url.endsWith('data/manifest.json') && !url.startsWith('http');
			args[0] = normalizeUrl(url);
			const res = originalFetch.apply(null, args);
			if (isManifest) {
				return res
						.then(response => response.json())
						.then(processManifest)
						.then(data => {
							return new Response(JSON.stringify(data), {
								status: 200,
								headers: {
									'Content-Type': 'application/json'
								}
							});
						});
			}
			return res;
		}

		window.fetch = fetchProxy;

	</script>
</svelte:head>

<EvidenceDefaultLayout {data}>
	<slot slot="content" />
</EvidenceDefaultLayout>
