<script>
    import { onMount } from 'svelte';
	import { fly } from 'svelte/transition';

	export let data;

	const GRIDRESIZELIMIT = .8;

	let grillageHeight = 0;
	let grillageWidth = 0;
	let grillage;
	onMount(() => {
		grillage = document.getElementById("grillage");
		if (grillage) {
			grillage.addEventListener('contextmenu', (e) => {
				e.preventDefault();
			});
		}

		// Continually know height of the grid
		const updateGrillageSize = () => {
        	grillageHeight = grillage.clientHeight;
			grillageWidth = grillage.clientWidth;

			let nbPixelsCote = parseFloat(getComputedStyle(document.documentElement).getPropertyValue('--cote'));

			if (grillageHeight > GRIDRESIZELIMIT*window.innerHeight) {
				let newHeight = (GRIDRESIZELIMIT*window.innerHeight) / (grillageHeight/nbPixelsCote);
				document.documentElement.style.setProperty('--cote', newHeight+"px");
			}
			if (grillageWidth > GRIDRESIZELIMIT*window.innerWidth) {
				let newHeight = (GRIDRESIZELIMIT*window.innerWidth) / (grillageWidth/nbPixelsCote);
				document.documentElement.style.setProperty('--cote', newHeight+"px");
			}
      	};
		const observer = new ResizeObserver(updateGrillageSize);
		observer.observe(grillage);

		// Clean up
		return () => observer.disconnect();
	});
	
	function handleMouseEvent(event) {
		// clic gauche -> ajout
		if (event.button === 0) {
			ajouterBlock(event)
		} else if (event.button === 2) {
			// clic droit -> suppression
			retirerBlock(event)
		}
	}

	/// différents cas de figure dès qu'il faut retirer un block.
	/// si aucun block à l'endroit visé, ne rien faire
	/// si il y a un block à l'endroit visé
	/// 	s'il est seul sur la ligne, 'retirer cette ligne'
	///		sinon, enlever le block sans rien faire d'autre
	function retirerBlock(event) {
		let co = getCoordinates(event);
		let classe = getEventClass(event);

		if (classe.includes("top")) {
			retirerEnHaut(co);
		}
		else if (classe.includes("right")) {
			retirerAdroite(co);
		}
		else if (classe.includes("bottom")) {
			retirerEnBas(co);
		}
		else {
			retirerAGauche(co);
		}
	}

	function retirerAdroite(co) {
		let toSearch = {
			row: co.row,
			col: parseInt(co.col) + 1,
		}
		if (blockAcetEndroit(toSearch)) {
			retirerLa(toSearch);
			if (estSeulSurColonne(toSearch.col)) {
				toutDecalAGauche(toSearch.col);
			}
		}
	}

	function retirerAGauche(co) {
		let toSearch = {
			row: co.row,
			col: co.col - 1,
		}
		if (blockAcetEndroit(toSearch)) {
			retirerLa(toSearch);
			if (estSeulSurColonne(toSearch.col)) {
				toutDecalAGauche(co.col);
			}
		}
	}

	function estSeulSurColonne(col) {
		let solo = true;
		data.listOfCo.forEach(coordonnee => {
			if (parseInt(coordonnee.col) == parseInt(col)) {
				solo = false;
			}
		});
		return solo;
	}

	function toutDecalAGauche(col) {
		data.listOfCo.forEach(coordonnee => {
			if (coordonnee.col >= col) {
				coordonnee.col --;
			}
		});
	}

	function retirerEnBas(co) {
		let toSearch = {
			row: parseInt(co.row) + 1,
			col: co.col,
		}
		if (blockAcetEndroit(toSearch)) {
			retirerLa(toSearch);
			if (estSeulSurLigne(toSearch.row)) {
				toutDecalEnHaut(toSearch.row);
			}
		}
	}

	function retirerEnHaut(co) {
		let toSearch = {
			row: co.row - 1,
			col: co.col,
		}
		if (blockAcetEndroit(toSearch)) {
			retirerLa(toSearch);
			if (estSeulSurLigne(toSearch.row)) {
				toutDecalEnHaut(co.row);
			}
		}
	}

	function toutDecalEnHaut(row) {
		data.listOfCo.forEach(coordonnee => {
			if (coordonnee.row >= row) {
				coordonnee.row --;
			}
		});
	}

	function retirerLa(co) {
		data.listOfCo = data.listOfCo.filter((c) => c.row != co.row || c.col != co.col);
	}

	function estSeulSurLigne(row) {
		let solo = true;
		data.listOfCo.forEach(coordonnee => {
			if (parseInt(coordonnee.row) == parseInt(row)) {
				// plus dès qu'un block est trouvé car l'élément car le block a déjà été enlevé. On chercher donc s'il y a le moindre autre block sur la ligne
				solo = false;
			}
		});
		return solo;
	}

	/// différents cas de figure dès qu'il faut ajouter un block
	/// si aucun block à l'endroit visé, l'ajouter simplement
	/// sinon 'décaler' la ligne ou la colonne visée au cran d'après, puis insérer notre block à l'endroit voulu
	function ajouterBlock(event) {
		let co = getCoordinates(event);
		let classe = getEventClass(event);

		if (classe.includes("top")) {
			insererEnHaut(co);
		}
		else if (classe.includes("right")) {
			insererAdroite(co);
		}
		else if (classe.includes("bottom")) {
			insererEnBas(co);
		}
		else {
			insererAgauche(co);
		}
	}

	function insererEnBas(co) {
		let toSearch = {
			row: parseInt(co.row) + 1,
			col: co.col,
		}
		let toInsert = toSearch;
		// block obstruant, tout décaler
		if (blockAcetEndroit(toSearch)) {
			toutDecalEnBas(parseInt(co.row)+1);
		}
		insererLa(toInsert);
	}

	function insererAdroite(co) {
		let toSearch = {
			row: co.row,
			col: parseInt(co.col) + 1,
		}
		let toInsert = toSearch;
		// block obstruant, tout décaler
		if (blockAcetEndroit(toSearch)) {
			console.log(toSearch, "et", co.col+1);
			toutDecalAdroite(parseInt(co.col)+1);
		}
		insererLa(toInsert);
	}

	function insererEnHaut(co) {
		let toSearch = {
			row: co.row - 1,
			col: co.col,
		}
		let toInsert = toSearch;
		// block obstruant, tout décaler
		if (blockAcetEndroit(toSearch) || toSearch.row <= 0) {
			toutDecalEnBas(co.row);
			toInsert = {
				row: co.row,
				col: co.col
			}
		}
		insererLa(toInsert);
	}

	function toutDecalEnBas(row) {
		data.listOfCo.forEach(coordonnee => {
			if (coordonnee.row >= row) {
				coordonnee.row ++;
			}
		});
	}

	function insererAgauche(co) {
		let toSearch = {
			row: co.row,
			col: co.col - 1,
		}
		let toInsert = toSearch;
		// block obstruant, tout décaler
		if (blockAcetEndroit(toSearch) || toSearch.col <= 0) {
			toutDecalAdroite(co.col);
			toInsert = {
				row: co.row,
				col: co.col
			}
		}
		insererLa(toInsert);
	}

	function insererLa(co) {
		data.listOfCo = [...data.listOfCo, co];
	}

	function toutDecalAdroite(col) {
		data.listOfCo.forEach(coordonnee => {
			if (coordonnee.col >= col) {
				coordonnee.col ++;
			}
		});
	}

	function blockAcetEndroit(co) {
		let toRet = false;
		data.listOfCo.forEach(coordonnee => {
			if (coordonnee.row == co.row && coordonnee.col == co.col) {
				toRet = true;
			}
		});
		return toRet;
	}

	function getEventClass(event) {
		return event.target.classList[0];
	}
	
	function getCoordinates(event) {
		let currentBlock = event.target.parentElement;
		return(getGridPosition(currentBlock));
	}

	function getGridPosition(element) {
        // Get the computed style
        const style = getComputedStyle(element);
        
        // Get the grid row and column start
        const gridRowStart = style.gridRowStart;
        const gridColumnStart = style.gridColumnStart;

        return {
            row: gridRowStart,
            col: gridColumnStart
        };
    }

	// garbage
	function handleKeyPress() {}
