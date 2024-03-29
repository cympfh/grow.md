<script lang="ts">
  import { onMount } from "svelte";
  import { filetree } from "./filetree";
  import Folder from "./components/Folder.svelte";

  const pathname = location.pathname;
  const slug = pathname.replace(/^\/*/, "").replace(/\/*$/, "");
  const slug_decoded = decodeURIComponent(slug).replace(/\+/g, " ");
  console.log(slug, slug_decoded);
  let content = "";
  let error = "";
  let tree = null;

  function mathjax() {
    let script = document.createElement("script");
    script.src = "https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js";
    document.head.append(script);
    script.onload = () => {
      MathJax = {
        tex: {
          inlineMath: [
            ["$", "$"],
            ["\\(", "\\)"],
          ],
        },
        svg: { fontCache: "global" },
      };
    };
  }

  function webemb() {
    let script = document.createElement("script");
    script.src = "https://cympfh.cc/resources/js/web_emb.js";
    document.head.append(script);
  }

  function youtube() {
    let script = document.createElement("script");
    script.src = "https://cympfh.cc/resources/js/youtube.js";
    document.head.append(script);
  }

  /// read a markdown
  function show() {
    error = `Fetching ${slug}`;
    fetch(`/api/content?slug=${slug}`)
      .then((res) => res.json())
      .then((res) => {
        if (res.ok) {
          content = res.html;
          error = "";
          mathjax();
          webemb();
          youtube();
        } else {
          error = res.error;
        }
      });
  }

  /// list menu
  function list() {
    let tag = '';
    if (location.hash) {
      tag = location.hash.slice(1);
    }
    fetch(`/api/list?tag=${tag}`)
      .then((res) => res.json())
      .then((paths) => {
        tree = filetree(paths, slug_decoded);
      });
  }

  onMount(async () => {
    list();
    if (pathname !== "/") {
      show();
    }
  });
</script>

<section class="hero">
  <div class="hero-body">
    <p class="title">grow.md</p>
    <p class="subtitle">{slug_decoded}</p>
  </div>
</section>
<div class="section">
  <div class="container">
    <div class="columns">
      <div class="column is-two-fifths">
        {#if tree}
          <Folder {tree} />
        {/if}
      </div>
      <div class="column">
        {#if error}
          <div class="notification is-danger">
            <button class="delete" />
            {error}
          </div>
        {:else}
          <div class="content">
            {@html content}
          </div>
        {/if}
      </div>
    </div>
  </div>
</div>

<style global lang="scss">
  @import "main.scss";
  @import "https://cympfh.cc/resources/css/web_emb.css";
  @import "https://cympfh.cc/resources/css/youtube.css";
  .menu-label {
    text-transform: none !important;
  }
  aside.menu svg {
    position: relative;
    top: 0.3rem;
  }
  div.web-emb {
    border: 1px #f0f0f0 solid !important;
    margin-bottom: 1em;
  }
</style>
