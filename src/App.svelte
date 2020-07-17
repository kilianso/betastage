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
		format;

	$: header = ['Beta Stage Festival', dayjs(date).locale('de').format('DD.MMMM YYYY')];
	$: timePlace = [(startTime || 16) + ' — ' + (endTime ||  23) + ' Uhr', place || 'Dampfzentrale, Bern'];

	onMount( () => {
		generatePDF(1);
	});
	
	function generatePDF(action, button){
		// orientation, unit, format, compression
		const doc = new jsPDF('portrait', 'px', (format == 'square' ? [1000,1000] : format), true),
			posterWidth = doc.internal.pageSize.width || doc.internal.pageSize.getWidth(),
			posterHeight = doc.internal.pageSize.height || doc.internal.pageSize.getHeight(),
			textWidth = 180, /* text width cannot be more than 50% of total width. */
			baseSize = posterWidth / 15,
			lines = doc.splitTextToSize(text || 'Eine kurze Beschreibung zu diesem Event wird hier platziert.', textWidth);

		doc.setFont('Inter-Bold', 'bold');

		doc.setFillColor(color);
		doc.rect(0, 0, posterWidth, posterHeight, "F");
		doc.setTextColor(textColor);
		doc.setFontSize(baseSize);
		doc.text(baseSize / 3, baseSize , header);
		doc.setFontSize(baseSize / 2);
		doc.text(baseSize / 3, baseSize * 3 , timePlace);
		doc.text(baseSize / 3, baseSize * 3.85 , lines);
		
		if (action === 1) {
			let uri = doc.output('datauristring');
			document.getElementById('preview').innerHTML = '';
			renderPDF(uri, document.getElementById('preview'));
		} else if (action === 2) {
			doc.save('betastage.pdf');
		}
		else if (action === 3)  {
			var canvas = document.getElementsByTagName('canvas')[0];
			var image = canvas.toDataURL('image/jpeg');
			button.href = image;
    	}
	}
	
	function renderPDF(url, canvasContainer, options) {

		options = options || { scale: 2 };
			
		function renderPage(page) {
			var viewport = page.getViewport(options.scale);
			var wrapper = document.createElement("div");
			wrapper.className = "canvas-wrapper";
			var canvas = document.createElement('canvas');
			var ctx = canvas.getContext('2d');
			var renderContext = {
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