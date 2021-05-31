<script lang="ts">
    let modal = false;
    let gameInterval = 100;
    type Cell = 'empty' | 'alive';
    let optionIcon = 'fa-play';
    let grid: Cell[][] = [];
    let gridTotal = grid;
    let playInterval: number;
    let gridSize: { x: number; y: number };
    init();

    function init() {
        gridSize = {
            x:
                Math.floor(document.documentElement.clientWidth / 20) -
                Math.ceil(Math.ceil(document.documentElement.clientWidth / 20) / 20) +
                5,
            y:
                Math.floor(document.documentElement.clientHeight / 20) -
                Math.ceil(Math.ceil(document.documentElement.clientHeight / 20) / 20) +
                5
        };
        grid = [...Array(gridSize.y)].map(() => [...Array(gridSize.x)].map(() => 'empty'));
        update();
    }

    function update() {
        gridTotal = grid;
        grid = gridTotal;
    }

    function play() {
        clearInterval(playInterval);
        optionIcon = 'fa-pause';
        playInterval = setInterval(gameTick, gameInterval);
    }

    function handleModal() {
        if (optionIcon === 'fa-pause') {
            optionIcon = 'fa-play';
            clearInterval(playInterval);
        } else {
            modal = !modal;
        }
    }

    function gameTick() {
        for (let i = 0; i < gridSize.y; i++) {
            for (let j = 0; j < gridSize.x; j++) {
                const numNeighbours = getNeighbours(i, j);
                if ((grid[i][j] === 'alive' && numNeighbours < 2) || numNeighbours > 3) {
                    grid[i][j] = 'empty';
                } else if (grid[i][j] === 'empty' && numNeighbours === 3) {
                    grid[i][j] = 'alive';
                }
            }
        }
        update();
    }
    function getNeighbours(i: number, j: number): number {
        let count = 0;
        if (i + 1 < gridSize.y && grid[i + 1][j] === 'alive') {
            count++;
        }
        if (i + 1 < gridSize.y && j + 1 < gridSize.x && grid[i + 1][j + 1] === 'alive') {
            count++;
        }
        if (j + 1 < gridSize.x && grid[i][j + 1] === 'alive') {
            count++;
        }
        if (i !== 0 && j + 1 < gridSize.x && grid[i - 1][j + 1] === 'alive') {
            count++;
        }
        if (j !== 0 && grid[i][j - 1] === 'alive') {
            count++;
        }
        if (i !== 0 && j !== 0 && grid[i - 1][j - 1] === 'alive') {
            count++;
        }
        if (i !== 0 && grid[i - 1][j] === 'alive') {
            count++;
        }
        if (i + 1 < gridSize.y && j !== 0 && grid[i + 1][j - 1] === 'alive') {
            count++;
        }
        return count;
    }
</script>

<main class="w-screen h-screen font-sans">
    <div class="center">
        <div>
            {#each gridTotal as row, i}
                <div class="row">
                    {#each row as cell, k}
                        <div
                            on:click={() => {
                                grid[i][k] = grid[i][k] === 'alive' ? 'empty' : 'alive';
                                update();
                            }}
                            class={`square ${cell}`}
                        />
                    {/each}
                </div>
            {/each}
        </div>
    </div>
    {#if modal}
        <div class="fixed z-10 inset-0 overflow-y-auto" aria-labelledby="modal-title" role="dialog" aria-modal="true">
            <div class="flex items-end justify-center min-h-screen pt-4 px-4 pb-20 text-center sm:block sm:p-0">
                <div class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity" aria-hidden="true" />
                <span class="hidden inline-block align-middle h-screen" aria-hidden="true">&#8203;</span>
                <div
                    class="inline-block align-bottom bg-white rounded-lg text-left overflow-hidden shadow-xl transform transition-all my-8 align-middle max-w-7xl w-full"
                >
                    <div class="bg-white px-4 pt-5 pb-4 sm:p-6 sm:pb-4">
                        <div class="sm:flex sm:items-start">
                            <div class="mt-3 text-center sm:mt-0 sm:ml-4 sm:text-left">
                                <h1 class="text-4xl leading-3 font-medium text-gray-900" id="modal-title">
                                    Conway's Game of Life
                                </h1>
                                <div class="mt-10">
                                    <div class="mb-4">
                                        <label class="block text-gray-700 text-sm font-bold mb-2" for="gameInterval">
                                            Game interval
                                        </label>
                                        <input
                                            class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                                            id="gameInterval"
                                            type="number"
                                            placeholder="Game interval (ms)"
                                            min="10"
                                            value={gameInterval}
                                        />
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="bg-gray-50 px-4 py-3 sm:px-6 sm:flex sm:flex-row-reverse">
                        <button
                            on:click={() => {
                                modal = false;
                                play();
                            }}
                            type="button"
                            class="w-full inline-flex justify-center rounded-md border border-transparent shadow-sm px-4 py-2 bg-indigo-800 text-base font-medium text-white hover:bg-indigo-900 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500 sm:ml-3 sm:w-auto sm:text-sm"
                        >
                            Start
                        </button>
                        <button
                            on:click={() => (modal = false)}
                            type="button"
                            class="mt-3 w-full inline-flex justify-center rounded-md border border-gray-300 shadow-sm px-4 py-2 bg-white text-base font-medium text-gray-700 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:mt-0 sm:ml-3 sm:w-auto sm:text-sm"
                        >
                            Cancel
                        </button>
                    </div>
                </div>
            </div>
        </div>
    {/if}
    <button
        class="absolute bottom-3 right-3 py-2 px-4 bg-indigo-800 w-11 h-11 hover:bg-indigo-900 font-bold rounded-full"
        on:click={handleModal}
    >
        <i class={`fas ${optionIcon} text-white`} />
    </button>
    <button
        class="absolute bottom-16 right-3 py-2 px-4 bg-indigo-800 w-11 h-11 hover:bg-indigo-900 font-bold rounded-full"
        on:click={init}
    >
        <i class="fas fa-redo-alt text-white" />
    </button>
</main>

<style lang="postcss">
    @tailwind base;
    @tailwind components;
    @tailwind utilities;
    .center {
        display: flex;
        justify-content: center;
        align-items: center;
    }
    .row {
        display: flex;
    }
    .square {
        width: 20px;
        height: 20px;
        border: solid 1px rgba(255, 255, 255, 0.01);
    }
    .alive {
        background-color: #f8f9fc;
        box-shadow: inset 0 0 1px #f8f9fc, 0 0 5px #f8f9fc;
    }
</style>
