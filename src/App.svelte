<script>
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
	$: timePlace = [startTime + ' — ' + endTime + ' Uhr', place];
	
	function printPdf(action, button){
		// orientation, unit, format, compression
		const doc = new jsPDF('portrait', 'px', (format == 'square' ? [1000,1000] : format), true),
			posterWidth = doc.internal.pageSize.width || doc.internal.pageSize.getWidth(),
			posterHeight = doc.internal.pageSize.height || doc.internal.pageSize.getHeight(),
			textWidth = posterWidth * 0.5, /* text width cannot be more than 50% of total width. */
			baseSize = posterWidth / 15,
			lines = doc.splitTextToSize(text, textWidth);

		doc.setFillColor(color);
		doc.rect(0, 0, posterWidth, posterHeight, "F");
		doc.setTextColor(textColor);
		doc.setFontSize(baseSize);
		doc.text(baseSize, baseSize * 1.5 , header);
		doc.setFontSize(baseSize / 2);
		doc.text(baseSize, baseSize * 3 , timePlace);
		doc.text(baseSize, baseSize * 4 , lines);
		
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

	function generatePdfPreview() {
		renderPDF(dataURL, document.getElementById('preview'));
	}
	// * this is not important for jsPDF, it's here just to render the result *
	// It's a Mozilla lib called PDFjs that handles pdf rendering
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
	:global(*) {
		box-sizing: border-box;
	}
	:global(html, body) {
		margin: 0;
		padding: 0;
	}
	:global(canvas) {
		width: 100%;
		max-height: calc(100vh - 10rem);
		object-fit: contain;
	}
	main {
		display: flex;
		min-height: 100%;
		flex-wrap: wrap;
	}
	section {
		display: flex;
		flex: 1 0 50%;
		min-width: 480px;
		padding: 5rem;
		justify-content: center;
	}
	.controls {
		flex-direction: column;
		background: #f3f3f3;
	}
	#preview {
		position: sticky;
		top: 0;
		align-self: flex-start;
	}
	input, textarea, select {
		display: block;
		width: 100%;
		padding: 1rem;
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
		border: none;
		background: black;
		color: white;
		padding: 1rem;
		cursor: pointer;
		text-decoration: none;
	}
	button:hover, a[type="button"]:hover  {
		background: rgba(0,0,0,0.85);
	}
</style>
<main>
	<section class="controls">
		<h1>BETA Stage Generator</h1>
		<form class="controls__form">
			<label for="date">Datum</label>
			<input id="date" type="date" placeholder="Datum" bind:value={date}>
			<label for="start">Startzeit</label>
			<input id="start" type="time" bind:value={startTime}>
			<label for="end">Endzeit</label>
			<input id="end" type="time" bind:value={endTime}>
			<label for="location">Location</label>
			<input id="location" type="text" maxlength="30" placeholder="Wo findet der Event statt?" bind:value={place}>
			<label for="text">Text</label>
			<textarea id="text" maxlength="100" rows="3" placeholder="Beschreibung max. 100 Zeichen" bind:value={text}></textarea>
			<label for="color">Farbe</label>
			<select name="color" id="color" bind:value={color}>
				<option selected value="#99FF00">Grün</option>
				<option value="#E400FF">Pink</option>
				<option value="#FF7700">Orange</option>
				<option value="#FFFFFF">Weiss</option>
			</select>
			<label for="format">Format</label>
			<select name="color" id="format" bind:value={format}>
				<option selected value="square">Quadratisch</option>
				<option value="a4">A4</option>
			</select>
		</form>
		<div class="controls__buttons">
			<button on:click={() => printPdf(1, this)}>Preview</button>
			<button on:click={() => {printPdf(2, this)}}>Download PDF</button>
			<a type="button" href="/" download="betastage.jpg" on:click={(e) => {printPdf(3, e.target)}}>Download JPG</a>
		</div>
	</section>
	<section id="preview">
		<!-- preview goes here -->
	</section>
</main>