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
		const doc = new jsPDF('portrait', 'mm', (format == 'square' ? [210,210] : format), true),
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
				yOffset: (posterHeight / 6 * i) + posterHeight / 10 ,
				scale: format === 'square' ? 0.3 : 0.35
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

		options = options || { scale: 2 };
			
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
			<input id="date" type="date" placeholder="Datum" required bind:value={date}>
			<label for="start">Startzeit</label>
			<input id="start" type="time" required bind:value={startTime}>
			<label for="end">Endzeit</label>
			<input id="end" type="time" required bind:value={endTime}>
			<label for="location">Location</label>
			<input id="location" type="text" maxlength="30" placeholder="Wo findet der Event statt?" bind:value={place}>
			<label for="text">Text</label>
			<textarea id="text" maxlength="85" rows="3" placeholder="Beschreibung max. 85 Zeichen" bind:value={text}></textarea>
			<label for="color">Farbe</label>
			<select name="color" id="color" bind:value={color}>
				<option selected value="#99FF00">Grün</option>
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
	<svg bind:this={svg[0]} width="255" height="255" viewBox="0 0 255 255" xmlns="http://www.w3.org/2000/svg"><g transform="translate(9 9)" fill="none" fill-rule="evenodd"><path d="M119.13 186.39c19.95 0 36.48-16.34 36.48-36.29 0-14.06-7.41-25.27-18.81-31.73 11.02-6.27 18.81-17.29 18.81-31.73 0-19.95-16.53-36.29-36.48-36.29H87.21c-2.85 0-4.56 2.28-4.56 4.56v126.92c0 2.85 2.28 4.56 4.56 4.56h31.92zm0-72.58H91.77V59.47h27.36c15.2 0 27.36 11.97 27.36 27.17 0 15.2-12.35 27.17-27.36 27.17zm0 63.46H91.77v-54.34h27.36c15.01 0 27.36 11.97 27.36 27.17 0 15.2-12.16 27.17-27.36 27.17z" fill="#3F4EA9" fill-rule="nonzero"/><circle stroke="#3F4EA9" stroke-width="12.35" cx="118.5" cy="118.5" r="118.5"/></g></svg>
	<svg bind:this={svg[1]} width="255" height="255" viewBox="0 0 255 255" xmlns="http://www.w3.org/2000/svg"><g fill="none" fill-rule="evenodd"><path d="M117.3 191.78c14.63 0 27.93-12.73 27.93-26.98 0-2.28-.76-3.61-3.8-4.56l-20.52-7.22-3.23 2.28-.76 2.28c-3.42 9.88-10.07 17.48-18.05 19-.76-1.52-.95-3.23-.95-5.32 0-10.45 8.17-27.74 16.34-43.51l5.51 17.67 3.04 2.28c7.79-4.18 32.68-6.46 31.16-11.4l-6.08-20.52-3.04-2.28c-5.7 3.04-19.76 4.75-26.98 7.41 5.32-10.26 9.88-19.19 11.4-24.32 8.74 6.27 17.48 9.31 27.36 9.31 13.68 0 23.75-15.2 23.75-30.02 0-2.28-.76-3.61-3.8-4.56l-20.33-7.22-3.42 2.28v1.52c0 7.6-3.99 21.47-12.92 22.42-2.28-2.66-1.52-17.1-5.89-18.62l-22.23-7.6-3.42 2.28-.19 1.52c-1.52 19.38-33.63 53.77-33.63 83.6 0 24.32 19.19 40.28 42.75 40.28z" fill="#3F4EA9" fill-rule="nonzero"/><path stroke="#3F4EA9" stroke-width="12.35" d="M9 9h237.5v237.5H9z"/></g></svg>
	<svg bind:this={svg[2]} width="255" height="255" viewBox="0 0 255 255" xmlns="http://www.w3.org/2000/svg"><g transform="translate(9 9)" fill="none" fill-rule="evenodd"><path fill="#3F4EA9" fill-rule="nonzero" d="M134.793 189V79.608H178V49H59v30.608h43.207V189z"/><circle stroke="#3F4EA9" stroke-width="12.35" cx="118.5" cy="118.5" r="118.5"/></g></svg>
	<svg bind:this={svg[3]} width="255" height="255" viewBox="0 0 255 255" xmlns="http://www.w3.org/2000/svg"><g fill="none" fill-rule="evenodd"><path d="M154.775 165.512v13.3h28.975V75.975h-28.975v13.3c-9.738-15.2-25.175-16.625-33.013-16.625-30.4 0-51.062 24.225-51.062 54.625 0 30.162 20.9 54.862 52.487 54.862 7.363 0 21.138-1.662 31.588-16.625zm-25.65-9.024c-16.625 0-28.5-12.826-28.5-29.213 0-16.388 11.875-28.975 28.5-28.975s28.5 12.588 28.5 28.975-11.875 29.213-28.5 29.213z" fill="#3F4EA9" fill-rule="nonzero"/><path stroke="#3F4EA9" stroke-width="12.35" d="M8 9h237.5v237.5H8z"/></g></svg>
</div>