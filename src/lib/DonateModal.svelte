<script>
  import { createEventDispatcher } from 'svelte'
  const dispatch = createEventDispatcher()

  const DONATION_URL = 'https://app.managedmissions.com/Donations/Donate/9526'
  const MISSIONARY_NAME = 'Matthew Kitchens'

  function close() {
    dispatch('close')
  }

  function handleKey(e) {
    if (e.key === 'Escape') close()
  }
</script>

<svelte:window on:keydown={handleKey} />

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
<div class="overlay" on:click|self={close}>
  <div class="modal" role="dialog" aria-modal="true" aria-label="Donation instructions">
    <button class="close-btn" on:click={close} aria-label="Close">&#x2715;</button>

    <div class="modal-header">
      <span class="cross">✝</span>
      <h2>How to Donate</h2>
      <p class="subtitle">Every gift — big or small — sends hope to Poland</p>
    </div>

    <div class="steps">

      <div class="step">
        <div class="step-num">1</div>
        <div class="step-body">
          <h3>Open the Giving Portal</h3>
          <p>All donations go through our secure giving portal — please don't use any other method.</p>
          <a href={DONATION_URL} target="_blank" rel="noopener noreferrer" class="portal-link">
            Open Giving Portal &rarr;
          </a>
        </div>
      </div>

      <div class="step important">
        <div class="step-num">2</div>
        <div class="step-body">
          <h3>Select <em>{MISSIONARY_NAME}</em> From the Drop-Down</h3>
          <div class="callout">
            <span class="callout-icon">⚠️</span>
            <p>
              <strong>This step is critical.</strong> On the giving page, you'll see a drop-down menu of missionaries.
              You <strong>must select "{MISSIONARY_NAME}"</strong> before submitting your gift — otherwise
              the donation won't be credited to my trip.
            </p>
          </div>
        </div>
      </div>

      <div class="step">
        <div class="step-num">3</div>
        <div class="step-body">
          <h3>Choose Your Amount &amp; Submit</h3>
          <p>Enter any amount that's meaningful to you and complete the secure checkout. You'll receive a tax-deductible receipt by email.</p>
        </div>
      </div>

    </div>

    <a href={DONATION_URL} target="_blank" rel="noopener noreferrer" class="btn-donate-bottom">
      Open Giving Portal &rarr;
    </a>

    <div class="prayer-note">
      <p>
        Can't give financially right now? <strong>Please pray for our team.</strong><br/>
        Your prayers are just as powerful as any dollar amount.
      </p>
    </div>

    <button class="btn-close-bottom" on:click={close}>Close</button>
  </div>
</div>

