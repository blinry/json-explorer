<script>
    let header = ""
    let data = ""

    async function getData(url) {
        header = url
        data = ""
        window.location.hash = url
        fetch(url, {
            headers: {
                Accept: 'application/ld+json; profile="https://www.w3.org/ns/activitystreams"',
                "Access-Control-Allow-Origin": "*",
                mode: "no-cors",
            },
        }).then(async (response) => {
            let json = await response.json()
            // pretty print
            data = urlify(JSON.stringify(json, null, 4))
        })
    }

    function urlify(text) {
        let urlRegex = /(https?:\/\/[^\s,"\\]+)/g
        return text.replace(urlRegex, function (url) {
            console.log(url)
            // wrap in tag that, when clicked, calls getData
            return `<a href="#${url}">${url}</a>`
        })
    }

    function hashChange() {
        getData(window.location.hash.substring(1))
    }

    // is the hash non-empty?
    if (window.location.hash) {
        hashChange()
    } else {
        getData("https://chaos.social/users/blinry")
    }
</script>

<svelte:window on:hashchange={hashChange} />

<main>
    <input
        type="text"
        bind:value={header}
        on:keydown={(e) => {
            if (e.key === "Enter") {
                getData(header)
            }
        }}
    />
    <div class="json">{@html data}</div>
</main>

<style>
    input {
        font-size: 120%;
        width: 100%;
        margin-bottom: 1rem;
        padding: 0.5rem;
    }
    .json {
        font-family: monospace;
        white-space: pre-wrap;
        text-align: left;
        background: #eee;
        border-radius: 1rem;
        padding: 1rem;
    }
</style>
