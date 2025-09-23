<script>
// Utility to get local ISO date string (YYYY-MM-DD)  
const pad = (n) => String(n).padStart(2, '0');
const localISO = (d = new Date()) => `${d.getFullYear()}-${pad(d.getMonth() + 1)}-${pad(d.getDate())}`;

// Basic app state
let user = { name: 'Adi' };

// Habit templates
let habitTemplates = [
    { id: 1, name: 'Quiet Morning' },
    { id: 2, name: 'Read for 15 mins' }
];

// Default entry factory
const createDefaultEntry = (dateString) => ({
    date: dateString,
    steps: 0,
    mood: 'Neutral',
    thought: '',
    sleep: { hours: 0, minutes: 0 },
    workout: { completed: false, type: 'Push' },
    medication: [],
    customHabits: habitTemplates.map(t => ({ ...t, completed: false }))
});

// Sample entries for demonstration
let entries = [
    { date: '2025-09-12', steps: 4310, sleep: { hours: 8, minutes: 0 }, mood: 'Unpleasant', workout: { completed: false, type: 'Legs' }, customHabits: [{ id: 1, name: 'Brushed Teeth', completed: true }, { id: 2, name: 'Read for 15 mins', completed: false }] },
    { date: '2025-09-15', steps: 12034, sleep: { hours: 7, minutes: 45 }, mood: 'Neutral', workout: { completed: true, type: 'Push' }, customHabits: [{ id: 1, name: 'Brushed Teeth', completed: true }, { id: 2, name: 'Read for 15 mins', completed: true }] },
    { date: '2025-09-18', steps: 8204, sleep: { hours: 7, minutes: 30 }, mood: 'Pleasant', workout: { completed: true, type: 'Push' }, customHabits: [{ id: 1, name: 'Brushed Teeth', completed: true }, { id: 2, name: 'Read for 15 mins', completed: true }] },
    { date: '2025-09-19', steps: 10593, sleep: { hours: 6, minutes: 15 }, mood: 'Slightly Pleasant', workout: { completed: false, type: 'Pull' }, customHabits: [{ id: 1, name: 'Brushed Teeth', completed: true }, { id: 2, name: 'Read for 15 mins', completed: false }] },
    { date: '2025-09-20', steps: 9500, sleep: { hours: 8, minutes: 15 }, mood: 'Pleasant', workout: { completed: true, type: 'Push' }, customHabits: [{ id: 1, name: 'Brushed Teeth', completed: true }, { id: 2, name: 'Read for 15 mins', completed: true }] },
    { date: '2025-09-21', steps: 11000, sleep: { hours: 7, minutes: 0 }, mood: 'Slightly Pleasant', workout: { completed: true, type: 'Legs' }, customHabits: [{ id: 1, name: 'Brushed Teeth', completed: true }, { id: 2, name: 'Read for 15 mins', completed: true }] },
    { date: '2025-09-22', steps: 3000, sleep: { hours: 6, minutes: 30 }, mood: 'Neutral', workout: { completed: false, type: 'Legs' }, customHabits: [{ id: 1, name: 'Brushed Teeth', completed: true }, { id: 2, name: 'Read for 15 mins', completed: false }] }
];

// Ensure today exists
let todayString = localISO();
if (!entries.find(e => e.date === todayString)) entries.push(createDefaultEntry(todayString));

// UI state
let selectedDate = todayString;
let dateForCalendar = new Date(todayString + 'T00:00:00');
let entryBeingEdited = {};
let newHabitName = '';
let saved = false;
let isLoggingWorkout = false;

// reactive month/year for calendar
$: displayMonth = dateForCalendar.getMonth();
$: displayYear = dateForCalendar.getFullYear();

// Keep entryBeingEdited in sync with selectedDate
$: {
    const original = entries.find(e => e.date === selectedDate);
    const base = original ? JSON.parse(JSON.stringify(original)) : createDefaultEntry(selectedDate);
    if (!base.customHabits) base.customHabits = [];
    habitTemplates.forEach(t => {
        if (!base.customHabits.find(h => h.id === t.id)) base.customHabits.push({ ...t, completed: false });
    });
    base.customHabits = base.customHabits.filter(h => habitTemplates.some(t => t.id === h.id));
    entryBeingEdited = base;
    isLoggingWorkout = false;
}

