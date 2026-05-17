<script>
  import { createEventDispatcher } from 'svelte'
  const dispatch = createEventDispatcher()

  // UPDATE THIS with your actual donation link/platform
  const DONATION_URL = 'YOUR_DONATION_LINK_HERE'
  const CHURCH_NAME  = 'YOUR_CHURCH_NAME'
  const PAYPAL_EMAIL = 'YOUR_PAYPAL_OR_VENMO'

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

    <div class="options">

      <!-- Option 1: Online link -->
      <div class="option">
        <div class="option-num">1</div>
        <div class="option-body">
          <h3>Give Online</h3>
          <p>Visit the official giving portal and designate your gift.</p>
          {#if DONATION_URL !== 'YOUR_DONATION_LINK_HERE'}
            <a href={DONATION_URL} target="_blank" rel="noopener noreferrer" class="option-link">
              Open Giving Portal &rarr;
            </a>
          {:else}
            <span class="placeholder-note">[ Donation link coming soon — check back! ]</span>
          {/if}
          <p class="memo-note">In the memo or designation field, write:<br />
            <strong>"Poland Mission — Matthew Kitchens"</strong>
          </p>
        </div>
      </div>

      <!-- Option 2: Check -->
      <div class="option">
        <div class="option-num">2</div>
        <div class="option-body">
          <h3>Write a Check</h3>
          <p>
            Make checks payable to <strong>{CHURCH_NAME !== 'YOUR_CHURCH_NAME' ? CHURCH_NAME : '[ Your Church Name ]'}</strong>
            and write <strong>"Poland Mission — Matthew Kitchens"</strong> in the memo line.
          </p>
          <p class="memo-note">Checks can be mailed to the church office or handed to Matthew directly.</p>
        </div>
      </div>

      <!-- Option 3: Digital payment -->
      <div class="option">
        <div class="option-num">3</div>
        <div class="option-body">
          <h3>PayPal / Venmo / Cash App</h3>
          <p>
            Send directly to:
            <strong>{PAYPAL_EMAIL !== 'YOUR_PAYPAL_OR_VENMO' ? PAYPAL_EMAIL : '[ Contact Matthew for details ]'}</strong>
          </p>
          <p class="memo-note">Include a note: <strong>"Poland Mission"</strong></p>
        </div>
      </div>

    </div>

    <div class="prayer-note">
      <p>
        Can't give financially right now? <strong>Please pray for our team.</strong><br/>
        Your prayers are just as powerful as any dollar amount.
      </p>
    </div>

    <button class="btn-close-bottom" on:click={close}>Got it, thanks!</button>
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

  .options {
    display: flex;
    flex-direction: column;
    gap: 1.25rem;
    margin-bottom: 1.5rem;
  }

  .option {
    display: flex;
    gap: 1rem;
    background: rgba(255, 255, 255, 0.04);
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-radius: 12px;
    padding: 1.1rem 1.25rem;
  }

  .option-num {
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

  .option-body h3 {
    font-size: 1rem;
    font-weight: 600;
    color: #fff;
    margin-bottom: 0.3rem;
  }

  .option-body p {
    font-size: 0.88rem;
    color: #aab8d8;
    line-height: 1.5;
  }

  .option-body strong {
    color: #d4c080;
  }

  .option-link {
    display: inline-block;
    margin: 0.5rem 0;
    padding: 0.4rem 1rem;
    background: linear-gradient(135deg, #dc143c, #a00028);
    color: #fff;
    border-radius: 6px;
    font-size: 0.85rem;
    font-weight: 600;
    text-decoration: none;
    transition: opacity 0.15s;
  }

  .option-link:hover {
    opacity: 0.85;
    text-decoration: none;
  }

  .placeholder-note {
    display: inline-block;
    margin: 0.5rem 0;
    font-size: 0.82rem;
    color: #7a8db5;
    font-style: italic;
  }

  .memo-note {
    margin-top: 0.5rem;
    font-size: 0.83rem !important;
    color: #8899bb !important;
  }

  .prayer-note {
    text-align: center;
    padding: 1rem;
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
    padding: 0.85rem;
    background: linear-gradient(135deg, #d4a843, #a07820);
    color: #0d1333;
    border-radius: 10px;
    font-weight: 700;
    font-size: 0.95rem;
    transition: opacity 0.15s, transform 0.15s;
  }

  .btn-close-bottom:hover {
    opacity: 0.9;
    transform: translateY(-1px);
  }
</style>
