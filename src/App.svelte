<script>
	
	function printPdf(action, button){
		var doc = new jsPDF();
		var para='JSPDF is the HTML5 client-side solution for generating PDFs. This is perfect for event tickets, reports, and certificates. Just include the JSPDF library in your <head>, generate your PDF using the many built-in functions';

		var doc = new jsPDF(); 
		var pageWidth = doc.internal.pageSize.width || doc.internal.pageSize.getWidth();
		var ParaWidth=pageWidth*0.85; /* para width will 85% of the page width. */
		var LeftMargin=(pageWidth-ParaWidth)/2;/* Left margin will be half of the remaining space*/
		var TopMargin=30;/*According to your requirement*/
		var lines = doc.splitTextToSize(para, ParaWidth);
		doc.text(LeftMargin, TopMargin , lines);
		
		if (action === 1) {
			let uri = doc.output('datauristring');
			renderPDF(uri, document.getElementById('canvas'));
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
		renderPDF(dataURL, document.getElementById('canvas'));
	}
	// * this is not important for jsPDF, it's here just to render the result *
	// It's a Mozilla lib called PDFjs that handles pdf rendering
	function renderPDF(url, canvasContainer, options) {

		options = options || { scale: 1 };
			
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
	main {
		display: flex;
		height: 100%;
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
		border-right: 1px solid black;
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
		<form class="controls__form">
			<input type="date" placeholder="Datum">
			<input type="time" placeholder="Zeit">
			<input type="text" maxlength="30" placeholder="Ort">
			<textarea maxlength="100" rows="3" placeholder="Text max. 100 Zeichen"></textarea>
			<label for="color">Farbe</label>
			<select name="color" id="color">
				<option selected value="none">Keine</option>
				<option value="green">Grün</option>
				<option value="pink">Pink</option>
				<option value="orange">Orange</option>
			</select>
			<label for="format">Format</label>
			<select name="color" id="format">
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
	<section class="preview">
		<div id="canvas"></div>
	</section>
</main>