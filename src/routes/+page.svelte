<script>
	export let data;

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
	<div class="grillage">
		{#each data.listOfCo as co}
			<div class="block" style="grid-row:{co.row}; grid-column: {co.col};">
				<div on:click={ajouterBlock} role="button" tabindex="0" on:keypress={handleKeyPress} class="border-top"></div>
				<div on:click={ajouterBlock} role="button" tabindex="0" on:keypress={handleKeyPress} class="border-right"></div>
				<div on:click={ajouterBlock} role="button" tabindex="0" on:keypress={handleKeyPress} class="border-bottom"></div>
				<div on:click={ajouterBlock} role="button" tabindex="0" on:keypress={handleKeyPress} class="border-left"></div>
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

	.grillage {
		display: grid;
		gap: 10px;
	}

	.block {
		background-color: grey;
		border: 1px black solid;
		border-radius: 5px;
		display: block;
		position: relative;

		width: var(--cote);
		height: var(--cote);
	}

	[class*="border"] {
		position: absolute;
		border: 5px rgba(0, 255, 255, 0) solid;
		border-radius: 5px;
		transition: .3s ease;
	}
	[class*="border"]:hover {
		border-width: 10px;
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
