<script>
  let showInfo = false;

  let weight = 21.4;
  let maxWeight = 23;

  function addWeight() {
    weight = Math.min(weight + 1, 40);
  }

  function removeWeight() {
    weight = Math.max(weight - 1, 0);
  }
</script>

<main class="page">
  <section class="device-section">
    <h2>Device UI</h2>

    <div class="suitcase">
      <div class="handle"></div>

      <div
        class="weight-display"
        class:overweight-display={weight > maxWeight}
      >
        Weight: {weight.toFixed(1)} kg / {maxWeight} kg
      </div>

      <div class="main-display">
        <h3>SmartCase</h3>

        {#if weight > maxWeight}
          <p class="status overweight-status">⚠ Overweight</p>
        {:else}
          <p class="status">✓ Trip Ready</p>
        {/if}

        <div class="info-row">
          <span>Weight</span>
          <strong>{weight.toFixed(1)} kg</strong>
        </div>

        {#if weight > maxWeight}
          <div class="weight-warning">
            ⚠ Bag is {(weight - maxWeight).toFixed(1)} kg over the limit
          </div>
        {:else}
          <div class="weight-safe">
            ✓ {(maxWeight - weight).toFixed(1)} kg remaining
          </div>
        {/if}

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

        <p class="bluetooth">
          <svg
            class="bluetooth-icon"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
          >
            <path d="M7 7l10 10-5 4V3l5 4L7 17" />
          </svg>

          Bluetooth Connected
        </p>
      </div>

      <div class="side-panel">
        <button>🔒</button>
        <button>✈</button>
        <button class="find-button">⌖</button>
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
      <button onclick={addWeight}>Add 1 kg</button>
      <button onclick={removeWeight}>Remove 1 kg</button>

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