// Utility / actions
function formatDate(dateInput, options = { month: 'short', day: 'numeric', weekday: 'long' }) {
    const date = dateInput instanceof Date ? dateInput : new Date(dateInput + 'T00:00:00');
    return date.toLocaleDateString('en-US', options);
}

function changeMonth(amount) {
    dateForCalendar.setMonth(dateForCalendar.getMonth() + amount);
    dateForCalendar = new Date(dateForCalendar);
}

function changeDay(amount) {
    const d = new Date(selectedDate + 'T00:00:00');
    d.setDate(d.getDate() + amount);
    const newIso = localISO(d);
    selectedDate = newIso;
    dateForCalendar = new Date(newIso + 'T00:00:00');
}

function saveChanges() {
    const index = entries.findIndex(e => e.date === entryBeingEdited.date);
    const entryCopy = JSON.parse(JSON.stringify(entryBeingEdited));
    if (index !== -1) entries[index] = entryCopy;
    else entries.push(entryCopy);
    entries = entries.sort((a, b) => new Date(a.date) - new Date(b.date));
    saved = true;
    setTimeout(() => (saved = false), 2000);
}

function removeWorkout() {
    entryBeingEdited.workout = createDefaultEntry('').workout;
    isLoggingWorkout = false;
}

function saveWorkout() {
    isLoggingWorkout = false;
    entryBeingEdited.workout.completed = true;
}

function cancelWorkout() {
    isLoggingWorkout = false;
}

function addNewHabitTemplate() {
    if (newHabitName.trim() === '') return;
    const newHabit = { id: Date.now(), name: newHabitName.trim() };
    habitTemplates = [...habitTemplates, newHabit];
    entries.forEach(entry => {
        if (!entry.customHabits) entry.customHabits = [];
        if (!entry.customHabits.find(h => h.id === newHabit.id)) entry.customHabits.push({ ...newHabit, completed: false });
    });
    entries = entries;
    newHabitName = '';
}

function deleteHabitTemplate(idToDelete) {
    habitTemplates = habitTemplates.filter(t => t.id !== idToDelete);
    entries.forEach(entry => {
        if (entry.customHabits) entry.customHabits = entry.customHabits.filter(h => h.id !== idToDelete);
    });
    entries = entries;
}

// Build calendar grid (6 weeks)
$: calendarDays = (() => {
    const days = [];
    const firstDay = new Date(displayYear, displayMonth, 1);
    const startDate = new Date(firstDay);
    startDate.setDate(startDate.getDate() - firstDay.getDay());
    for (let i = 0; i < 42; i++) {
        const date = new Date(startDate);
        date.setDate(startDate.getDate() + i);
        const dateString = date.toISOString().split('T')[0];
        days.push({ date, dateString, isCurrentMonth: date.getMonth() === displayMonth, entry: entries.find(e => e.date === dateString) });
    }
    return days;
})();

