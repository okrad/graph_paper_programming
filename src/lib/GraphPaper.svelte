<script>
	import { onMount } from 'svelte';
	import drawIcon from '../../assets/img/draw.svg?raw'
	import changeColorIcon from '../../assets/img/refresh.svg?raw'
	import arrowRightIcon from '../../assets/img/arrow-right.svg?raw'
	import arrowLeftIcon from '../../assets/img/arrow-left.svg?raw'
	import arrowDownIcon from '../../assets/img/arrow-down.svg?raw'
	import arrowUpIcon from '../../assets/img/arrow-up.svg?raw'

	export let rows = 16;
	export let cols = 16;

	let x = 0;
	let y = 0;
	let colorIdx = 1;
	let cmdIdx = -1;
	let expectRepeat = false;
	let inLoop = false;
	let showPlaceholder;

	let cmdList = [];

	let grid = Array(cols * rows).fill(0);

	onMount(async () => {
		load();
	});

	const move = (dir) => {
		switch(dir) {
			case 'left':
				if(x > 0)
					x--;
				break;
			case 'right':
				if(x < cols - 1)
					x++;
				break;
			case 'up':
				if(y > 0)
					y--;
				break;
			case 'down':
				if(y < rows - 1)
					y++;
				break;
		}

	}

	const draw = () => {
		grid[x + y * cols] = colorIdx;
		grid = grid;
	}

	const changeColor = () => {
		colorIdx = colorIdx % 2 + 1;
	}

	const clear = () => {
		grid.fill(0);
		colorIdx = 1;
		x = 0;
		y = 0;
		cmdIdx = -1;
		inLoop = false;
		expectRepeat = false;
		grid = grid;
	}

	const onKeyup = (evt) => {

		let cmd = null;

		if(expectRepeat && !isNaN(evt.key)) {
			// expectRepeat = false;
			if(cmdList[cmdIdx].repeat > 0) {
				cmdList[cmdIdx].repeat = cmdList[cmdIdx].repeat * 10 + parseInt(evt.key);
			}
			else {
				cmdList[cmdIdx].repeat = parseInt(evt.key);
			}
			// inLoop = false;
		}
		else {

			if(expectRepeat && cmdList[cmdIdx].repeat > 0) {
				expectRepeat = false;
				inLoop = false;
			}

			switch(evt.key) {
				case 'ArrowUp':
					cmd = {cmd: 'up'};
					break;

				case 'ArrowDown':
					cmd = {cmd: 'down'};
					break;

				case 'ArrowLeft':
					cmd = {cmd: 'left'};
					break;

				case 'ArrowRight':
					cmd = {cmd: 'right'};
					break;

				case 'c':
					cmd = {cmd: 'chgcol'};
					break;

				case 'm':
					cmd = {cmd: 'draw'};
					break;

				case '(':  case '.':
					cmd = {
						cmd: 'loop',
						repeat: 0,
						cmds: []
					};
					expectRepeat = true;
					break;

				case 'Backspace':
					if(inLoop) {
						if(cmdList[cmdIdx].cmds.length > 0) {
							cmdList[cmdIdx].cmds.splice(cmdList[cmdIdx].cmds.length - 1, 1);
						}
						else {
							cmdList.splice(cmdIdx, 1);
							inLoop = false;
						}
					}
					else {
						cmdList.splice(cmdIdx, 1);
					}
					break;
			}

			if(cmd) {
				if(cmdIdx >= 0 && inLoop)
					cmdList[cmdIdx].cmds.push(cmd);
				else
					cmdList.push(cmd);

				if(cmd.cmd == 'loop')
					inLoop = true;
			}
		}

		cmdIdx = cmdList.length - 1;
		cmdList = cmdList;

		// console.log(evt.key);
	}

	const execCmd = (c) => {
		switch(c.cmd) {
			case 'up':
			case 'down':
			case 'left':
			case 'right':
				move(c.cmd);
				break;

			case 'chgcol':
				changeColor();
				break;

			case 'draw':
				draw();
				break;

			case 'loop':
				for(let i = 0; i < c.repeat; i++) {
					c.cmds.forEach(sc => execCmd(sc));
				}
				break;
		}
	};

	const run = () => {
		clear();
		cmdList.forEach(c => execCmd(c));
	}

	const getCmdSymbol = (c) => {

		let s = '';

		switch(c.cmd) {
			case 'up':
				s = '<svg><use href="#icon:arrow-up"></use></svg>';
				break;
			case 'down':
				s = '<svg><use href="#icon:arrow-down"></use></svg>';
				break;
			case 'left':
				s = '<svg><use href="#icon:arrow-left"></use></svg>';
				break;
			case 'right':
				s = '<svg><use href="#icon:arrow-right"></use></svg>';
				break;
			case 'chgcol':
				s = '<svg><use href="#icon:chgcolor"></use></svg>';
				break;
			case 'draw':
				s = '<svg><use href="#icon:draw"></use></svg>';
				break;
			case 'loop':
				s = '<div class="block">' + c.cmds.map(sc => getCmdSymbol(sc)).join('<span class="dot">·</span>') + '</div><span class="mul">x' + c.repeat + '</span>';
				break;
		}

		return s;
	};

	const load = () => {
		const jsn = window.localStorage.getItem('graph_paper_programming');
		if(jsn)
			cmdList = JSON.parse(jsn);
	};

	const save = () => {
		window.localStorage.setItem('graph_paper_programming', JSON.stringify(cmdList));
	};

	const clearCmds = () => {
		clear();
		cmdList = [];
	};

	$: {
		showPlaceholder = cmdList.length == 0;
	}

