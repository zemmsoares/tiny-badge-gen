<script>
    import opentype from "opentype.js";

    import archExample from "../assets/archExample.svelte";

    let font = null;

    let file;
    let badgeText = "";
    let logoScale = 0.18;
    let logoTranslateX = 0;
    let logoTranslateY = 0;
    let textTranslateX = 0;
    let textTranslateY = 0;
    let svgOutput = "";
    let svgUrl = "";
    let isBackgroundRed = false;

    // Load the font file
    opentype
        .load("./src/lib/liberation/LiberationSans-Regular.ttf")
        .then((loadedFont) => {
            font = loadedFont;
        });

    let spanStyle = "";

    $: {
        spanStyle = isBackgroundRed
            ? "background-color: none;"
            : "background-color: red";
    }

    let svgWidth = 64;
    let svgHeight = 10;

    async function updateSvg() {
        if (!file && !svgUrl) return;
        let content;
        if (file) {
            content = await file[0].text();
        } else if (svgUrl) {
            const response = await fetch(svgUrl);
            content = await response.text();
        }

        let parser = new DOMParser();
        let doc = parser.parseFromString(content, "image/svg+xml");

        let defsElement = doc.querySelector("defs");
        let defs = defsElement ? defsElement.outerHTML : "";

        let pathElements = doc.querySelectorAll("path");
        let paths = [];
        for (let i = 0; i < pathElements.length; i++) {
            let pathElement = pathElements[i];
            let path = pathElement.getAttribute("d");
            let color = pathElement.getAttribute("fill");
            paths.push({ path, color });
        }

        // Get path data for the badge text
        let badgeTextPath = font.getPath(
            badgeText,
            13 + textTranslateX,
            9 + textTranslateY,
            10
        );
        let badgeTextPathData = badgeTextPath.toPathData(2);

        let svgPaths = paths
            .map(
                ({ path, color }) => `
            <g fill-rule="evenodd" transform="translate(${logoTranslateX}, ${logoTranslateY}) scale(${logoScale})">
                <path d="${path}" fill="${color}" fill-rule="evenodd" />
            </g>
        `
            )
            .join("");
        svgOutput = `
        <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 ${svgWidth} ${svgHeight}" width="${svgWidth}" height="${svgHeight}">
            ${defs}
            <linearGradient id="smooth" x2="0" y2="100%">
                <stop offset="0" stop-color="#bbb" stop-opacity=".1"/>
                <stop offset="1" stop-opacity=".1"/>
            </linearGradient>
            <!-- Distro Logo -->
            ${svgPaths}
            <!-- Text on the badge -->
            <path d="${badgeTextPathData}" fill="#666666" />
        </svg>
    `;
    }

    function handleCreateBadge() {
        console.log(svgOutput);
    }
    function handleBackgroundRedChange() {
        updateSvg();
    }

    $: updateSvg(),
        file,
        svgUrl,
        badgeText,
        logoScale,
        logoTranslateX,
        logoTranslateY,
        textTranslateX,
        textTranslateY,
        svgWidth,
        svgHeight;
</script>

<form>
    <label>
        Upload SVG:
        <input type="file" bind:files={file} accept=".svg" />
    </label>or <br />
    <label>
        SVG URL:
        <input type="text" bind:value={svgUrl} />
    </label><br />
    <label>
        Badge Text:
        <input type="text" bind:value={badgeText} />
    </label><br />
    <label>
        SVG Width:
        <input type="number" bind:value={svgWidth} />
    </label><br />
    <label>
        SVG Height:
        <input type="number" bind:value={svgHeight} /> Must be 10 to be inline
    </label><br />
    <label>
        Logo Scale:
        <input type="number" bind:value={logoScale} step="0.001" />
    </label><br />
    <label>
        Logo Translate X:
        <input type="number" bind:value={logoTranslateX} step="0.1" />
    </label><br />
    <label>
        Logo Translate Y:
        <input type="number" bind:value={logoTranslateY} step="0.1" />
    </label><br />
    <label>
        Text Translate X:
        <input type="number" bind:value={textTranslateX} step="0.01" />
    </label><br />
    <label>
        Text Translate Y:
        <input type="number" bind:value={textTranslateY} step="0.01" />
    </label><br />
    <button type="button" on:click={handleCreateBadge}>Create Badge</button>
    <br /><br />
</form>

<div
    style="position:absolute;  border: 0.5px solid red;height: 15px; width: 12px"
/>
<div
    style="position:absolute;  border: 0.5px solid green;height: 15px; width: 1px; margin-left:13px"
/>
<div
    style="position: absolute; line-height:0; background-color: #eee; border: 1px solid #000; height: 10px"
>
    {@html svgOutput}
</div>

<div class="tip" markdown="1">
    <br />
    <br />
    <label style="font-size: 6px;">
        Make background red:
        <input
            type="checkbox"
            bind:checked={isBackgroundRed}
            on:change={handleBackgroundRedChange}
        />
    </label><br />
    <div class="parent-of-span-and-svg">
        zemmsoares&nbsp;<svelte:component this={archExample} />&nbsp;
        <span style={spanStyle}>{@html svgOutput}</span>&nbsp;
        <svelte:component this={archExample} />&nbsp;
        <svelte:component this={archExample} />
    </div>
</div>

<div style="font-size: 7px;">
    <br /><br />
    Tips :woof:
    <br />
    **Increase browser zoom to move stuff and see changes better<br />
    **dont move text up or down to make consistence to all badges<br />
    **Somehow margins must be consistent <br />
    **if svgs are not working well, the person who did them had no talent XD<br
    />
    **most of the ones i tried from https://commons.wikimedia.org/wiki/File:Void_Linux_logo.svg
    worked (Download > File URL:) **Copy the export from the console after "Create
    Badge" and save as svg
</div>
<br />

<style>
    span {
        display: inline-block;
        line-height: 0;
        vertical-align: middle;
        padding: 0;
        margin: 0;
    }

    span,
    svg {
        vertical-align: middle;
    }

    .parent-of-span {
        display: flex;
        align-items: flex-start;
    }

    .parent-of-span-and-svg {
        display: flex;
        align-items: center;
    }
</style>