// Summary data for the charts
$: summary = (() => {
    const sortedEntries = [...entries].sort((a, b) => new Date(a.date).getTime() - new Date(b.date).getTime());
    const todayISO = localISO();
    const isFutureSelected = new Date(selectedDate + 'T00:00:00') > new Date(todayISO + 'T00:00:00');

    const dataForGraphs = isFutureSelected
        ? sortedEntries
        : [...sortedEntries.filter(e => e.date !== selectedDate), entryBeingEdited].sort((a, b) => new Date(a.date).getTime() - new Date(b.date).getTime());

    const selectedIndex = dataForGraphs.findIndex(e => e.date === selectedDate);
    const startIndex = Math.max(0, selectedIndex - 6);
    const recentData = selectedIndex > -1 ? dataForGraphs.slice(startIndex, selectedIndex + 1) : dataForGraphs.slice(-7);

    const getLabel = entry => new Date(entry.date + 'T00:00:00').toLocaleDateString('en-US', { month: 'numeric', day: 'numeric' });

    const moodMap = { 'very pleasant': 6, pleasant: 5, 'slightly pleasant': 4, neutral: 3, 'slightly unpleasant': 2, unpleasant: 1, 'very unpleasant': 0 };
    const getMoodValue = moodStr => {
        if (!moodStr || typeof moodStr !== 'string') return 3;
        const key = moodStr.trim().toLowerCase();
        return moodMap.hasOwnProperty(key) ? moodMap[key] : 3;
    };

    const moodLinePoints = recentData.map((entry, index) => {
        const moodValue = getMoodValue(entry.mood);
        const x = recentData.length <= 1 ? 50 : (index / (recentData.length - 1)) * 100;
        const y = 100 - (moodValue / 6) * 100;
        return { x, y };
    });

    // Steps chart data 
    const stepSource = [...recentData];
    while (stepSource.length > 1 && (stepSource[stepSource.length - 1].steps || 0) === 0 && stepSource[stepSource.length - 1].date !== selectedDate) stepSource.pop();
    const maxSteps = Math.max(...stepSource.map(e => e.steps || 0), 5000);
    const stepChartData = stepSource.map((entry, index) => {
        const slotWidth = stepSource.length > 0 ? 100 / stepSource.length : 0;
        const barWidth = slotWidth * 0.6;
        const barPadding = slotWidth * 0.2;
        return { x: index * slotWidth + barPadding, y: 100 - ((entry.steps || 0) / maxSteps) * 100, label: getLabel(entry), width: barWidth };
    });
    const stepAxisLabels = [`${Math.round(maxSteps / 1000)}k`, `${Math.round(maxSteps / 2000)}k`];

    // Sleep chart data 
    const maxSleep = 10 * 60;
    const sleepSource = [...recentData];
    const totalMinutesOf = entry => (entry.sleep?.hours || 0) * 60 + (entry.sleep?.minutes || 0);
    while (sleepSource.length > 1 && totalMinutesOf(sleepSource[sleepSource.length - 1]) === 0 && sleepSource[sleepSource.length - 1].date !== selectedDate) sleepSource.pop();
    const sleepChartData = sleepSource.map((entry, index) => {
        const slotWidth = sleepSource.length > 0 ? 100 / sleepSource.length : 0;
        const barWidth = slotWidth * 0.6;
        const barPadding = slotWidth * 0.2;
        const totalMinutes = totalMinutesOf(entry);
        return { x: index * slotWidth + barPadding, y: 100 - (totalMinutes / maxSleep) * 100, label: getLabel(entry), width: barWidth };
    });
    const sleepAxisLabels = ['10hr', '5hr'];

    const stepLabels = stepChartData.map(d => d.label);
    const sleepLabels = sleepChartData.map(d => d.label);
    if (entries.find(e => e.date === todayISO)) {
        const todayLabel = getLabel({ date: todayISO });
        if (!stepLabels.includes(todayLabel)) stepLabels.push(todayLabel);
        if (!sleepLabels.includes(todayLabel)) sleepLabels.push(todayLabel);
    }

    return { moodLinePoints, stepChartData, stepAxisLabels, sleepChartData, sleepAxisLabels, chartLabels: recentData.map(getLabel), stepLabels, sleepLabels };
})();
</script>

