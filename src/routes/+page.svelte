<script>
	export const prerender = true;
	
    import { fade } from "svelte/transition"
    import { Clock, Heart } from "lucide-svelte"

    const farmAnimals = ["🐶", "🐱", "🐷", "🐮", "🐔", "🐑", "🐴", "🐰", "🦆", "🐐"]

    let game = $state({
        cards: [],
        flippedIndices: [],
        matchedPairs: [],
        matches: 0,
        guesses: 0,
        timerInterval: null,
        timer: 0,
        isLoaded: true,
        completed: false,
    })

    /**
     * @param {string[]} array
     */
    function shuffleArray(array) {
        const shuffled = [...array, ...array]
		for (let i = shuffled.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
			[shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]]
        }
        return shuffled
    }

    /**
     * @param {number} index
     */
    function handleCardClick(index) {
        if (game.flippedIndices.length === 2) return
        if (game.flippedIndices.includes(index)) return
        if (game.matchedPairs.includes(game.cards[index])) return

        game.flippedIndices = [...game.flippedIndices, index]

        if (game.flippedIndices.length === 2) {
            game.guesses += 1

            if (game.cards[game.flippedIndices[0]] === game.cards[game.flippedIndices[1]]) {
                game.matches += 1
                game.matchedPairs = [...game.matchedPairs, game.cards[index]]
                game.flippedIndices = []

                if (game.matches === game.cards.length / 2) {
                    game.completed = true
                    clearInterval(game.timerInterval)
                }
            } else {
                setTimeout(() => {
                    game.flippedIndices = []
                }, 500)
            }
        }
    }

    /**
     * @param {number} seconds
     */
    function formatTime(seconds) {
        const minutes = Math.floor(seconds / 60)
        const remainingSeconds = seconds % 60
        return `${minutes}:${remainingSeconds.toString().padStart(2, "0")}`
    }

    function restartGame() {
        game.isLoaded = false
        game.cards = shuffleArray(farmAnimals)
        game.flippedIndices = []
        game.matchedPairs = []
        game.matches = 0
        game.guesses = 0
        game.timer = 0
        game.completed = false

        clearInterval(game.timerInterval)
        game.timerInterval = setInterval(() => {
            game.timer += 1
        }, 1000)
    }

    game.cards = shuffleArray(farmAnimals)
</script>

<main class="game-container">
    <div class="game-header">
        <h1 class="game-title">Farm Animal Matching Game</h1>
        <div class="game-stats">
            <div class="stat">
                <Clock size={28} />
                <span>{formatTime(game.timer)}</span>
            </div>
            <div class="stat">
                <Heart size={28} />
                <span>{game.matches} / {game.guesses}</span>
            </div>
        </div>
    </div>
    <div class="game-board">
        {#each game.cards as animal, index}
            <button
                class="card"
                class:flipped={game.flippedIndices.includes(index) || game.matchedPairs.includes(animal)}
                class:matched={game.matchedPairs.includes(animal)}
                onclick={() => handleCardClick(index)}
            >
                {#if game.flippedIndices.includes(index) || game.matchedPairs.includes(animal)}
                    <span class="animal">{animal}</span>
                {:else}
                    <div class="card-back"></div>
                {/if}
            </button>
        {/each}
    </div>

    <div class="footer">Created with Svelte 5</div>
</main>

{#if game.isLoaded}
    <div class="overlay" transition:fade>
        <div class="popup">
            <h2>Welcome</h2>
            <p>Hit the START button when your ready!</p>
            <button onclick={restartGame}>START</button>
        </div>
    </div>
{/if}

{#if game.completed}
    <div class="overlay" transition:fade>
        <div class="popup">
            <h2>Congratulations!</h2>
            <p>You've completed the game in {formatTime(game.timer)}!</p>
            <p>Total guesses: {game.guesses}</p>
            <button onclick={restartGame}>Play Again</button>
        </div>
    </div>
{/if}

<style>
    :global {
        body {
            margin: 0;
        }
    }

    .game-container {
        min-height: 100vh;
        background-color: #1e3a8a;
        background-image: url("bg.webp");
        background-size: auto 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    .game-header {
        background-color: #1c3036e3;
        padding: 1.5rem;
        border-radius: 0.5rem;
        border-color: white;
        border-style: solid;
        border-width: 2px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        margin-bottom: 1.5rem;
    }

    .game-title {
        font-size: 2.25rem;
        font-weight: bold;
        margin: 0;
        margin-bottom: 1rem;
        text-align: center;
        color: #ffffff;
    }

    .game-stats {
        display: flex;
        justify-content: space-between;
        align-items: center;
        font-size: 1.25rem;
        color: #fbff04;
    }

    .stat {
        display: flex;
        align-items: center;
    }

    .stat :global(svg) {
        margin-right: 0.5rem;
    }

    .game-board {
        display: grid;
        grid-template-columns: repeat(5, 1fr);
        gap: 1rem;
        padding: 1.5rem;
        background-color: #1c3036e3;
        border-radius: 0.5rem;
        border-color: white;
        border-style: solid;
        border-width: 2px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.5);
    }

    .card {
        width: 6rem;
        height: 6rem;
        background-color: white;
        border-radius: 0.5rem;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;
        transition: all 0.3s;
        border: none;
        padding: 0;
    }

    .card:hover {
        border: 4px solid #fbbf24;
    }

    .card.flipped {
        transform: rotateY(180deg);
    }

    .card.matched {
        opacity: 0.5;
    }

    .card-back {
        width: 100%;
        height: 100%;
        background-color: #e5e7eb;
        border-radius: 0.5rem;
    }

    .animal {
        font-size: 3rem;
    }

    .footer {
        padding: 15px;
        color: white;
        background-color: #333333a8;
        margin-top: 15px;
        font-size: 1rem;
        font-weight: bolder;
        border-radius: 0.5rem;
        border-color: white;
        border-style: solid;
        border-width: 2px;
    }

    .overlay {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.5);
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: 1000;
    }

    .popup {
        background-color: white;
        padding: 2rem;
        border-radius: 0.5rem;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        text-align: center;
    }

    .popup h2 {
        font-size: 1.5rem;
        font-weight: bold;
        margin-bottom: 1rem;
        color: #1e3a8a;
    }

    .popup p {
		margin-bottom: 1rem;
    }

    .popup button {
		background-color: #1e3a8a;
		color: white;
		border: none;
		padding: 0.5rem 1rem;
		border-radius: 0.25rem;
		cursor: pointer;
		font-size: 1rem;
		transition: background-color 0.3s;
    }

    .popup button:hover {
        background-color: #2563eb;
    }
</style>
