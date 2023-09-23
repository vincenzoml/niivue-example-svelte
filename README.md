# Minimal example demonstrating some lags in UI when niivue at work

Pressing the button adds images to load to a queue. Niivue loads them asynchronously. However, the button starts lagging as the command queue grows. This is mitigated by adding delay instructions before starting to process images (see source code in src/routes/+page.svelte for the commented delay instructions). 
