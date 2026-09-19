<script>
  let showInfo = $state(false);

  let weight = $state(21.4);
  let maxWeight = 23;

  let items = $state([
    { name: "Passport", packed: true },
    { name: "Charger", packed: true },
    { name: "Toiletries", packed: true },
    { name: "Headphones", packed: true }
  ]);

  let packedCount = $derived(
    items.filter((item) => item.packed).length
  );

  let hasMissingItems = $derived(
    packedCount < items.length
  );

  function addWeight() {
    if (weight < 40) {
      weight += 1;
    }
  }

  function removeWeight() {
    if (weight >= 1) {
      weight -= 1;
    }
  }

  function removeItem() {
    for (let i = items.length - 1; i >= 0; i--) {
      if (items[i].packed) {
        items[i].packed = false;
        return;
      }
    }
  }

  function addItem() {
    for (let i = 0; i < items.length; i++) {
      if (!items[i].packed) {
        items[i].packed = true;
        return;
      }
    }
  }
</script>

<main class="page">

  <!-- DEVICE UI -->
  <section class="device-section">

    <h2>Device UI</h2>

    <div class="suitcase">

      <div class="handle"></div>

      <!-- TOP WEIGHT DISPLAY -->
      <div
        class="weight-display"
        class:overweight-display={weight > maxWeight}
      >
        Weight: {weight.toFixed(1)} kg / {maxWeight} kg
      </div>


      <!-- MAIN DISPLAY -->
      <div class="main-display">

        <h3>SmartCase</h3>

        {#if weight > maxWeight}
          <p class="status overweight-status">
            ⚠ Overweight
          </p>

        {:else if hasMissingItems}
          <p class="status packing-status">
            ⚠ Packing Incomplete
          </p>

        {:else}
          <p class="status">
            ✓ Trip Ready
          </p>
        {/if}


        <!-- WEIGHT -->
        <div class="info-row">
          <span>Weight</span>
          <strong>{weight.toFixed(1)} kg</strong>
        </div>


        {#if weight > maxWeight}

          <div class="weight-warning">
            ⚠ {(weight - maxWeight).toFixed(1)} kg over the limit
          </div>

        {:else}

          <div class="weight-safe">
            ✓ {(maxWeight - weight).toFixed(1)} kg remaining
          </div>

        {/if}


        <!-- LOCK -->
        <div class="info-row">
          <span>Lock</span>
          <strong>Locked</strong>
        </div>


        <!-- BATTERY -->
        <div class="info-row">
          <span>Battery</span>
          <strong>82%</strong>
        </div>


        <hr />


        <!-- PACKING -->
        <h4>Packing Items</h4>

        <p class="packing-count">
          {packedCount} / {items.length} essentials packed
        </p>


        <div class="packing-list">

          {#each items as item}

            {#if item.packed}

              <p class="packed-item">
                ✓ {item.name}
              </p>

            {:else}

              <p class="missing-item">
                ⚠ {item.name} missing
              </p>

            {/if}

          {/each}

        </div>


        <!-- BLUETOOTH -->
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


      <!-- SIDE PANEL -->
      <div class="side-panel">

        <button title="Lock">
          🔒
        </button>

        <button title="Travel Mode">
          ✈
        </button>

        <button
          class="find-button"
          title="Find My Bag"
        >
          ⌖
        </button>

        <div class="usb">
          USB-C
        </div>

      </div>

    </div>

  </section>


  <!-- TESTING UI -->
  <section class="testing-section">

    <h1>SmartCase</h1>

    <p class="subtitle">
      Smart Suitcase Interface
    </p>

    <p>
      <strong>Maria Malik</strong>
    </p>

    <a
      href="#"
      class="writeup-link"
    >
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

      <button onclick={addWeight}>
        Add 1 kg
      </button>

      <button onclick={removeWeight}>
        Remove 1 kg
      </button>


      <button
        onclick={removeItem}
        disabled={packedCount === 0}
      >
        Remove Item
      </button>

      <button
        onclick={addItem}
        disabled={packedCount === items.length}
      >
        Add Item
      </button>


      <button>
        Lock
      </button>

      <button>
        Unlock
      </button>


      <button>
        Connect Phone
      </button>

      <button>
        Disconnect Phone
      </button>

    </div>


    <button
      class="info-button"
      onclick={() => showInfo = !showInfo}
    >
      ℹ Info
    </button>


    {#if showInfo}

      <div class="info-box">

        <strong>
          How to test SmartCase
        </strong>

        <p>
          Use the controls above to simulate weight changes,
          adding or removing packed items, security controls,
          and phone connectivity.
        </p>

      </div>

    {/if}

  </section>

</main>