</script>

<svg style="display:none" xmlns="http://www.w3.org/2000/svg">
	<symbol id="icon:draw">{@html drawIcon}</symbol>
	<symbol id="icon:chgcolor">{@html changeColorIcon}</symbol>
	<symbol id="icon:arrow-right">{@html arrowRightIcon}</symbol>
	<symbol id="icon:arrow-left">{@html arrowLeftIcon}</symbol>
	<symbol id="icon:arrow-down">{@html arrowDownIcon}</symbol>
	<symbol id="icon:arrow-up">{@html arrowUpIcon}</symbol>
</svg>

<p class="intro">
	Digita i comandi nel riquadro sottostante, poi premi il pulsante "esegui" per vedere il risultato.
</p>
<p>Comandi disponibili:</p>
<ul class="cmd-legend">
	<li>
		<span class="cmd-symbol"><svg><use href="#icon:arrow-up"></use></svg></span> Muoviti verso l'alto
	</li>
	<li>
		<span class="cmd-symbol"><svg><use href="#icon:arrow-down"></use></svg></span> Muoviti verso il basso
	</li>
	<li>
		<span class="cmd-symbol"><svg><use href="#icon:arrow-left"></use></svg></span> Muoviti a sinistra
	</li>
	<li>
		<span class="cmd-symbol"><svg><use href="#icon:arrow-right"></use></svg></span> Muoviti a destra
	</li>
	<li>
		<span class="cmd-symbol"><svg><use href="#icon:chgcolor"></use></svg></span> Cambia colore
	</li>
	<li>
		<span class="cmd-symbol"><svg><use href="#icon:draw"></use></svg></span> Colora il blocco corrente
	</li>
	<li>
		<span class="cmd-symbol">.</span> Inizia un loop. Tutti i comandi successivi faranno parte del ciclo, finchè non verrà digitato il numero di ripetizioni. Ad esempio ".<svg><use href="#icon:arrow-down"></use></svg>5" crea un ciclo che muove la posizione 5 blocchi in basso.
	</li>
	<li>
		<span class="cmd-symbol">Backspace</span> Cancella l'ultimo comando inserito
	</li>
</ul>