<style>
  .overlay {
    position: fixed;
    inset: 0;
    background: rgba(4, 8, 28, 0.88);
    backdrop-filter: blur(6px);
    z-index: 1000;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 1rem;
    animation: fadeIn 0.2s ease;
  }

  @keyframes fadeIn {
    from { opacity: 0; }
    to   { opacity: 1; }
  }

  .modal {
    background: linear-gradient(160deg, #0d1a40 0%, #091228 100%);
    border: 1px solid rgba(212, 168, 67, 0.3);
    border-radius: 20px;
    padding: 2.5rem 2rem 2rem;
    max-width: 540px;
    width: 100%;
    max-height: 90vh;
    overflow-y: auto;
    position: relative;
    animation: slideUp 0.25s ease;
    box-shadow: 0 0 80px rgba(212, 168, 67, 0.08), 0 20px 60px rgba(0, 0, 0, 0.6);
  }

  @keyframes slideUp {
    from { transform: translateY(24px); opacity: 0; }
    to   { transform: translateY(0); opacity: 1; }
  }

  .close-btn {
    position: absolute;
    top: 1rem;
    right: 1rem;
    background: rgba(255, 255, 255, 0.07);
    color: #aab8d8;
    border-radius: 50%;
    width: 32px;
    height: 32px;
    font-size: 1rem;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: background 0.15s, color 0.15s;
  }

  .close-btn:hover {
    background: rgba(220, 20, 60, 0.3);
    color: #fff;
  }

  .modal-header {
    text-align: center;
    margin-bottom: 2rem;
  }

  .cross {
    font-size: 1.5rem;
    color: #d4a843;
    display: block;
    margin-bottom: 0.5rem;
  }

  .modal-header h2 {
    font-family: 'Cinzel', serif;
    font-size: 1.75rem;
    font-weight: 700;
    color: #fff;
    margin-bottom: 0.4rem;
  }

  .subtitle {
    color: #7a8db5;
    font-size: 0.9rem;
  }

  .steps {
    display: flex;
    flex-direction: column;
    gap: 1.25rem;
    margin-bottom: 1.5rem;
  }

  .step {
    display: flex;
    gap: 1rem;
    background: rgba(255, 255, 255, 0.04);
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-radius: 12px;
    padding: 1.1rem 1.25rem;
  }

  .step.important {
    border-color: rgba(220, 20, 60, 0.45);
    background: rgba(220, 20, 60, 0.06);
  }

  .step-num {
    width: 32px;
    height: 32px;
    min-width: 32px;
    background: linear-gradient(135deg, #d4a843, #a07820);
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: 700;
    font-size: 0.9rem;
    color: #0d1333;
    margin-top: 2px;
  }

  .step.important .step-num {
    background: linear-gradient(135deg, #dc143c, #a00028);
    color: #fff;
  }

  .step-body h3 {
    font-size: 1rem;
    font-weight: 600;
    color: #fff;
    margin-bottom: 0.3rem;
  }

  .step-body h3 em {
    font-style: normal;
    color: #d4c080;
  }

  .step-body p {
    font-size: 0.88rem;
    color: #aab8d8;
    line-height: 1.5;
  }

  .step-body strong {
    color: #d4c080;
  }

  .step.important .step-body strong {
    color: #ffb8c4;
  }

  .callout {
    display: flex;
    gap: 0.6rem;
    margin-top: 0.6rem;
    padding: 0.75rem 0.85rem;
    background: rgba(220, 20, 60, 0.12);
    border: 1px solid rgba(220, 20, 60, 0.35);
    border-radius: 8px;
  }

  .callout-icon {
    font-size: 1.1rem;
    line-height: 1.4;
  }

  .callout p {
    margin: 0;
    color: #f0d6dd;
    font-size: 0.86rem;
    line-height: 1.55;
  }

  .portal-link {
    display: inline-block;
    margin-top: 0.6rem;
    padding: 0.4rem 1rem;
    background: linear-gradient(135deg, #dc143c, #a00028);
    color: #fff;
    border-radius: 6px;
    font-size: 0.85rem;
    font-weight: 600;
    text-decoration: none;
    transition: opacity 0.15s;
  }

  .portal-link:hover {
    opacity: 0.85;
    text-decoration: none;
  }

  .btn-donate-bottom {
    display: block;
    width: 100%;
    text-align: center;
    padding: 0.9rem;
    background: linear-gradient(135deg, #dc143c, #a00028);
    color: #fff;
    border-radius: 10px;
    font-weight: 700;
    font-size: 0.95rem;
    text-decoration: none;
    transition: opacity 0.15s, transform 0.15s;
    margin-bottom: 1.25rem;
    box-shadow: 0 6px 20px rgba(220, 20, 60, 0.25);
  }

  .btn-donate-bottom:hover {
    opacity: 0.92;
    transform: translateY(-1px);
    text-decoration: none;
  }

  .prayer-note {
    text-align: center;
    padding: 1rem 0 0;
    border-top: 1px solid rgba(255, 255, 255, 0.08);
    margin-bottom: 1.25rem;
    font-size: 0.88rem;
    color: #8899bb;
    line-height: 1.6;
  }

  .prayer-note strong {
    color: #d4a843;
  }

  .btn-close-bottom {
    width: 100%;
    padding: 0.75rem;
    background: rgba(255, 255, 255, 0.06);
    color: #aab8d8;
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 10px;
    font-weight: 600;
    font-size: 0.9rem;
    transition: background 0.15s;
  }

  .btn-close-bottom:hover {
    background: rgba(255, 255, 255, 0.1);
  }
</style>
