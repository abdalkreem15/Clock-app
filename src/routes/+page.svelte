<script lang="ts">
    import { writable } from "svelte/store";

    type Mode = "clock" | "countdown" | "stopwatch";

    // Local storage helpers
    function getLS<T>(key: string, fallback: T): T {
        try {
            const val = localStorage.getItem(key);
            return val !== null ? JSON.parse(val) : fallback;
        } catch {
            return fallback;
        }
    }
    function setLS<T>(key: string, value: T) {
        try {
            localStorage.setItem(key, JSON.stringify(value));
        } catch {}
    }

    // Stores
    const mode = writable<Mode>(getLS("mode", "clock"));
    const now = writable(new Date());

    // Get the initial countdown value from localStorage
    const savedCountdownValue = getLS("countdownSeconds", 10);
    // This will hold the currently displayed countdown value (which can change during countdown)
    const countdownSeconds = writable(savedCountdownValue);
    // This will hold the last user-entered value, for resetting
    let lastUserCountdownInput = $state(savedCountdownValue);

    const stopwatchMilliseconds = writable(0);

    const stopwatchRunning = writable(false);
    const countdownRunning = writable(false);

    let interval: ReturnType<typeof setInterval>;
    let stopwatchInterval: ReturnType<typeof setInterval>;
    let countdownInterval: ReturnType<typeof setInterval>;

    // Reference to the audio element
    let audioPlayer: HTMLAudioElement;

    // Persist mode and countdownSeconds to localStorage
    // Note: We'll persist the value in countdownSeconds store, which reflects the current state
    $effect(() => setLS("mode", $mode));
    $effect(() => setLS("countdownSeconds", $countdownSeconds));


    // Effect to update lastUserCountdownInput whenever the countdownSeconds store changes
    // due to user input (via bind:value)
    $effect(() => {
        if ($mode === "countdown" && !$countdownRunning) {
            // Only update lastUserCountdownInput if the countdown is not running,
            // meaning the change came from the user input field
            lastUserCountdownInput = $countdownSeconds;
        }
    });

    // Clock
    function startClock() {
        stopAllIntervals();
        interval = setInterval(() => {
            now.set(new Date());
        }, 1000);
    }

    // Stopwatch
    function startStopwatch() {
        stopAllIntervals();
        stopwatchRunning.set(true);
        const startTime = Date.now() - $stopwatchMilliseconds;

        stopwatchInterval = setInterval(() => {
            stopwatchMilliseconds.set(Date.now() - startTime);
        }, 10);
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

    // Countdown
    function startCountdown() {
        stopAllIntervals();
        countdownRunning.set(true);
        countdownInterval = setInterval(() => {
            countdownSeconds.update(n => {
                if (n > 0) return n - 1;
                else {
                    clearInterval(countdownInterval);
                    countdownRunning.set(false);
                    // Play sound when countdown reaches zero
                    if (audioPlayer) {
                        audioPlayer.play();
                    }
                    return 0;
                }
            });
        }, 1000);
    }

    function resetCountdown() {
        clearInterval(countdownInterval);
        // Reset to the last value the user input, or the initial saved value
        countdownSeconds.set(lastUserCountdownInput);
        countdownRunning.set(false);
    }

    function stopAllIntervals() {
        clearInterval(interval);
        clearInterval(stopwatchInterval);
        clearInterval(countdownInterval);
        stopwatchRunning.set(false);
        countdownRunning.set(false);
    }

    // React to mode changes
    $effect(() => {
        if ($mode === "clock") {
            startClock();
        } else {
            stopAllIntervals();
        }
    });

    // Format helpers
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
        <button
            onclick={() => mode.set("clock")}
            class:active={$mode === "clock"}
            disabled={$mode === "clock"}
        >
            Clock
        </button>
        <button
            onclick={() => mode.set("countdown")}
            class:active={$mode === "countdown"}
            disabled={$mode === "countdown"}
        >
            Countdown
        </button>
        <button
            onclick={() => mode.set("stopwatch")}
            class:active={$mode === "stopwatch"}
            disabled={$mode === "stopwatch"}
        >
            Stopwatch
        </button>
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
        <button onclick={resetCountdown} disabled={!$countdownRunning && $countdownSeconds === lastUserCountdownInput}>
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

<audio bind:this={audioPlayer} src="/audio/ding.mp3" preload="auto"></audio>

<style>
    /* ... (your existing styles remain the same) ... */

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