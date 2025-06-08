# Clock App

This is a simple yet versatile web application built with Svelte, offering three essential time-keeping modes: a **real-time clock**, a **countdown timer**, and a **stopwatch**. It's designed to be intuitive and visually pleasing across all devices.

## Features

* **Clock Mode**: Displays the current time, updated every second.

* **Countdown Timer Mode**: Set a duration in seconds, start the countdown, and get an audible alert when it reaches zero. The last entered value is remembered for easy resets.

* **Stopwatch Mode**: Start, pause, and reset a stopwatch that tracks time with millisecond precision.

* **Persistence**: Your last selected mode and countdown value are saved in your browser's local storage, so your settings are remembered when you revisit the app.

* **Responsive Design**: The app's layout and appearance adapt seamlessly to different screen sizes, ensuring a great user experience on mobile phones, tablets, and desktops.

## How to Use

1. **Switch Modes**: Use the buttons in the header (`Clock`, `Countdown`, `Stopwatch`) to toggle between the different time-keeping functions.

2. **Clock**: Just watch the time go by!

3. **Countdown**:

   * Enter your desired countdown duration (in seconds) into the input field.

   * Click the **Start** button to begin.

   * Once it hits zero, a "ding" sound will play.

   * Click **Reset** to return to your last entered value.

4. **Stopwatch**:

   * Click **Start** to begin timing.

   * Click **Pause** to temporarily stop the timer.

   * Click **Start** again to resume.

   * Click **Reset** to clear the stopwatch and set it back to zero.

## Technologies Used

* **Svelte 5**: For building the reactive user interface.

* **TypeScript**: For enhanced code quality and type safety.

* **CSS**: For custom styling and responsive design.