<section class="cnt">
	<div class="grid" style="grid-template-columns: repeat({cols}, 1fr);">
		{#each grid as col, idx}
			<div
				class:current={idx == (x + y * cols)}
				class="cell color{col}">
			</div>
		{/each}
	</div>

	<div class="cmd-area-cnt">
		<div class="cmd-area" contenteditable="true"
			on:click={() => {showPlaceholder = false}}
			on:blur={() => {showPlaceholder = cmdList.length == 0}}
			on:keydown|preventDefault={() => {}}
			on:keyup|preventDefault={onKeyup}>
			{#if cmdList.length > 0}
				{#each cmdList as c, idx}
					<div class="cmd" data-idx="idx">{@html getCmdSymbol(c)}</div>
					<span class="dot">·</span>
				{/each}
			{:else if showPlaceholder}
				<p class="placeholder">Digita qui i comandi...</p>
			{/if}
		</div>
		<div class="buttons">
			<div style="flex-grow: 1">
				<button on:click|preventDefault={run}>esegui</button>
				<button on:click|preventDefault={clearCmds}>pulisci</button>
			</div>
			<button on:click|preventDefault={save} style="margin-right: 0;">salva</button>
		</div>
	</div>
</section>

<style>
	p {
		margin: .5rem 0;
	}

	svg {
		width: 1rem;
		height: 1rem;
	}

	.cmd-legend {
		padding: 1rem 1rem;
		margin: 0;
		display: grid;
		grid-template-columns: 1fr 1fr;
		list-style-type: none;
		gap: .5rem;
	}

	.cmd-legend .cmd-symbol {
		background: #f1f1f1;
		border: 1px solid #ccc;
		border-radius: .2rem;
		padding: .2rem;
		min-width: 1rem;
		display: inline-block;
		text-align: center;
		margin-right: .5rem;
	}

	.cnt {
		display: flex;
		flex-flow: column;
		align-items: start;
		gap: 1rem;
		width: 100%;
	}

	.grid {
		display: grid;
		gap: 1px 1px	;
		background-color: #ccc;
	}

	.cell {
		width: 2rem;
		height: 2rem;
		background-color: #f8f8f8;
	}

	.color1 {
		background-color: #ccc;
	}

	.color2 {
		background-color: #777;
	}

	.cell.current {
		background-color:#0f03;
	}

	.cmd-area-cnt {
		display: flex;
		flex-flow: column;
		width: 100%;
		border: 1px solid #ccc;
		border-radius: .2rem;
	}

	.cmd-area {
		flex-grow: 1;
		font-size:1.5rem;
		word-break: break-all;
		font-family: monospace;

		display: flex;
		flex-flow: row wrap;
		align-items: center;
		gap: .2rem;
		min-height: 5rem;
	}

	.cmd-area:focus {
		outline: none;
		background-color: #f8f8f8;
		box-shadow: -1px -1px 3px rgba(0,0,0,.2) inset;
	}

	.placeholder {
		color: #555;
		font-size: 1rem;
		padding: .5rem;
	}

	:global(.cmd-area .cmd) {
		display: flex;
		align-items: center;
	}

	:global(.cmd-area .block) {
		border: 1px solid #ccc;
		border-radius: .2rem;
		padding: 0 .2rem;
		display: inline-flex;
		margin: .1rem;
		align-items: center;
		min-width: 1rem;
		min-height: 1em;
	}

	:global(.cmd-area .mul) {
		font-size: 1rem;
		color: #555;
	}

	:global(.cmd-area .dot) {
		font-size: 1.2rem;
		color: #555;
	}

	:global(.cmd-area svg) {
		width: 1rem;
		height: 1rem;
	}

	.buttons {
		display: flex;
		background-color: #f1f1f1;
		padding: .5rem;
		border-radius: 0 0 .2rem .2rem;
		gap: .5rem;
	}

	@media screen and (min-width: 85em) {
		.cnt {
			flex-flow: row;
		}
	}
</style>