<!-- Main Dashboard Layout -->
<main>
  <div class="dashboard-grid">
    <header class="card header">
        <h1>Welcome, {user.name} 👋</h1>
        <p>{formatDate(new Date(), { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' })}</p>
    </header>

    <section class="card calendar-card">
        <div class="calendar-header">
            <button on:click={() => changeMonth(-1)} title="Previous Month">‹</button>
            <h2>{dateForCalendar.toLocaleString('default', { month: 'long', year: 'numeric' })}</h2>
            <button on:click={() => changeMonth(1)} title="Next Month">›</button>
        </div>
        <div class="calendar-grid">
            {#each ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'] as day}<div class="day-name">{day}</div>{/each}
            {#each calendarDays as day (day.dateString)}
                <div class="day-cell" class:other-month={!day.isCurrentMonth} class:today={day.dateString === todayString} class:selected={day.dateString === selectedDate} on:click={() => selectedDate = day.dateString}>
                    <div class="day-number">{day.date.getDate()}</div>
                    {#if day.entry?.workout.completed}<div class="workout-dot" title="Workout completed"></div>{/if}
                </div>
            {/each}
        </div>
        <div class="journal-card">
            <label class="journal-label">Thought / Journal</label>
            <textarea class="journal-text" bind:value={entryBeingEdited.thought} placeholder="Write a short thought for this day..." rows="4"></textarea>
        </div>
    </section>

        <section class="card entry-form">
            <div class="editor-header">
                <button class="day-nav" title="Previous Day" on:click={() => changeDay(-1)}>‹</button>
                <h2>Editing: {formatDate(selectedDate)}</h2>
                <button class="day-nav" title="Next Day" on:click={() => changeDay(1)}>›</button>
            </div>
      <div class="form-content">
    <fieldset><label for="mood">Mood:</label><select id="mood" bind:value={entryBeingEdited.mood}>{#each ['Very Pleasant', 'Pleasant', 'Slightly Pleasant', 'Neutral', 'Slightly Unpleasant', 'Unpleasant', 'Very Unpleasant'] as mood}<option value={mood}>{mood}</option>{/each}</select></fieldset>
        <fieldset><label for="steps">Steps: <strong>{entryBeingEdited.steps?.toLocaleString() || 0}</strong></label><input class="slider" id="steps" type="range" bind:value={entryBeingEdited.steps} min="0" max="25000" step="100"/></fieldset>
        <fieldset class="form-grid">
            <legend>Health Stats</legend>
            <div class="grid-item"><label>Sleep:</label><div class="sleep-inputs"><input type="number" bind:value={entryBeingEdited.sleep.hours} min="0" max="24" placeholder="hr"/><span>hr</span><input type="number" bind:value={entryBeingEdited.sleep.minutes} min="0" max="59" placeholder="min"/><span>min</span></div></div>
            <div class="grid-item">
                <label>Workout:</label>
                {#if isLoggingWorkout}
                    <div class="workout-form"><select bind:value={entryBeingEdited.workout.type}><option value="Push">Push</option><option value="Pull">Pull</option><option value="Legs">Legs</option></select><button class="icon-btn" on:click={saveWorkout} title="Save Workout">✔</button><button class="icon-btn danger" on:click={cancelWorkout} title="Cancel">✖</button></div>
                {:else if entryBeingEdited.workout.completed}
                    <div class="workout-summary"><span>{entryBeingEdited.workout.type} Workout</span><div class="summary-actions"><button class="icon-btn" on:click={() => isLoggingWorkout = true} title="Edit Workout">✏️</button><button class="icon-btn danger" on:click={removeWorkout} title="Remove Workout">🗑️</button></div></div>
                {:else} <button class="log-btn" on:click={() => isLoggingWorkout = true}>Log Workout</button> {/if}
            </div>
        </fieldset>
        <fieldset>
            <legend>Daily Habits</legend>
            <div class="habits-list">
                {#each habitTemplates as template (template.id)}
                    {@const habit = entryBeingEdited.customHabits?.find(h => h.id === template.id) || {}}
                    <div class="habit-item">
                        <label class="checkbox-label"><input type="checkbox" bind:checked={habit.completed}>{template.name}</label>
                        <button class="delete-habit-btn" on:click={() => deleteHabitTemplate(template.id)} title="Delete This Habit Permanently">🗑️</button>
                    </div>
                {/each}
            </div>
            <div class="add-habit-form"><input type="text" bind:value={newHabitName} placeholder="Add a new habit..." on:keydown={(e) => e.key === 'Enter' && addNewHabitTemplate()}/><button class="add-btn" on:click={addNewHabitTemplate}>+</button></div>
        </fieldset>
      </div>
      <div class="form-footer"><button class="save-btn" on:click={saveChanges}>Save Changes</button>{#if saved}<p class="save-feedback">✅ Changes saved!</p>{/if}</div>
    </section>

    <section class="card stats">
      <h2>Summary</h2>
      <div class="stats-content">
        <div class="chart-wrapper">
            <h3>Mood Over Time</h3>
            <div class="chart-with-axis">
                <div class="y-axis-labels"><span>V. Pleasant</span><span>Pleasant</span><span>Neutral</span><span>Unpleasant</span><span>V. Unpleasant</span></div>
                <div class="svg-container">
                    <svg class="mood-chart-svg" viewBox="0 0 100 100" preserveAspectRatio="none">
                        <polyline class="mood-line" points={summary.moodLinePoints.map(p => `${p.x},${p.y}`).join(' ')} />
                    </svg>
                </div>
            </div>
            <div class="x-axis-labels-mood">{#each summary.chartLabels as label}<span>{label}</span>{/each}</div>
        </div>

        <div class="chart-wrapper">
            <h3>Steps</h3>
            <div class="chart-with-axis">
                <div class="y-axis-labels-bar"><span>{summary.stepAxisLabels[0]}</span><span>{summary.stepAxisLabels[1]}</span><span>0</span></div>
                <div class="svg-container">
                    <svg class="bar-chart-svg" viewBox="0 0 100 100" preserveAspectRatio="none">
                        {#each summary.stepChartData as item}
                            <rect x={item.x} y={item.y} width={item.width} height={100 - item.y} rx="2"/>
                        {/each}
                    </svg>
                </div>
            </div>
            <div class="x-axis-labels-bar">{#each summary.stepLabels as label}<span>{label}</span>{/each}</div>
        </div>

        <div class="chart-wrapper">
            <h3>Sleep</h3>
            <div class="chart-with-axis">
                <div class="y-axis-labels-bar"><span>{summary.sleepAxisLabels[0]}</span><span>{summary.sleepAxisLabels[1]}</span><span>0hr</span></div>
                <div class="svg-container">
                    <svg class="bar-chart-svg" viewBox="0 0 100 100" preserveAspectRatio="none">
                        {#each summary.sleepChartData as item}
                            <rect x={item.x} y={item.y} width={item.width} height={100 - item.y} rx="2"/>
                        {/each}
                    </svg>
                </div>
            </div>
            <div class="x-axis-labels-bar">{#each summary.sleepLabels as label}<span>{label}</span>{/each}</div>
        </div>
      </div>
    </section>
  </div>
</main>

<style>
        :root {
            --bg-color: #121212;
            --card-color: #1E1E1E;
            --text-color: #EAEAEA;
            --text-muted: #A0A0A0;
            --primary-color: #98f6f4;
            --border-color: #333333;
            --y-axis-width: 56px;
            --chart-height: 120px;
        }
    main {
      font-family: 'Inter', sans-serif; background-color: var(--bg-color); color: var(--text-color);
      min-height: 100vh; display: grid; place-items: center; padding: 1rem; box-sizing: border-box;
    }
            .dashboard-grid {
                display: grid;
                grid-template-areas: "header header header" "summary editor calendar";
                grid-template-columns: 1.3fr 1.7fr 320px;
                grid-template-rows: auto minmax(0, 1fr);
                gap: 1.5rem; width: 100%; max-width: 1600px; height: calc(100vh - 2rem);
            }
    .card { background-color: var(--card-color); border-radius: 16px; padding: 1.5rem; border: 1px solid var(--border-color); display: flex; flex-direction: column; overflow: hidden; }
    h2 { margin-top: 0; border-bottom: 1px solid var(--border-color); padding-bottom: 0.5rem; color: var(--primary-color); font-size: 1.2rem; }
    .header { grid-area: header; }
    .calendar-card { grid-area: calendar; }
    .entry-form { grid-area: editor; }
    .stats { grid-area: summary; }
    .header h1 { margin: 0; font-size: 2rem; }
    .header p { margin: 0.25rem 0 0; color: var(--text-muted); font-size: 1rem; }
    .calendar-card { gap: 1rem; padding-top: 1.25rem; }
    .calendar-header { position: relative; display: flex; justify-content: center; align-items: center; padding: 0 0.5rem; }
    .calendar-header h2 { font-size: 1.1rem; border: none; padding: 0; margin: 0; text-align: center; }
    .calendar-header button { background: none; border: none; color: var(--primary-color); font-size: 2rem; cursor: pointer; line-height: 1; position: absolute; top: 50%; transform: translateY(-50%); }
    .calendar-header button[title="Previous Month"] { left: 0.5rem; }
    .calendar-header button[title="Next Month"] { right: 0.5rem; }
    .calendar-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 0.25rem; }
    .journal-card { margin-top: 0.75rem; display: flex; flex-direction: column; gap: 0.5rem; }
    .journal-label { color: var(--text-muted); font-weight: 600; font-size: 0.9rem; }
    .journal-text { width: 100%; min-height: 88px; resize: vertical; background-color: #2a2a2a; border: 1px solid var(--border-color); border-radius: 8px; padding: 0.75rem; color: var(--text-color); }
    
    .editor-header { position: relative; display: flex; justify-content: center; align-items: center; gap: 0.75rem; }
    .editor-header h2 { margin: 0; }
    .editor-header .day-nav { background: none; border: none; color: var(--primary-color); font-size: 1.6rem; cursor: pointer; line-height: 1; position: absolute; top: 50%; transform: translateY(-50%); }
    .editor-header .day-nav[title="Previous Day"] { left: 0.5rem; }
    .editor-header .day-nav[title="Next Day"] { right: 0.5rem; }
    .day-name { text-align: center; font-size: 0.75rem; color: var(--text-muted); }
    .day-cell { position: relative; display: flex; align-items: center; justify-content: center; aspect-ratio: 1/1; border-radius: 50%; cursor: pointer; border: 2px solid transparent; }
    .day-cell.other-month { color: var(--text-muted); }
    .day-cell:not(.other-month):hover { background-color: #2a2a2a; }
    .day-cell.today { border-color: var(--text-muted); font-weight: bold; }
    .day-cell.selected { border-color: var(--primary-color); background-color: rgba(52, 211, 153, 0.1); }
    .workout-dot { position: absolute; bottom: 5px; width: 6px; height: 6px; background-color: var(--primary-color); border-radius: 50%; }
    .entry-form { justify-content: space-between; }
    .form-content { display: flex; flex-direction: column; gap: 1rem; overflow-y: auto; padding-right: 1rem; margin-right: -1rem; }
    .form-footer { margin-top: 1rem; }
    label { font-weight: 600; color: var(--text-muted); font-size: 0.9rem; }
    input, select, textarea { width: 100%; background-color: #2a2a2a; border: 1px solid var(--border-color); border-radius: 8px; padding: 0.75rem; color: var(--text-color); font-size: 1rem; box-sizing: border-box; }
    .slider { padding: 0; -webkit-appearance: none; appearance: none; height: 8px; background: #333; border-radius: 5px; outline: none; }
    .slider::-webkit-slider-thumb { -webkit-appearance: none; appearance: none; width: 20px; height: 20px; background: var(--primary-color); cursor: pointer; border-radius: 50%;}
    .save-btn { width: 100%; background: var(--primary-color); color: var(--bg-color); padding: 0.75rem; border: none; border-radius: 8px; cursor: pointer; font-weight: 700; font-size: 1rem; }
    .save-feedback { text-align: center; color: var(--primary-color); margin-top: 0.5rem; height: 1em; }
    fieldset { border: 1px solid var(--border-color); border-radius: 8px; padding: 1rem; margin: 0; }
    legend { color: var(--primary-color); padding: 0 0.5rem; font-weight: 600; font-size: 0.9rem; }
    .form-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; align-items: start; }
    .sleep-inputs { display: flex; gap: 0.5rem; align-items: center; }
    .sleep-inputs input { text-align: center; }
    .sleep-inputs span { color: var(--text-muted); }
    .log-btn { background-color: #2a2a2a; border: 1px solid var(--border-color); color: var(--text-color); width: 100%; padding: 0.75rem; border-radius: 8px; cursor: pointer; height: 100%; }
    .workout-summary, .workout-form { display: flex; align-items: center; justify-content: space-between; background-color: #2a2a2a; padding: 0.5rem 0.75rem; border-radius: 8px; gap: 0.5rem; }
    .icon-btn { background: none; border: none; font-size: 1rem; cursor: pointer; padding: 0.25rem; }
    .icon-btn.danger { color: #F97066; }
    .habits-list { display: flex; flex-direction: column; gap: 0.75rem; }
    .habit-item { display: flex; align-items: center; justify-content: space-between; background-color: #2a2a2a; border-radius: 8px; transition: background-color 0.2s; }
    .habit-item:hover { background-color: #333; }
    .checkbox-label { display: flex; align-items: center; gap: 0.75rem; padding: 0.75rem; width: 100%; cursor: pointer; }
    .delete-habit-btn { background: none; border: none; color: var(--text-muted); cursor: pointer; font-size: 1.2rem; padding: 0 0.75rem; opacity: 0; transition: opacity 0.2s; }
    .habit-item:hover .delete-habit-btn { opacity: 1; }
    .add-habit-form { display: flex; gap: 0.5rem; margin-top: 1rem; }
    .add-btn { background: var(--primary-color); border: none; color: var(--bg-color); font-size: 1.5rem; width: 48px; border-radius: 8px; cursor: pointer; flex-shrink: 0;}
    .stats-content { overflow-y: auto; padding-right: 1rem; margin-right: -1rem; display: flex; flex-direction: column; gap: 1rem; }
    h3 { border-bottom: none; text-align: center; color: var(--text-muted); font-size: 1rem; font-weight: 600; margin: 0 0 0.5rem 0; }
    .chart-wrapper { display: flex; flex-direction: column; }
    .chart-with-axis { display: flex; gap: 0.75rem; align-items: stretch; }
    .y-axis-labels-bar { display: flex; flex-direction: column; justify-content: space-between; font-size: 0.7rem; color: var(--text-muted); text-align: right; width: var(--y-axis-width); flex-shrink: 0; padding-right: 0.5rem; height: var(--chart-height); box-sizing: border-box; }
    .bar-chart-svg { width: 100%; height: var(--chart-height); margin-bottom: 0.5rem; }
    .bar-chart-svg rect { fill: var(--primary-color); transition: all 0.3s ease-in-out; }
    .bar-chart-svg text { font-size: 8px; text-anchor: middle; fill: var(--text-muted); dominant-baseline: hanging; }
    .line-chart-wrapper { display: flex; height: 120px; }
    .y-axis-labels { display: flex; flex-direction: column; justify-content: space-between; font-size: 0.75rem; color: var(--text-muted); padding-right: 0.5rem; text-align: right; height: var(--chart-height); width: var(--y-axis-width); flex-shrink: 0; box-sizing: border-box; }
    .svg-container { flex-grow: 1; border-left: 1px solid var(--border-color); border-bottom: 1px solid var(--border-color); display: flex; align-items: stretch; }
    .mood-chart-svg { width: 100%; height: var(--chart-height); overflow: visible; }
    .mood-line { fill: none; stroke: var(--primary-color); stroke-width: 0.5; }
    .x-axis-labels-mood { display: flex; justify-content: space-between; font-size: 0.75rem; color: var(--text-muted); margin-left: calc(var(--y-axis-width) + 0.75rem); margin-top: 0.5rem; margin-right: 0.25rem; }
    svg text[transform] { font-size: 7px; fill: var(--text-muted); }
    .x-axis-labels-bar { display: flex; justify-content: space-between; margin-left: calc(var(--y-axis-width) + 0.75rem); margin-top: 0.5rem; margin-right: 0.25rem; }
    .x-axis-labels-bar span, .x-axis-labels-mood span { transform-origin: center; display: inline-block; transform: rotate(-30deg); font-size: 0.8rem; color: var(--text-muted); }
</style>
