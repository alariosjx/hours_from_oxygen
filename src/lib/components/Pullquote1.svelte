<!--
  PullQuote.svelte
  Large typographic pull quote with gold left border and oversized opening mark.
  Drop anywhere in +page.svelte or inside DocRenderer as a block type "pullquote".
-->

<div class="pull-quote-wrap">
    <span class="open-mark">&ldquo;</span>
    <blockquote class="pull-quote">
      <p class="quote-text">It felt like an eternal trip. It felt like a haze of not knowing what would happen.</p>
      <footer class="quote-attribution">— Lourdes Jimenez, sister</footer>
    </blockquote>
  </div>
  
  <style>
    .pull-quote-wrap {
      position: relative;
      margin: 3rem 0;
      padding-left: 2rem;
      border-left: 3px solid #c8960a;
    }
  
    .open-mark {
      font-family: 'Crimson Text', Georgia, serif;
      font-size: clamp(5rem, 12vw, 9rem);
      line-height: 0;
      color: #c8960a;
      opacity: 0.25;
      position: absolute;
      top: 2.5rem;
      left: -1.5rem;
      pointer-events: none;
      user-select: none;
    }
  
    .pull-quote {
      margin: 0;
      padding: 0;
    }
  
    .quote-text {
      font-family: 'Crimson Text', Georgia, serif;
      font-size: clamp(1.4rem, 3vw, 2rem);
      font-style: italic;
      line-height: 1.5;
      color: #2a1a0e;
      margin: 0 0 1rem 0;
    }
  
    .quote-attribution {
      font-family: 'Crimson Text', Georgia, serif;
      font-size: clamp(0.85rem, 1.5vw, 1rem);
      letter-spacing: 0.12em;
      text-transform: uppercase;
      color: #8a6010;
      font-style: normal;
    }
  </style>
  