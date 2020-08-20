<script>
	import {onMount} from 'svelte';
	import dayjs from 'dayjs';
	import 'dayjs/locale/de';

	let date,
		startTime,
		endTime,
		place,
		text,
		color,
		textColor = '#3F4EA9',
		format,
		svg = [,,,];

	$: header = ['Beta Stage Festival', dayjs(date).locale('de').format('DD.MMMM YYYY')];
	$: timePlace = [(startTime || 16) + ' — ' + (endTime ||  23) + ' Uhr', place || 'Dampfzentrale, Bern'];

	onMount( () => {
		generatePDF(1);
	});
	
	function generatePDF(action, button){
		// orientation, unit, format, compression
		const doc = new jsPDF('portrait', 'mm', (format == 'square' ? [210,210] : format), false),
			posterWidth = doc.internal.pageSize.width || doc.internal.pageSize.getWidth(),
			posterHeight = doc.internal.pageSize.height || doc.internal.pageSize.getHeight(),
			textWidth = posterWidth / 2, /* text width cannot be more than 50% of total width. */
			baseSize = posterWidth / 15,
			lines = doc.splitTextToSize(text || 'Eine kurze Beschreibung zu diesem Event wird hier platziert.', textWidth);

		doc.setFont('Inter-Bold', 'bold');

		doc.setFillColor(color);
		doc.rect(0, 0, posterWidth, posterHeight, "F");
		doc.setTextColor(textColor);
		doc.setFontSize(baseSize * 2);
		doc.text(baseSize / 3, baseSize , header);
		doc.setFontSize(baseSize);
		doc.text(baseSize / 3, baseSize * 3 , timePlace);
		doc.text(baseSize / 3, baseSize * 3.85 , lines);

		// render the svg element
		svg.forEach((el, i) => {
			svg2pdf(svg[i], doc, {
				xOffset: i % 2 ? Math.floor(Math.random() * posterWidth / 8) + posterWidth / 20 : posterWidth / 2 - Math.floor(Math.random() * posterWidth / 8) + posterWidth / 20,
				yOffset: (posterHeight / 6 * i) + posterHeight / 9 ,
				scale: format === 'square' ? 0.325 : 0.425
			});
		})

		if (action === 1) {
			let uri = doc.output('datauristring');
			document.getElementById('preview').innerHTML = '';
			renderPDF(uri, document.getElementById('preview'));
		} else if (action === 2) {
			doc.save('betastage.pdf');
		}
		else if (action === 3)  {
			let canvas = document.getElementsByTagName('canvas')[0],
				image = canvas.toDataURL('image/jpeg');
			button.href = image;
    	}
	}
	
	function renderPDF(url, canvasContainer, options) {

		options = options || { scale: 3 };
			
		function renderPage(page) {
			let viewport = page.getViewport(options.scale),
				wrapper = document.createElement("div");
			wrapper.className = "canvas-wrapper";

			let canvas = document.createElement('canvas'),
				ctx = canvas.getContext('2d'),
				renderContext = {
					canvasContext: ctx,
					viewport: viewport
				};
			
			canvas.height = viewport.height;
			canvas.width = viewport.width;
			wrapper.appendChild(canvas)
			canvasContainer.appendChild(wrapper);
			
			page.render(renderContext);
		}
		
		function renderPages(pdfDoc) {
			for(var num = 1; num <= pdfDoc.numPages; num++)
				pdfDoc.getPage(num).then(renderPage);
		}

		PDFJS.disableWorker = true;
		PDFJS.getDocument(url).then(renderPages);
	}
</script>

