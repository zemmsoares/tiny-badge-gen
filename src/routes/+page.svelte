<script>
    import generated4 from "../assets/generated4.svelte";

    // final exports
    import arch from "../assets/export/arch.svelte";
    import neovim from "../assets/export/neovim.svelte";
    import Nixos from "../assets/export/nixos.svelte";
    import Artix from "../assets/export/artix.svelte";
    import Debian from "../assets/export/debian.svelte";
    import Fedora from "../assets/export/fedora.svelte";
    import Freebsd from "../assets/export/freebsd.svelte";
    import Gentoo from "../assets/export/gentoo.svelte";
    import Kali from "../assets/export/kali.svelte";
    import Manjaro from "../assets/export/manjaro.svelte";
    import Mint from "../assets/export/mint.svelte";
    import Ubuntu from "../assets/export/ubuntu.svelte";
    import Void from "../assets/export/void.svelte";
    import Xfce from "../assets/export/xfce.svelte";

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

    let spanStyle = "";

    $: {
        spanStyle = isBackgroundRed
            ? "background-color: none;"
            : "background-color: red";
        console.log(spanStyle);
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
    <g fill="#666666" text-anchor="left" font-family="DejaVu Sans,Verdana,Geneva,sans-serif" font-size="10">
        <text x="${13 + textTranslateX}" y="${
            9 + textTranslateY
        }">${badgeText}</text>
    </g>
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
        zemmsoares&nbsp;<svelte:component this={generated4} />&nbsp;
        <span style={spanStyle}>{@html svgOutput}</span>&nbsp;
        <svelte:component this={generated4} />&nbsp;
        <svelte:component this={generated4} />
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

<div>
    This Badges are Inline
    <svelte:component this={arch} />
    <svelte:component this={neovim} />
    <svelte:component this={Nixos} />
    <svelte:component this={Artix} />
    <svelte:component this={Debian} />
    <svelte:component this={Fedora} />
    <svelte:component this={Freebsd} />
    <svelte:component this={Gentoo} />
    <svelte:component this={Kali} />
    <svelte:component this={Manjaro} />
    <svelte:component this={Mint} />
    <svelte:component this={Ubuntu} />
    <svelte:component this={Void} />
    <svelte:component this={Xfce} />
</div>

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
