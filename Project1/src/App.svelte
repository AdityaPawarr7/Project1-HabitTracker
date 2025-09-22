<script>
  // User data
  // Not sure if we need it to be dynamically changed. But for now, it's static
  let user = {
    name: "Adi",
  };

  // Date
  let today = new Date().toLocaleDateString();

  // Activities
  let mood = "";
  let gratitude = "";
  let steps = 0;

  // Habit Tracker
  let habits = {
    water: false,
    exercise: false
  };

  let saved = false;

  function saveEntries() {
    saved = true;
    setTimeout(() => (saved = false), 3000);
    //We are not actually saving, this is just for visual purposes right now
  }
</script>

<main>
  <!-- User Info -->
  <section>
    <h1>Welcome, {user.name} 👋</h1>
    <p>Today’s Date: {today}</p>
  </section>

  <!-- Mood Selector & Gratitude -->
  <section>
    <div>
      <label for="mood">Mood for today:</label>
      <select id="mood" name="mood" bind:value={mood}>
        <option value="">-- Select --</option>
        <option>😊 Happy</option>
        <option>😔 Sad</option>
        <option>😡 Angry</option>
        <option>😴 Tired</option>
        <option>🤩 Excited</option>
      </select>
      {#if mood}
        <p>You’re feeling: <strong>{mood}</strong></p>
      {/if}
    </div>

    <div>
      <label for="gratitude">Gratitude:</label>
      <input
        id="gratitude"
        type="text"
        bind:value={gratitude}
        placeholder="Something you're thankful for"
      />
    </div>

    <div>
      <label for="steps">Steps walked today:</label>
      <input id="steps" type="number" bind:value={steps} step="500" />
    </div>
  </section>

  <!-- Habit Tracker -->
  <section>
    <h2>Habit Tracker</h2>

    <label class="flex items-center gap-2">
      <input type="checkbox" bind:checked={habits.water} />
      💧 Drank 8 glasses of water
    </label>

    <label class="flex items-center gap-2">
      <input 
      type="checkbox" bind:checked={habits.exercise} />
      🏃 Exercised
    </label>

  </section>

  <!-- Save Button -->
  <section>
    <button on:click={saveEntries}>Save Entries</button>
    {#if saved}
      <p>✅ Entries saved successfully!</p>
    {/if}
  </section>
</main>


<style>
  main {
    font-family: Arial, sans-serif;
    max-width: 800px;
    margin: 2rem auto;
    padding: 1rem;
    border: 1px solid #9efae0;
    border-radius: 8px;
  }
  section {
    margin-bottom: 1.5rem;
  }
  label {
    font-weight: bold;
    display: block;
    margin-bottom: 0.5rem;
  }
  input[type="text"],
  input[type="number"],
  select {
    margin-bottom: 1rem;
    padding: 0.3rem;
    width: 100%;
    box-sizing: border-box;
  }
  button {
    background: #44ee58;
    color: white;
    padding: 0.5rem 1rem;
    border: none;
    border-radius: 6px;
    cursor: pointer;
  }
  button:hover {
    background: #acf8ae;
  }
  p {
    margin-top: 0.5rem;
  }
  h2 {
    margin-bottom: 0.5rem;
  }
</style>