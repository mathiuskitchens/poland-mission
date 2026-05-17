<script>
  import GlobeScene from './lib/GlobeScene.svelte'
  import SpinWheel from './lib/SpinWheel.svelte'
  import DonateModal from './lib/DonateModal.svelte'

  let showModal = false
  let globeRef
  let polandFound = false

  function handleWhereIsPoland() {
    if (globeRef) {
      globeRef.flyToPoland()
    }
  }

  function onPinDropped() {
    polandFound = true
  }

  // --- UPDATE THESE AS FUNDRAISING PROGRESSES ---
  const GOAL      = 3700
  const RAISED    = 500      // dollars raised so far
  const JUNE_GOAL = 1850   // half by June 1
  // ------------------------------------------------

  $: progress     = Math.min((RAISED / GOAL) * 100, 100)
  $: juneProgress = Math.min((RAISED / JUNE_GOAL) * 100, 100)

  function fmt(n) {
    return n.toLocaleString('en-US', { style: 'currency', currency: 'USD', maximumFractionDigits: 0 })
  }

  // Smooth scroll helper
  function scrollTo(id) {
    document.getElementById(id)?.scrollIntoView({ behavior: 'smooth' })
  }
</script>

<!-- ── NAV ─────────────────────────────────────────────── -->
<nav>
  <div class="nav-inner">
    <div class="nav-brand">
      <span class="nav-cross">✝</span>
      <span class="nav-title">Poland Mission <span class="nav-year">2025</span></span>
    </div>
    <div class="nav-links">
      <button class="nav-link" on:click={() => scrollTo('mission')}>The Mission</button>
      <button class="nav-link" on:click={() => scrollTo('games')}>Spin the Wheel</button>
      <button class="btn-nav-donate" on:click={() => showModal = true}>Donate Now</button>
    </div>
  </div>
</nav>