<style>
	@font-face {
		font-family: 'Inter-Regular';
		src: url('/fonts/Inter-Regular.woff2');
	}
	:root {
		--darkGray: #202020;
		--lightGray: #999999;
	}
	:global(*) {
		box-sizing: border-box;
	}
	:global(html, body) {
		margin: 0;
		padding: 0;
		color: white;
		background: black;
		font-family: 'Inter-Regular', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
	}
	@media screen and (min-width: 800px) {
		:global(body) {
			box-shadow: inset 50vw 0 0 0 var(--darkGray);
		}
	}
	:global(html) {
		height: 100%;
	}
	:global(body) {
		display: flex;
		flex-direction: column;
		justify-content: center;
		min-height: 100%;
	}
	:global(canvas) {
		width: 100%;
		max-height: calc(100vh - 10rem);
		object-fit: contain;
	}
	main {
		display: flex;
		margin: 0 auto;
		flex-wrap: wrap;
		max-width: 1440px;
		flex: 1 0 100%;
	}
	section {
		display: flex;
		flex: 1 1 400px;
		padding: 5vw;
		justify-content: center;
	}
	.controls {
		flex-direction: column;
		background: none;
	}
	#preview {
		position: sticky;
		top: 0;
		align-self: center;
	}
	input, textarea, select {
		-webkit-appearance: none;
		appearance: none;
		resize: none;
		display: block;
		width: 100%;
		padding: 1rem;
		background: none!important;
		color: white;
		border: 1px solid var(--lightGray);
		box-shadow: none;
		outline: none;
	}
	label {
		margin: 1rem 0 0.25rem 0;
		font-size: 0.75rem;
		color: var(--lightGray);
	}
	::placeholder {
		color: var(--lightGray);
	}
	::-webkit-calendar-picker-indicator {
    	filter: invert(0.7);
	}
	input[type=time]:invalid::-webkit-datetime-edit,
	input[type=date]:invalid::-webkit-datetime-edit {
		color: var(--lightGray);
	}
	.controls__buttons {
		display: flex;
		justify-content: space-between;
		margin-top: 1rem;
	}
	.controls__buttons * {
		flex: 0 0 32%;
		text-align: center;
	}
	.hiddenSVG {
		display: none;
	}

	button, button:active, button:focus,
	a[type="button"], a[type="button"]:active, a[type="button"]:focus {
		margin: 0;
		appearance: none;
		-webkit-appearance: none;
		border: 1px solid var(--darkGray);
		background: white;
		color: black;
		padding: 1rem;
		cursor: pointer;
		text-decoration: none;
	}
	button:hover, a[type="button"]:hover  {
		background: rgba(255,255,255,0.9);
	}
</style>
<main>
	<section class="controls">
		<h1>BETA Stage — Grafik Generator</h1>
		<form class="controls__form" autocomplete="off">
			<label for="date">Datum</label>
			<input id="date" type="date" placeholder="Wähle ein Datum" required bind:value={date}>
			<label for="start">Startzeit</label>
			<input id="start" type="time" placeholder="Wann geht's los?" required bind:value={startTime}>
			<label for="end">Endzeit</label>
			<input id="end" type="time" placeholder="Wann ist es zu Ende?" required bind:value={endTime}>
			<label for="location">Location</label>
			<input id="location" type="text" maxlength="30" placeholder="Wo findet der Event statt?" bind:value={place}>
			<label for="text">Text</label>
			<textarea id="text" maxlength="85" rows="3" placeholder="Beschreibung max. 85 Zeichen" bind:value={text}></textarea>
			<label for="color">Farbe</label>
			<select name="color" id="color" bind:value={color}>
				<option selected value="#99FF00">Grün</option>
				<option value="#E2E3C6">Beige</option>
				<option value="#E400FF">Pink</option>
				<option value="#FF7700">Orange</option>
				<option value="#FFFFFF">Weiss</option>
			</select>
			<label for="format">Format</label>
			<select name="color" id="format" bind:value={format}>
				<option selected value="a4">A4</option>
				<option value="square">Quadratisch</option>
			</select>
		</form>
		<div class="controls__buttons">
			<button on:click={() => generatePDF(1, this)}>&#x21bb; Vorschau</button>
			<button on:click={() => {generatePDF(2, this)}}>Download PDF</button>
			<a type="button" href="/" download="betastage.jpg" on:click={(e) => {generatePDF(3, e.target)}}>Download JPG</a>
		</div>
	</section>
	<section id="preview">
		<!-- preview goes here -->
	</section>
