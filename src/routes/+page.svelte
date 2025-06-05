<script lang="ts">
    import { writable } from "svelte/store";

    type Mode = "clock" | "countdown" | "stopwatch";

    const mode = writable<Mode>("clock");
    const now = writable(new Date());
    const countdownSeconds = writable(10);
    const stopwatchMilliseconds = writable(0); // in ms

    const stopwatchRunning = writable(false);
    const countdownRunning = writable(false);

    let interval: ReturnType<typeof setInterval>;
    let stopwatchInterval: ReturnType<typeof setInterval>;
    let countdownInterval: ReturnType<typeof setInterval>;

    function startClock() {
        stopAllIntervals();
        interval = setInterval(() => {
            now.set(new Date());
        }, 1000);
    }

    function startStopwatch() {
        stopAllIntervals();
        stopwatchRunning.set(true);
        const startTime = Date.now() - $stopwatchMilliseconds;

        stopwatchInterval = setInterval(() => {
            stopwatchMilliseconds.set(Date.now() - startTime);
        }, 10); // update every 10ms for smooth ms display
    }

    function pauseStopwatch() {
        stopwatchRunning.set(false);
        clearInterval(stopwatchInterval);
    }

    function resetStopwatch() {
        stopwatchRunning.set(false);
        clearInterval(stopwatchInterval);
        stopwatchMilliseconds.set(0);
    }

    function startCountdown() {
        stopAllIntervals();
        countdownRunning.set(true);
        countdownInterval = setInterval(() => {
            countdownSeconds.update(n => {
                if (n > 0) return n - 1;
                else {
                    clearInterval(countdownInterval);
                    countdownRunning.set(false);
                    return 0;
                }
            });
        }, 1000);
    }

    function resetCountdown() {
        clearInterval(countdownInterval);
        countdownSeconds.set(10);
        countdownRunning.set(false);
    }

    function stopAllIntervals() {
        clearInterval(interval);
        clearInterval(stopwatchInterval);
        clearInterval(countdownInterval);
        stopwatchRunning.set(false);
        countdownRunning.set(false);
    }

    $: if ($mode === "clock") {
        startClock();
    } else {
        stopAllIntervals();
    }

    // Format time helper for stopwatch with ms: HH:MM:SS.ms (two digits for ms)
    function formatStopwatch(ms: number) {
        const totalSeconds = Math.floor(ms / 1000);
        const h = Math.floor(totalSeconds / 3600)
            .toString()
            .padStart(2, "0");
        const m = Math.floor((totalSeconds % 3600) / 60)
            .toString()
            .padStart(2, "0");
        const s = (totalSeconds % 60).toString().padStart(2, "0");
        const milliseconds = Math.floor((ms % 1000) / 10)
            .toString()
            .padStart(2, "0");
        return `${h}:${m}:${s}.${milliseconds}`;
    }

    // Format time helper function (HH:MM:SS) for countdown
    function formatTime(seconds: number) {
        const h = Math.floor(seconds / 3600)
            .toString()
            .padStart(2, "0");
        const m = Math.floor((seconds % 3600) / 60)
            .toString()
            .padStart(2, "0");
        const s = (seconds % 60).toString().padStart(2, "0");
        return `${h}:${m}:${s}`;
    }
</script>

<header>
    <h1>Clock App</h1>

    <div class="mode-select">
        <label>
            <input type="radio" bind:group={$mode} value="clock" />
            Clock
        </label>
        <label>
            <input type="radio" bind:group={$mode} value="countdown" />
            Countdown
        </label>
        <label>
            <input type="radio" bind:group={$mode} value="stopwatch" />
            Stopwatch
        </label>
    </div>
</header>

<main>
    {#if $mode === "clock"}
        <h2>Current Time</h2>
        <p class="time-display">{$now.toLocaleTimeString()}</p>
    {:else if $mode === "countdown"}
        <h2>Countdown Timer</h2>
        <input
            type="number"
            min="1"
            max="3600"
            bind:value={$countdownSeconds}
            class="countdown-input"
            aria-label="Set countdown duration in seconds"
            disabled={$countdownRunning}
        />
        <p class="time-display">{formatTime($countdownSeconds)}</p>
        <button onclick={startCountdown} disabled={$countdownSeconds === 0 || $countdownRunning}>
            Start
        </button>
        <button onclick={resetCountdown} disabled={!$countdownRunning}>
            Reset
        </button>
    {:else if $mode === "stopwatch"}
        <h2>Stopwatch</h2>
        <p class="time-display">{formatStopwatch($stopwatchMilliseconds)}</p>
        {#if $stopwatchRunning}
            <button onclick={pauseStopwatch}>Pause</button>
        {:else}
            <button onclick={startStopwatch}>Start</button>
        {/if}
        <button onclick={resetStopwatch}>Reset</button>
    {/if}
</main>

<style>
    :global(body) {
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        background: #1e1e2f;
        color: #e0e0e0;
        margin: 0;
        padding: 0;
        display: flex;
        flex-direction: column;
        min-height: 100vh;
        align-items: center;
        justify-content: flex-start;
    }

    header {
        background-color: #2a2a3d;
        width: 100%;
        padding: 1rem 0;
        text-align: center;
        box-shadow: 0 2px 8px rgba(0,0,0,0.6);
        margin-bottom: 2rem;
    }

    h1 {
        margin: 0;
        font-weight: 700;
        font-size: 2.5rem;
    }

    .mode-select {
        margin-top: 1rem;
        display: flex;
        justify-content: center;
        gap: 2rem;
    }

    label {
        font-weight: 600;
        cursor: pointer;
        user-select: none;
    }

    input[type="radio"] {
        margin-right: 0.5rem;
        accent-color: #4caf50;
        cursor: pointer;
    }

    main {
        text-align: center;
        max-width: 360px;
        width: 100%;
        padding: 0 1rem;
    }

    h2 {
        font-weight: 700;
        margin-bottom: 0.5rem;
        font-size: 1.8rem;
    }

    .time-display {
        font-size: 3rem;
        font-weight: 700;
        letter-spacing: 0.1em;
        margin-bottom: 1rem;
        font-family: 'Courier New', Courier, monospace;
        color: #81d4fa;
    }

    .countdown-input {
        width: 100%;
        padding: 0.5rem;
        font-size: 1.2rem;
        border-radius: 6px;
        border: none;
        margin-bottom: 1rem;
        text-align: center;
        font-family: monospace;
        background-color: #323244;
        color: #e0e0e0;
        box-shadow: inset 0 0 5px rgba(0,0,0,0.7);
        transition: background-color 0.3s;
    }

    .countdown-input:disabled {
        background-color: #444459;
        color: #8888aa;
    }

    button {
        background-color: #4caf50;
        border: none;
        padding: 0.6rem 1.5rem;
        font-size: 1rem;
        font-weight: 600;
        border-radius: 6px;
        color: white;
        cursor: pointer;
        margin: 0 0.5rem 1rem 0.5rem;
        transition: background-color 0.3s ease;
        box-shadow: 0 3px 6px rgba(0,0,0,0.3);
    }

    button:hover:not(:disabled) {
        background-color: #388e3c;
    }

    button:disabled {
        background-color: #777777;
        cursor: not-allowed;
        box-shadow: none;
    }
</style>
