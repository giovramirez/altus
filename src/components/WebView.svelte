<script lang="ts">
  import { afterUpdate } from "svelte";
  import type { TabType } from "../types";
  import { themes } from "../store";
  export let partition;
  export let tab: TabType;

  let webviewElement;
  let hasStoppedLoading = false;

  const sendWebviewConfig = () => {
    webviewElement.send("set-id", tab.id);
    let currentTheme = $themes.find((theme) => theme.id === tab.config.theme);
    if (currentTheme) {
      webviewElement.send("set-theme", {
        name: tab.config.theme,
        ...(currentTheme.css
          ? {
              css: currentTheme.css,
            }
          : {}),
      });
    } else {
      webviewElement.send("set-theme", {
        name: tab.config.theme,
        css: "",
      });
    }
    webviewElement.send("toggle-notifications", tab.config.notifications);
    webviewElement.setAudioMuted(!tab.config.sound);
  };

  afterUpdate(() => {
    if (!hasStoppedLoading) {
      webviewElement.addEventListener("did-stop-loading", () => {
        hasStoppedLoading = true;
        sendWebviewConfig();
      });
    } else {
      sendWebviewConfig();
    }
  });
</script>

<webview
  id={`webview-${tab.id}`}
  src="https://web.whatsapp.com"
  preload="../src/preload.js"
  {partition}
  bind:this={webviewElement}
  webpreferences={`spellcheck=${tab.config.spellChecker ? 1 : 0}`}
  nodeintegration
/>