</main>
<div class="hiddenSVG">
	<svg bind:this={svg[0]} width="255" height="255" viewBox="0 0 255 255" xmlns="http://www.w3.org/2000/svg"><g transform="translate(8 9)" fill="none" fill-rule="evenodd"><circle stroke="#36499B" stroke-width="9" cx="119.04" cy="119.04" r="118.749"/><path d="M131.17 166.942H93.685V128.27h37.483c14.331 0 22.97 6.281 22.97 19.437 0 12.169-8.05 19.236-22.97 19.236m-2.75-96c13.744 0 20.811 7.278 20.811 17.671 0 11.58-7.067 18.844-20.812 18.844H93.672V70.94h34.746zm28.468 44.368c11.584-5.888 18.65-16.293 18.65-30.038 0-19.04-12.958-36.122-45.154-36.122H67.168v139.783h65.373c32.985 0 48.688-16.1 48.688-40.052 0-13.938-7.853-27.483-24.342-33.571" fill="#36499B" fill-rule="nonzero"/></g></svg>
	<svg bind:this={svg[1]} width="255" height="255" viewBox="0 0 255 255" xmlns="http://www.w3.org/2000/svg"><g fill="none" fill-rule="evenodd"><path stroke="#36499B" stroke-width="9" d="M9.312 9.312h237.591v237.591H9.312z"/><path d="M127.899 91.969c15.76 0 25.218 10.517 27.317 25.845h-54.424c2.52-17.86 13.025-25.845 27.107-25.845m53.793 38.033c0-34.461-20.384-58.627-53.583-58.627-32.151 0-53.587 22.693-53.587 56.738 0 34.668 22.067 56.738 54.008 56.738 26.055 0 44.967-14.081 50.432-36.144h-25.429c-3.992 9.664-12.188 15.55-24.793 15.55-15.343 0-26.27-8.616-28.159-27.948h81.111v-6.307z" fill="#36499B" fill-rule="nonzero"/></g></svg>
	<svg bind:this={svg[2]} width="255" height="255" viewBox="0 0 255 255" xmlns="http://www.w3.org/2000/svg"><g fill="none" fill-rule="evenodd"><path stroke="#36499B" stroke-width="9" d="M128.082 9.236L9.27 246.866h237.629z"/><path d="M159.12 141.088v-19.41h-20.614v-30.33h-25.284v30.33H97.049v19.41h16.173v59.062c0 19.208 10.516 26.69 32.555 26.69 4.044 0 8.695-.405 13.343-.81V206.6h-8.692c-8.493 0-11.93-2.226-11.93-9.1v-56.412h20.622z" fill="#36499B" fill-rule="nonzero"/></g></svg>
	<svg bind:this={svg[3]} width="255" height="255" viewBox="0 0 255 255" xmlns="http://www.w3.org/2000/svg"><g fill="none" fill-rule="evenodd"><path stroke="#36499B" stroke-width="8" d="M8.333 99.88l45.47 146.978h147.134l45.465-146.977L127.368 9.042z"/><path d="M121.167 182.456c-11.992 0-17.99-5.58-17.99-14.471 0-8.478 4.756-13.648 19.232-15.3l14.057-1.655c5.584-.61 9.505-1.86 12.188-4.343v9.72c0 18.607-13.648 26.06-27.499 26.06m57.063 16.128c-2.894-4.135-4.342-12.613-4.342-20.262v-56.867c0-19.228-12.61-32.048-44.458-32.048-33.27 0-46.5 13.862-47.948 36.39h25.635c1.035-13.03 7.24-16.955 22.126-16.955 15.092 0 19.434 4.963 19.434 13.233 0 7.857-5.17 10.958-15.092 12.2l-16.127 1.444c-30.185 2.894-40.94 15.507-40.94 33.29 0 20.672 15.714 32.047 38.456 32.047 15.713 0 27.295-5.584 34.324-15.51.414 5.17 1.238 10.133 2.894 13.03l26.038.008z" fill="#36499B" fill-rule="nonzero"/></g></svg>
</div>