<!-- ── HERO ──────────────────────────────────────────────── -->
<section class="hero" id="home">
  <div class="globe-bg">
    <GlobeScene bind:this={globeRef} on:pinDropped={onPinDropped} />
  </div>

  <div class="hero-overlay"></div>

  <div class="hero-content">
    <p class="eyebrow">Help Send Matthew to</p>
    <h1 class="hero-title" class:found={polandFound}>
      {#if polandFound}
        Poland Found<span class="question-mark">!</span>
      {:else}
        <button type="button" class="hero-title-button" on:click={handleWhereIsPoland}>
          Poland<span class="question-mark">?</span>
        </button>
      {/if}
    </h1>
    {#if polandFound}
      <p class="hero-tagline found-tagline">Serving communities, sharing hope, crossing oceans for the Gospel</p>
    {:else}
      <p class="hero-tagline">Click above to find out!</p>
    {/if}

    <!-- Progress Card -->
    <div class="progress-card">
      <div class="progress-top">
        <div>
          <p class="progress-label">Total Raised</p>
          <p class="progress-raised">{fmt(RAISED)}</p>
        </div>
        <div class="progress-right">
          <p class="progress-label">Goal</p>
          <p class="progress-goal">{fmt(GOAL)}</p>
        </div>
      </div>

      <div class="progress-track">
        <div class="progress-fill" style="width: {progress}%"></div>
        <!-- June 1 milestone marker at 50% -->
        <div class="milestone">
          <div class="milestone-pin"></div>
          <div class="milestone-chip">June 1<br/>{fmt(JUNE_GOAL)}</div>
        </div>
      </div>

      <div class="progress-foot">
        <span class="progress-pct">{Math.round(progress)}% of total goal</span>
        <span class="progress-june">June milestone: {Math.round(juneProgress)}%</span>
      </div>
    </div>

    <div class="hero-ctas">
      <button class="btn-primary" on:click={() => showModal = true}>Donate Now</button>
      <button class="btn-outline" on:click={() => scrollTo('games')}>Find My Amount</button>
    </div>
  </div>

  <div class="scroll-hint" on:click={() => scrollTo('mission')} on:keydown role="button" tabindex="0">
    <span>Scroll to learn more</span>
    <div class="scroll-chevron"></div>
  </div>
</section>

<!-- ── MISSION ─────────────────────────────────────────── -->
<section class="section" id="mission">
  <div class="container">
    <div class="section-head">
      <p class="eyebrow">The Mission</p>
      <h2>Why Poland?</h2>
    </div>

    <div class="mission-layout">
      <div class="mission-text">
        <p class="lead">
          This September, I'm joining a team traveling to Poland to serve communities in need — sharing hope, love, and the Gospel with people who are hungry for it.
        </p>
        <p>
          Poland sits at the crossroads of Europe and has become home to millions of Ukrainian refugees fleeing war. This has created one of the greatest mission opportunities of our generation: people displaced, hurting, and open to hearing about a God who loves them.
        </p>
        <p>
          Our team will partner with local churches and ministries to run outreach programs, serve at refugee centers, share meals, lead worship, and share the love of Christ in practical, tangible ways.
        </p>
        <p>
          I believe God has called me to be part of this. And I believe He's calling <em>you</em> to be part of it too — through your giving and your prayers.
        </p>
        <blockquote>
          "How beautiful are the feet of those who bring good news!"<br/>
          <cite>— Romans 10:15</cite>
        </blockquote>
      </div>

      <div class="mission-stats">
        <div class="stat-card">
          <div class="stat-icon">&#127757;</div>
          <h3>Where</h3>
          <p>Poland, Central Europe</p>
        </div>
        <div class="stat-card">
          <div class="stat-icon">&#128197;</div>
          <h3>When</h3>
          <p>September 2025</p>
        </div>
        <div class="stat-card highlight">
          <div class="stat-icon">&#127919;</div>
          <h3>Goal</h3>
          <p>{fmt(GOAL)} total<br/><small>{fmt(JUNE_GOAL)} by June 1</small></p>
        </div>
        <div class="stat-card">
          <div class="stat-icon">&#128591;</div>
          <h3>Impact</h3>
          <p>Every dollar sends hope across the world</p>
        </div>
      </div>
    </div>
  </div>
</section>

<!-- ── SPIN WHEEL ──────────────────────────────────────── -->
<section class="section games-section" id="games">
  <div class="container">
    <div class="section-head">
      <p class="eyebrow">Not Sure How Much To Give?</p>
      <h2>Let the Wheel Decide!</h2>
      <p class="section-desc">
        Spin it and see what amount lands on you. Take it as a sign — no pressure, just fun!
      </p>
    </div>
    <SpinWheel on:donate={() => showModal = true} />
  </div>
</section>

<!-- ── HOW TO GIVE ─────────────────────────────────────── -->
<section class="section give-section" id="give">
  <div class="container">
    <div class="section-head">
      <p class="eyebrow">Ready To Give?</p>
      <h2>It Only Takes a Minute</h2>
    </div>

    <div class="steps">
      <div class="step">
        <div class="step-num">1</div>
        <h3>Open the Giving Portal</h3>
        <p>Click <strong>Donate Now</strong> to go to our secure giving portal.</p>
      </div>
      <div class="step-arrow">&rarr;</div>
      <div class="step">
        <div class="step-num">2</div>
        <h3>Select <strong>Matthew Kitchens</strong></h3>
        <p>Choose my name from the drop-down so your gift is credited to my trip.</p>
      </div>
      <div class="step-arrow">&rarr;</div>
      <div class="step">
        <div class="step-num">3</div>
        <h3>Give Any Amount</h3>
        <p>Any amount makes a real difference. Need inspiration? Spin the wheel above!</p>
      </div>
    </div>

    <div class="give-cta">
      <button class="btn-primary btn-lg" on:click={() => showModal = true}>Donate Now</button>
      <p class="prayer-cta">
        Can't give right now? <strong>Please pray for our team.</strong><br/>
        Your prayers power this mission just as much as any gift.
      </p>
    </div>
  </div>
</section>

<!-- ── FOOTER ──────────────────────────────────────────── -->
<footer>
  <div class="footer-inner">
    <p class="footer-logo"><span class="footer-cross">✝</span> Poland Mission 2025</p>
    <p class="footer-name">Matthew Kitchens</p>
    <p class="footer-verse">"Go therefore and make disciples of all nations." — Matthew 28:19</p>
    <button class="footer-donate" on:click={() => showModal = true}>Donate Now</button>
  </div>
</footer>

{#if showModal}
  <DonateModal on:close={() => showModal = false} />
{/if}

<style>
  /* ── NAV ── */
  nav {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 100;
    background: rgba(7, 9, 26, 0.85);
    backdrop-filter: blur(12px);
    border-bottom: 1px solid rgba(212, 168, 67, 0.15);
  }

  .nav-inner {
    max-width: 1100px;
    margin: 0 auto;
    padding: 0.9rem 1.5rem;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
  }

  .nav-brand {
    display: flex;
    align-items: center;
    gap: 0.6rem;
  }

  .nav-cross {
    color: #d4a843;
    font-size: 1.1rem;
  }

  .nav-title {
    font-family: 'Cinzel', serif;
    font-size: 1rem;
    font-weight: 700;
    color: #fff;
    white-space: nowrap;
  }

  .nav-year {
    color: #d4a843;
  }

  .nav-links {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .nav-link {
    background: none;
    color: #aab8d8;
    font-size: 0.85rem;
    padding: 0.4rem 0.75rem;
    border-radius: 6px;
    transition: color 0.15s, background 0.15s;
  }

  .nav-link:hover {
    color: #fff;
    background: rgba(255, 255, 255, 0.06);
  }

  .btn-nav-donate {
    padding: 0.5rem 1.25rem;
    background: linear-gradient(135deg, #dc143c, #a00028);
    color: #fff;
    border-radius: 8px;
    font-size: 0.88rem;
    font-weight: 600;
    box-shadow: 0 2px 12px rgba(220, 20, 60, 0.4);
    transition: transform 0.15s, box-shadow 0.15s;
  }

  .btn-nav-donate:hover {
    transform: translateY(-1px);
    box-shadow: 0 4px 18px rgba(220, 20, 60, 0.55);
  }

  /* ── HERO ── */
  .hero {
    position: relative;
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
    padding: 5rem 1.5rem 4rem;
  }

  .globe-bg {
    position: absolute;
    inset: 0;
    z-index: 0;
  }

  .hero-overlay {
    position: absolute;
    inset: 0;
    z-index: 1;
    background: radial-gradient(ellipse at center, rgba(7, 9, 26, 0.15) 0%, rgba(7, 9, 26, 0.55) 65%, rgba(7, 9, 26, 0.92) 100%);
  }

  .hero-content {
    position: relative;
    z-index: 2;
    text-align: center;
    max-width: 680px;
    width: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1.5rem;
  }

  .eyebrow {
    font-size: 0.8rem;
    letter-spacing: 0.18em;
    text-transform: uppercase;
    color: #d4a843;
    font-weight: 600;
  }

  .hero-title {
    font-family: 'Cinzel', serif;
    font-size: clamp(2.8rem, 8vw, 5.5rem);
    font-weight: 900;
    line-height: 0.95;
    color: #fff;
    text-shadow: 0 0 60px rgba(212, 168, 67, 0.25), 0 4px 24px rgba(0, 0, 0, 0.6);
    letter-spacing: 0.04em;
  }

  .hero-title-button {
    font: inherit;
    color: inherit;
    background: none;
    border: 0;
    padding: 0;
    margin: 0;
    letter-spacing: inherit;
    line-height: inherit;
    text-shadow: inherit;
    cursor: pointer;
    display: inline;
    transition: color 0.3s, text-shadow 0.3s, transform 0.2s;
  }

  .hero-title-button:hover {
    color: #d4a843;
    text-shadow: 0 0 80px rgba(212, 168, 67, 0.5), 0 4px 24px rgba(0, 0, 0, 0.6);
    transform: scale(1.02);
  }

  .hero-title.found {
    color: #d4a843;
  }

  .question-mark {
    font-size: 1rem;
    display: inline-block;
    color: #d4a843;
    animation: bounce-q 2s ease-in-out infinite;
  }

  .found .question-mark {
    animation: none;
  }

  @keyframes bounce-q {
    0%, 100% { transform: translateY(0) rotate(0deg); }
    25% { transform: translateY(-4px) rotate(3deg); }
    75% { transform: translateY(2px) rotate(-2deg); }
  }

  .found-tagline {
    animation: fadeIn 0.8s ease-out;
  }

  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
  }

  .hero-tagline {
    font-size: clamp(0.9rem, 2vw, 1.05rem);
    color: #aab8d8;
    max-width: 460px;
    line-height: 1.6;
  }

  /* Progress card */
  .progress-card {
    background: rgba(13, 19, 51, 0.75);
    backdrop-filter: blur(16px);
    border: 1px solid rgba(212, 168, 67, 0.25);
    border-radius: 16px;
    padding: 1.5rem 1.75rem;
    width: 100%;
    max-width: 480px;
  }

  .progress-top {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin-bottom: 1rem;
  }

  .progress-right {
    text-align: right;
  }

  .progress-label {
    font-size: 0.72rem;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    color: #7a8db5;
    margin-bottom: 0.15rem;
  }

  .progress-raised {
    font-family: 'Cinzel', serif;
    font-size: 1.75rem;
    font-weight: 700;
    color: #d4a843;
    line-height: 1;
  }

  .progress-goal {
    font-size: 1.1rem;
    font-weight: 600;
    color: #aab8d8;
    line-height: 1;
  }

  .progress-track {
    position: relative;
    height: 10px;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 999px;
    overflow: visible;
    margin-bottom: 0.75rem;
  }

  .progress-fill {
    height: 100%;
    background: linear-gradient(90deg, #d4a843, #f0c868);
    border-radius: 999px;
    transition: width 1s ease;
    min-width: 4px;
  }

  .milestone {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
  }

  .milestone-pin {
    width: 2px;
    height: 18px;
    background: #dc143c;
  }

  .milestone-chip {
    background: rgba(220, 20, 60, 0.85);
    color: #fff;
    font-size: 0.62rem;
    font-weight: 700;
    padding: 2px 6px;
    border-radius: 4px;
    text-align: center;
    line-height: 1.3;
    white-space: nowrap;
    position: absolute;
    top: -38px;
  }

  .progress-foot {
    display: flex;
    justify-content: space-between;
    font-size: 0.75rem;
    color: #7a8db5;
  }

  /* CTA buttons */
  .hero-ctas {
    display: flex;
    gap: 0.75rem;
    flex-wrap: wrap;
    justify-content: center;
  }

  .btn-primary {
    padding: 0.85rem 2.2rem;
    background: linear-gradient(135deg, #dc143c 0%, #a00028 100%);
    color: #fff;
    border-radius: 10px;
    font-weight: 700;
    font-size: 1rem;
    box-shadow: 0 4px 20px rgba(220, 20, 60, 0.45);
    transition: transform 0.15s, box-shadow 0.15s;
  }

  .btn-primary:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 28px rgba(220, 20, 60, 0.6);
  }

  .btn-lg {
    padding: 1rem 2.8rem;
    font-size: 1.1rem;
  }

  .btn-outline {
    padding: 0.85rem 2.2rem;
    background: transparent;
    color: #fff;
    border: 1.5px solid rgba(255, 255, 255, 0.35);
    border-radius: 10px;
    font-weight: 600;
    font-size: 1rem;
    transition: border-color 0.15s, background 0.15s;
  }

  .btn-outline:hover {
    border-color: #d4a843;
    background: rgba(212, 168, 67, 0.08);
    color: #d4a843;
  }

  /* Scroll hint */
  .scroll-hint {
    position: absolute;
    bottom: 2rem;
    left: 50%;
    transform: translateX(-50%);
    z-index: 2;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.4rem;
    color: #7a8db5;
    font-size: 0.75rem;
    letter-spacing: 0.1em;
    cursor: pointer;
    animation: pulse 2.5s ease-in-out infinite;
  }

  .scroll-chevron {
    width: 20px;
    height: 20px;
    border-right: 2px solid #7a8db5;
    border-bottom: 2px solid #7a8db5;
    transform: rotate(45deg);
    margin-top: -4px;
  }

  @keyframes pulse {
    0%, 100% { opacity: 0.5; transform: translateX(-50%) translateY(0); }
    50%       { opacity: 1;   transform: translateX(-50%) translateY(4px); }
  }

  /* ── SECTIONS ── */
  .section {
    padding: 5rem 1.5rem;
  }

  .container {
    max-width: 1100px;
    margin: 0 auto;
  }

  .section-head {
    text-align: center;
    margin-bottom: 3.5rem;
  }

  .section-head h2 {
    font-family: 'Cinzel', serif;
    font-size: clamp(1.8rem, 4vw, 2.8rem);
    font-weight: 700;
    color: #fff;
    margin-top: 0.4rem;
  }

  .section-desc {
    margin-top: 0.75rem;
    color: #7a8db5;
    font-size: 1rem;
    max-width: 480px;
    margin-left: auto;
    margin-right: auto;
    line-height: 1.6;
  }

  /* ── MISSION ── */
  .mission-layout {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 3.5rem;
    align-items: start;
  }

  @media (max-width: 760px) {
    .mission-layout { grid-template-columns: 1fr; }
  }

  .mission-text p {
    color: #aab8d8;
    margin-bottom: 1rem;
    font-size: 0.97rem;
    line-height: 1.75;
  }

  .lead {
    font-size: 1.08rem !important;
    color: #ccd6f0 !important;
    font-weight: 500;
  }

  blockquote {
    border-left: 3px solid #d4a843;
    padding-left: 1rem;
    margin-top: 1.5rem;
    color: #aab8d8;
    font-style: italic;
    font-size: 0.92rem;
    line-height: 1.7;
  }

  cite {
    font-style: normal;
    color: #d4a843;
    font-size: 0.82rem;
  }

  .mission-stats {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
  }

  .stat-card {
    background: rgba(13, 19, 51, 0.6);
    border: 1px solid rgba(255, 255, 255, 0.07);
    border-radius: 14px;
    padding: 1.25rem 1rem;
    text-align: center;
    transition: border-color 0.2s, transform 0.2s;
  }

  .stat-card:hover {
    border-color: rgba(212, 168, 67, 0.3);
    transform: translateY(-2px);
  }

  .stat-card.highlight {
    border-color: rgba(212, 168, 67, 0.4);
    background: rgba(212, 168, 67, 0.06);
  }

  .stat-icon {
    font-size: 1.6rem;
    margin-bottom: 0.5rem;
    display: block;
  }

  .stat-card h3 {
    font-family: 'Cinzel', serif;
    font-size: 0.85rem;
    font-weight: 600;
    color: #d4a843;
    margin-bottom: 0.3rem;
    text-transform: uppercase;
    letter-spacing: 0.06em;
  }

  .stat-card p {
    font-size: 0.88rem;
    color: #aab8d8;
    line-height: 1.5;
  }

  .stat-card small {
    font-size: 0.78rem;
    color: #7a8db5;
  }

  /* ── GAMES ── */
  .games-section {
    background: linear-gradient(180deg, rgba(13, 19, 51, 0.5) 0%, rgba(7, 9, 26, 0.8) 100%);
  }

  /* ── HOW TO GIVE ── */
  .give-section {
    background: rgba(13, 19, 51, 0.4);
  }

  .steps {
    display: flex;
    align-items: flex-start;
    justify-content: center;
    gap: 0;
    margin-bottom: 3.5rem;
    flex-wrap: wrap;
  }

  .step {
    flex: 1;
    min-width: 200px;
    max-width: 280px;
    text-align: center;
    padding: 0 1rem;
  }

  .step-num {
    width: 48px;
    height: 48px;
    background: linear-gradient(135deg, #d4a843, #a07820);
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: 'Cinzel', serif;
    font-size: 1.1rem;
    font-weight: 700;
    color: #0d1333;
    margin: 0 auto 1rem;
  }

  .step h3 {
    font-size: 1rem;
    font-weight: 700;
    color: #fff;
    margin-bottom: 0.5rem;
  }

  .step p {
    font-size: 0.87rem;
    color: #7a8db5;
    line-height: 1.6;
  }

  .step strong {
    color: #d4c080;
  }

  .step-arrow {
    font-size: 1.5rem;
    color: rgba(212, 168, 67, 0.4);
    padding-top: 1.5rem;
    align-self: flex-start;
  }

  @media (max-width: 640px) {
    .step-arrow { display: none; }
  }

  .give-cta {
    text-align: center;
  }

  .prayer-cta {
    margin-top: 1.5rem;
    color: #7a8db5;
    font-size: 0.9rem;
    line-height: 1.7;
  }

  .prayer-cta strong {
    color: #d4a843;
  }

  /* ── FOOTER ── */
  footer {
    background: rgba(4, 6, 18, 0.95);
    border-top: 1px solid rgba(212, 168, 67, 0.1);
    padding: 3rem 1.5rem;
  }

  .footer-inner {
    max-width: 500px;
    margin: 0 auto;
    text-align: center;
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    align-items: center;
  }

  .footer-logo {
    font-family: 'Cinzel', serif;
    font-size: 1.1rem;
    font-weight: 700;
    color: #fff;
  }

  .footer-cross {
    color: #d4a843;
  }

  .footer-name {
    color: #7a8db5;
    font-size: 0.85rem;
  }

  .footer-verse {
    color: #4a5a80;
    font-size: 0.8rem;
    font-style: italic;
    margin-top: 0.25rem;
    margin-bottom: 0.75rem;
  }

  .footer-donate {
    padding: 0.6rem 1.75rem;
    background: linear-gradient(135deg, #dc143c, #a00028);
    color: #fff;
    border-radius: 8px;
    font-size: 0.85rem;
    font-weight: 600;
    transition: opacity 0.15s;
  }

  .footer-donate:hover {
    opacity: 0.85;
  }

  /* ── MOBILE NAV ── */
  @media (max-width: 520px) {
    .nav-link { display: none; }
    .nav-title { font-size: 0.85rem; }
  }
</style>
