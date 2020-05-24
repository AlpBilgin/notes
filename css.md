grid-template-columns: repeat( auto-fill, minmax(min(calc(180px + 12vmin), 100%), 1fr));



<section class="grid">
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
</section>


section {
  display: grid;
  grid-template-columns: repeat( auto-fill, minmax(min(calc(180px + 12vmin), 100%), 1fr) );
  border: var(--border);
}

div {
  background-color: #eee;
  height: 0;
  padding-bottom: 100%;
  border: var(--border);
}

:root {
  --border: 2px solid #777;
}

