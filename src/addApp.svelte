<script lang="ts">
import { generateConfig } from "./commons";
import type { Config } from "./commons";
import EntryCell from "./EntryCell.svelte";
import IconHandler from "./IconHandler.svelte";

	export const exported: {form: HTMLFormElement} = {form: null}
	$: form = exported.form
	let genbutton: HTMLInputElement
	let dlset = false
	let dlurl: string

	let config: Config = {
		config_name: "",
		config_author: "",
		webclips: []
	}

	$: payloads = config.webclips;

	let icon: string

	
	function parsedDataURL(url: string): {mime: string, data: string} {
		let start = url.indexOf("data:") + 5
		let end = url.indexOf(";base64,")
		let mime = url.substring(start, end)
		let data = url.substr(end + 8, url.length)
		return {
			mime,
			data
		}
	}

	export async function generate(evt: Event) {
		evt.preventDefault()
		let data = config;
		for(let id in data.webclips) {
			let webclip = data.webclips[id]
			let icondata = await fetch(webclip.iconurl).then(r => r.arrayBuffer())
			data.webclips[id].icon = new Uint8Array(icondata)
		}
		//console.log(data)
		 let gen = generateConfig(data as Config)
		 //console.log(gen)
	 	let blob = new Blob([gen], {type: "application/x-apple-aspen-config"})
	 	let url = URL.createObjectURL(blob)
		//genbutton.value = "Generate"
		dlurl = url;
		dlset = true;
	}

	function addPayload() {
		config.webclips = [...config.webclips, {
			name: "",
			icon: null,
			iconurl: "",
			url: ""
		}]
	}

	function removePayload(index: number) { 
		let webclips = [...config.webclips]
		webclips.splice(index, 1)
		config.webclips = [...webclips];
	}

	interface DispatchEvent extends Event {
		detail: {
			[k: string]: any
		}
	}

	function handleCell(event: DispatchEvent) {
		console.log(event)
		if(typeof event.detail.remove !== "boolean") return;
		if(event.detail.remove) {
			console.log(`removing payload ${event.detail.id}`, event.detail, config.webclips[event.detail.id])
			removePayload(event.detail.id)
		}
	}
</script>

<main>
	<form bind:this={exported.form} on:submit={generate} enctype="multipart/form-data">
		<div class="formgroup">
			<h3>Config Settings</h3>
			<input type="text" name="config_name" placeholder="Profile Name" bind:value={config.config_name} required/>
			<input type="text" name="config_author" placeholder="Profile Author" bind:value={config.config_author} required/>
		</div>
			{#each payloads as payload, index}
				<div class="formgroup">
						<EntryCell title="Web App Settings" on:message={handleCell} cellid={index}>
							<input type="text" name="name[{index}]" placeholder="Name" bind:value={config.webclips[index].name} required/>
							<IconHandler index={index} bind:icon={config.webclips[index].iconurl} />
							<input type="url" name="url[{index}]" placeholder="URL" bind:value={config.webclips[index].url} required/>
						</EntryCell>
				</div>
			{/each}
			<div class="button" on:click={addPayload}>Add App</div>
			<br /><br />

		<!-- <input bind:this={genbutton} type="submit" class="submit" value="Generate" /> -->
		
		{#if dlset}
			<p>Download: <a href={dlurl}>install.mobileconfig</a></p>
		{/if}
	</form>
</main>


<style>
	

	
	
	

	.button {
		display: inline;
		background: #007aff;
		padding-left: 1rem;
		padding-right: 1rem;
		padding-top: 0.5rem;
		padding-bottom: 0.5rem;
		border-radius: 0.5rem;
		cursor: pointer;
		color: #fff;
		cursor: pointer;
	}
	
</style>
