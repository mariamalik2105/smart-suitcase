<script>
  let showInfo = false;
</script>

<main class="page">
  <section class="device-section">
    <h2>Device UI</h2>

    <div class="suitcase">
      <div class="handle"></div>

      <div class="weight-display">
        Weight: 21.4 kg / 23 kg
      </div>

      <div class="main-display">
        <h3>SmartCase</h3>

        <p class="status">Trip Ready</p>

        <div class="info-row">
          <span>Weight</span>
          <strong>21.4 kg</strong>
        </div>

        <div class="info-row">
          <span>Lock</span>
          <strong>Locked</strong>
        </div>

        <div class="info-row">
          <span>Battery</span>
          <strong>82%</strong>
        </div>

        <hr />

        <h4>Packing Items</h4>

        <p>✓ Passport</p>
        <p>✓ Charger</p>
        <p>✓ Toiletries</p>
        <p class="warning">⚠ Headphones missing</p>

        <p class="bluetooth">Bluetooth Connected</p>
      </div>

      <div class="side-panel">
        <button>🔒</button>
        <button>✈</button>
        <button>⌖</button>
        <div class="usb">USB-C</div>
      </div>
    </div>
  </section>

  <section class="testing-section">
    <h1>SmartCase</h1>
    <p class="subtitle">Smart Suitcase Interface</p>

    <p><strong>Maria Malik</strong></p>

    <a href="#" class="writeup-link">
      Project Write-Up
    </a>

    <h3>About the Project</h3>

    <p>
      SmartCase is a smart suitcase designed to make travel easier
      through weight monitoring, packing reminders, security controls,
      tracking, and smart movement features.
    </p>

    <h3>Testing Controls</h3>

    <div class="testing-buttons">
      <button>Add Weight</button>
      <button>Remove Weight</button>

      <button>Remove Item</button>
      <button>Add Item</button>

      <button>Lock</button>
      <button>Unlock</button>

      <button>Connect Phone</button>
      <button>Disconnect Phone</button>
    </div>

    <button
      class="info-button"
      onclick={() => showInfo = !showInfo}
    >
      ℹ Info
    </button>

    {#if showInfo}
      <div class="info-box">
        <strong>How to test SmartCase</strong>

        <p>
          Use the buttons above to simulate changes to the suitcase.
          The Device UI will respond to actions such as changing its
          weight, removing packed items, locking the suitcase, and
          connecting a phone.
        </p>
      </div>
    {/if}
  </section>
</main>