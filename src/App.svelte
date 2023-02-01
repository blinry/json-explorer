<script>
    String.prototype.escape = function () {
        var tagsToReplace = {
            "&": "&amp;",
            "<": "&lt;",
            ">": "&gt;",
        }
        return this.replace(/[&<>]/g, function (tag) {
            return tagsToReplace[tag] || tag
        })
    }

    let header = ""
    let data = ""

    async function getData(url) {
        header = url
        data = ""
        let originalUrl = url
        window.location.hash = url
        if (url.startsWith("https")) {
            url = url.replace("https://", "")
            url = url.replace("/", ":443/")
        }
        console.log(url)
        fetch(`https://cors.blinry.org/${url}`, {
            headers: {
                Origin: "bla",
                Accept: 'application/ld+json; profile="https://www.w3.org/ns/activitystreams"',
            },
        })
            .then(async (response) => {
                if (!response.ok) {
                    data = `Error: ${response.status} ${response.statusText}`
                    try {
                        let json = await response.json()
                        let prettyPrinted = JSON.stringify(
                            json,
                            null,
                            4
                        ).escape()
                        data += "\n\n" + urlify(prettyPrinted)
                    } catch (e) {}
                } else {
                    let contentType = response.headers.get("Content-Type")
                    if (!contentType.match(/json/)) {
                        data = `Content was delivered as <b>${contentType}</b>, I can only display JSON. Try opening the link externally: <a href="${originalUrl}" target="_blank">${originalUrl}</a>`
                    } else {
                        let json = await response.json()
                        let prettyPrinted = JSON.stringify(
                            json,
                            null,
                            4
                        ).escape()
                        data = urlify(prettyPrinted)
                    }
                }
            })
            .catch((error) => {
                data = error
            })
    }

    function urlify(text) {
        let urlRegex = /"(https?:\/\/[^\s,"\\]+)"/g
        return text.replace(urlRegex, function (match, url) {
            console.log(url)
            return `<span class="link" onclick="window.location.hash='${url}'" onauxclick="if (event.button == 1) window.open('${url}', '_blank')">${match}</span>`
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
    main {
        display: flex;
        flex-direction: column;
        height: 100vh;
        width: 100vw;
        padding: 1rem;
        box-sizing: border-box;
    }
    input {
        font-size: 120%;
        margin-bottom: 1rem;
        padding: 0.5rem;
    }
    .json {
        font-family: monospace;
        white-space: pre-wrap;
        text-align: left;
        padding: 1rem;
        overflow: auto;
        background: #111;
    }
    @media (prefers-color-scheme: light) {
        .json {
            background: #eee;
        }
    }
</style>
