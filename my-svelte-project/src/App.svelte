<script>
    import { onMount } from 'svelte';

    const endpoint = 'https://cotton-concrete-catsup.glitch.me';

    let number0 = 1;
    let number1 = 1;
    let lexis = '';
    let bar = '';
    let baz = '';
    let sortOrder = 'desc';
    let language = 'en';
    let entries = [];

    function tiny_segmenter(lexis_str_param) {
        // TinySegmenterのインスタンス化を修正
        var segmenter = new TinySegmenter();
        var segs = segmenter.segment(lexis_str_param);
        return segs;
    }

    function count() {
        let words;
        if (language === 'ja') {
            words = tiny_segmenter(lexis.trim());
        } else {
            words = lexis.trim().split(/[\s\n]+/);
        }

        const wordCounts = words.reduce((acc, word) => {
            acc[word] = (acc[word] || 0) + 1;
            return acc;
        }, {});

        entries = Object.entries(wordCounts);
        number1 = Math.max(1, ...entries.map(entry => entry[1]));
        updateDisplay();
    }

    function updateDisplay() {
        const filteredEntries = entries.filter(entry => entry[1] <= number1 && entry[1] >= number0);
        const sortedEntries = sortOrder === 'desc'
            ? filteredEntries.sort((a, b) => b[1] - a[1])
            : filteredEntries.sort((a, b) => a[1] - b[1]);

        bar = sortedEntries.map(entry => entry[0]).join('\n');
        baz = sortedEntries.map(entry => entry[1]).join('\n');
    }

    function setLexis() {
        if (language === 'ja') {
            lexis = constitution_of_japan;
        } else {
            lexis = english_text; // 変更
        }
        count();
    }

    function clearLexis() {
        if (confirm("Are you sure you want to clear the text?")) {
            lexis = '';
            bar = '';
            baz = '';
        }
    }

    async function createLexis() {
        try {
            const response = await fetch(`${endpoint}/api/lexis`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    lexis,
                    created: new Date().toISOString(),
                    updated: new Date().toISOString()
                }),
            });
            const result = await response.json();
            console.log('Lexis created successfully:', result);
        } catch (error) {
            console.error('Error creating lexis:', error);
        }
    }

    async function readLexis() {
        try {
            const response = await fetch(`${endpoint}/api/lexis`);
            const data = await response.json();
            entries = data;
            console.log('Lexis retrieved successfully:', data);
        } catch (error) {
            console.error('Error retrieving lexis:', error);
        }
    }

    async function signInWithGoogle() {
        // Googleサインインの処理はそのままにしておきます
    }

    async function signOut() {
        // サインアウトの処理はそのままにしておきます
    }

    async function initializeDatabase() {
        const password = prompt("Enter password to initialize the database:");
        try {
            const response = await fetch(`${endpoint}/api/init-database-lexis`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ password }),
            });
            const result = await response.text();
            console.log(result);
            try {
                const jsonResult = JSON.parse(result);
                if (response.ok) {
                    alert(jsonResult.message);
                } else {
                    alert(jsonResult.error);
                }
            } catch (e) {
                console.error('Failed to parse JSON:', e);
                alert('Error initializing database. Response was not in JSON format.');
            }
        } catch (error) {
            console.error('Error initializing database:', error);
            alert('Error initializing database.');
        }
    }

    onMount(() => {
        new ClipboardJS('.btn');
        setLexis();
    });
</script>

<style>
    .btn { margin: 5px; }
</style>

<main>
    <div>
        Language:
        <label>
            <input type="radio" bind:group={language} value="en" on:change={setLexis}> English
        </label>
        <label>
            <input type="radio" bind:group={language} value="ja" on:change={setLexis}> Japanese
        </label>
    </div>
    <div>
        under <input type="number" bind:value={number1} min="1" on:input={updateDisplay}>
    </div>
    <div>
        over <input type="number" bind:value={number0} min="1" on:input={updateDisplay}>
    </div>
    <div>
        Sort Order:
        <label>
            <input type="radio" bind:group={sortOrder} value="desc" on:change={updateDisplay}> DESC
        </label>
        <label>
            <input type="radio" bind:group={sortOrder} value="asc" on:change={updateDisplay}> ASC
        </label>
    </div>

    <div>
        <textarea bind:value={lexis} cols="50" rows="10" placeholder="foo bar foo baz" on:input={count}></textarea>
        <button class="btn" data-clipboard-target="#bar">COPY</button>
        <button class="btn" data-clipboard-target="#baz">COPY</button>
        <button class="btn" on:click={clearLexis}>CLEAR</button>
        <button class="btn" on:click={createLexis}>Save</button>
        <button class="btn" on:click={readLexis}>Load</button>
        <button class="btn" on:click={initializeDatabase}>Initialize Database</button>
    </div>
    <div>
        <textarea bind:value={bar} id="bar" cols="20" rows="10" readonly></textarea>
        <textarea bind:value={baz} id="baz" cols="20" rows="10" readonly></textarea>
    </div>
    <div>
        <button class="btn" on:click={signInWithGoogle}>Sign In with Google</button>
        <button class="btn" on:click={signOut}>Sign Out</button>
    </div>
</main>