</script>

<svelte:head>
	<title>Jeu du grillage</title>
	<meta name="Jeu du grillage"/>
</svelte:head>

<main>
	<div id="grillage">
		{#each data.listOfCo as co}
			<div transition:fly={{ y: 100, duration: 100 }} class="block" style="grid-row:{co.row}; grid-column: {co.col};">
				<div on:mousedown={handleMouseEvent} role="button" tabindex="0" on:keypress={handleKeyPress} class="border-top"></div>
				<div on:mousedown={handleMouseEvent} role="button" tabindex="0" on:keypress={handleKeyPress} class="border-right"></div>
				<div on:mousedown={handleMouseEvent} role="button" tabindex="0" on:keypress={handleKeyPress} class="border-bottom"></div>
				<div on:mousedown={handleMouseEvent} role="button" tabindex="0" on:keypress={handleKeyPress} class="border-left"></div>
			</div>
		{/each}
	</div>

</main>

<style>

	:root {
		--cote: 75px;
	}

	main {
		height: 100vh;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	#grillage {
		border: 1px dotted black;
		display: grid;
		gap: calc(var(--cote)/7);
	}

	.block {
		background-color: grey;
		border: 1px black solid;
		border-radius: 5px;
		display: block;
		position: relative;

		width: var(--cote);
		height: var(--cote);

		transition: width .1s;
	}

	[class*="border"] {
		position: absolute;
		border: 5px rgba(0, 255, 255, 0) solid;
		border-radius: 5px;
		transition: .3s ease;
	}
	[class*="border"]:hover {
		border-width: calc(var(--cote)/7);
		border-color: rgba(0, 255, 255, .7);
		cursor: pointer;
	}
	
	.border-top {
		bottom: 90%;
		transform: translateY(100%);
	}
	.border-right {
		left: 90%;
		transform: translateX(-100%);
	}
	.border-bottom {
		top: 90%;
		transform: translateY(-100%);
	}
	.border-left {
		right: 90%;
		transform: translateX(100%);
	}

	.border-top, .border-bottom {
		width: 90%;
		left: 50%;
		transform: translateX(-50%)
	}
	.border-right, .border-left {
		height: 90%;
		top: 50%;
		transform: translateY(-50%);
	}

</style>
