<script>
  import { onDestroy } from "svelte";

  const maxWeight = 23;

  const itemNames = [
    "Passport",
    "Charger",
    "Toiletries",
    "Headphones"
  ];

  const writeupUrl = "/SmartCase_Writeup.docx";
  const demoUrl = "/SmartCase_Demo.webm";

  let showInfo = $state(false);

  // Main suitcase state
  let weight = $state(21.4);

  let items = $state([
    { name: "Passport", packed: true },
    { name: "Charger", packed: true },
    { name: "Toiletries", packed: true },
    { name: "Headphones", packed: true }
  ]);

  let isLocked = $state(true);
  let isOpen = $state(false);

  let battery = $state(82);

  let bluetoothConnected = $state(true);

  let travelMode = $state(false);
  let followMode = $state(false);

  let isFinding = $state(false);

  let location = $state("With you");
  let bagDistance = $state(1.2);

  // Phone state
  let phoneBattery = $state(76);
  let usbCharging = $state(false);

  // Feedback
  let notice = $state("SmartCase is ready for testing.");

  // Scenarios
  let selectedScenario = $state("ready");

  // Simulation
  let simulationRunning = $state(false);
  let simulationStep = $state(0);
  let simulationText = $state("Simulation not running.");

  let simulationTimer;
  let findTimer;
  let usbTimer;

  let packedCount = $derived(
    items.filter((item) => item.packed).length
  );

  let hasMissingItems = $derived(
    packedCount < items.length
  );

  let isOverweight = $derived(
    weight > maxWeight
  );

  let tripReady = $derived(
    !isOverweight &&
    !hasMissingItems &&
    isLocked &&
    !isOpen
  );


  /* ---------------------------
     BASIC WEIGHT CONTROLS
  ---------------------------- */

  function addWeight() {
    weight = Math.min(weight + 1, 40);
    notice = `Weight increased to ${weight.toFixed(1)} kg.`;
  }

  function removeWeight() {
    weight = Math.max(weight - 1, 0);
    notice = `Weight reduced to ${weight.toFixed(1)} kg.`;
  }


  /* ---------------------------
     PACKING CONTROLS
  ---------------------------- */

  function removeItem() {
    for (let i = items.length - 1; i >= 0; i--) {
      if (items[i].packed) {
        items[i].packed = false;
        notice = `${items[i].name} was removed from SmartCase.`;
        return;
      }
    }
  }

  function addItem() {
    for (let i = 0; i < items.length; i++) {
      if (!items[i].packed) {
        items[i].packed = true;
        notice = `${items[i].name} was added to SmartCase.`;
        return;
      }
    }
  }

  function toggleItem(index) {
    items[index].packed = !items[index].packed;

    notice = items[index].packed
      ? `${items[index].name} packed.`
      : `${items[index].name} removed.`;
  }


  /* ---------------------------
     LOCK / SECURITY
  ---------------------------- */

  function lockSuitcase() {
    isLocked = true;
    notice = "SmartCase locked.";
  }

  function unlockSuitcase() {
    isLocked = false;
    notice = "SmartCase unlocked.";
  }

  function toggleLock() {
    isLocked = !isLocked;

    notice = isLocked
      ? "SmartCase locked."
      : "SmartCase unlocked.";
  }


  /* ---------------------------
     OPEN / CLOSE SENSOR
  ---------------------------- */

  function openSuitcase() {
    isOpen = true;

    if (travelMode) {
      notice = "Warning: SmartCase opened while Travel Mode is active.";
    } else {
      notice = "SmartCase opened.";
    }
  }

  function closeSuitcase() {
    isOpen = false;
    notice = "SmartCase closed.";
  }


  /* ---------------------------
     BLUETOOTH
  ---------------------------- */

  function connectPhone() {
    bluetoothConnected = true;
    location = "With you";
    bagDistance = 1.2;

    notice = "Phone connected to SmartCase through Bluetooth.";
  }

  function disconnectPhone() {
    bluetoothConnected = false;
    followMode = false;

    notice = "Phone disconnected. Follow Mode has been disabled.";
  }


  /* ---------------------------
     TRAVEL MODE
  ---------------------------- */

  function toggleTravelMode() {
    travelMode = !travelMode;

    if (travelMode) {
      isLocked = true;
      followMode = false;

      notice =
        "Travel Mode enabled. SmartCase has been automatically locked.";
    } else {
      notice = "Travel Mode disabled.";
    }
  }


  /* ---------------------------
     FOLLOW MODE
  ---------------------------- */

  function toggleFollowMode() {
    if (!bluetoothConnected) {
      followMode = false;
      notice = "Connect your phone before using Follow Mode.";
      return;
    }

    if (battery <= 5) {
      followMode = false;
      notice = "Battery is too low to use Follow Mode.";
      return;
    }

    if (isOpen) {
      followMode = false;
      notice = "Close SmartCase before activating Follow Mode.";
      return;
    }

    followMode = !followMode;

    notice = followMode
      ? "Follow Mode activated. SmartCase is following your phone."
      : "Follow Mode paused.";
  }


  /* ---------------------------
     FIND MY
  ---------------------------- */

  function findBag() {
    isFinding = true;

    clearTimeout(findTimer);

    if (bagDistance <= 20) {
      notice = "SmartCase is ringing and flashing so you can find it.";
    } else {
      notice =
        `SmartCase located ${bagDistance.toFixed(0)} m away at ${location}.`;
    }

    findTimer = setTimeout(() => {
      isFinding = false;
    }, 4000);
  }


  /* ---------------------------
     BATTERY
  ---------------------------- */

  function drainBattery() {
    battery = Math.max(battery - 10, 0);

    if (battery === 0) {
      followMode = false;
    }

    notice = `SmartCase battery: ${battery}%.`;
  }

  function chargeSuitcase() {
    battery = Math.min(battery + 10, 100);
    notice = `SmartCase battery charged to ${battery}%.`;
  }


  /* ---------------------------
     USB-C CHARGING
  ---------------------------- */

  function chargePhone() {
    if (battery < 5) {
      notice = "SmartCase battery is too low to charge the phone.";
      return;
    }

    if (phoneBattery >= 100) {
      notice = "Phone battery is already full.";
      return;
    }

    battery = Math.max(battery - 5, 0);
    phoneBattery = Math.min(phoneBattery + 10, 100);

    usbCharging = true;

    notice =
      `USB-C charging: phone ${phoneBattery}%, SmartCase ${battery}%.`;

    clearTimeout(usbTimer);

    usbTimer = setTimeout(() => {
      usbCharging = false;
    }, 1500);
  }


  /* ---------------------------
     LOCATION TESTING
  ---------------------------- */

  function moveBagFarther() {
    bagDistance = Math.min(bagDistance + 10, 200);

    location =
      bagDistance > 30
        ? "Airport terminal"
        : "Nearby";

    notice = `SmartCase is now ${bagDistance.toFixed(0)} m away.`;
  }

  function moveBagCloser() {
    bagDistance = Math.max(bagDistance - 10, 0.5);

    if (bagDistance <= 5) {
      location = "With you";
    }

    notice = `SmartCase is now ${bagDistance.toFixed(1)} m away.`;
  }


  /* ---------------------------
     FOUR TEST SCENARIOS
  ---------------------------- */

  const scenarios = {
    ready: {
      weight: 21.4,
      packed: [true, true, true, true],
      locked: true,
      open: false,
      battery: 82,
      bluetooth: true,
      travel: false,
      follow: false,
      location: "With you",
      distance: 1.2
    },

    overweight: {
      weight: 25.2,
      packed: [true, true, true, true],
      locked: true,
      open: false,
      battery: 68,
      bluetooth: true,
      travel: false,
      follow: false,
      location: "With you",
      distance: 1.3
    },

    missing: {
      weight: 18.7,
      packed: [true, false, true, false],
      locked: false,
      open: false,
      battery: 54,
      bluetooth: true,
      travel: false,
      follow: false,
      location: "Hotel room",
      distance: 2.4
    },

    separated: {
      weight: 20.8,
      packed: [true, true, true, true],
      locked: true,
      open: false,
      battery: 44,
      bluetooth: false,
      travel: true,
      follow: false,
      location: "Baggage Claim C",
      distance: 65
    }
  };

  function loadScenario(name) {
    const scenario = scenarios[name];

    selectedScenario = name;

    weight = scenario.weight;

    items = itemNames.map((name, index) => ({
      name,
      packed: scenario.packed[index]
    }));

    isLocked = scenario.locked;
    isOpen = scenario.open;

    battery = scenario.battery;

    bluetoothConnected = scenario.bluetooth;

    travelMode = scenario.travel;
    followMode = scenario.follow;

    location = scenario.location;
    bagDistance = scenario.distance;

    isFinding = false;

    notice = `Loaded "${scenarioLabel(name)}" scenario.`;
  }

  function scenarioLabel(name) {
    const labels = {
      ready: "Ready to Fly",
      overweight: "Overweight Bag",
      missing: "Missing Essentials",
      separated: "Separated Bag"
    };

    return labels[name];
  }


  /* ---------------------------
     AIRPORT WALK SIMULATION
  ---------------------------- */

  const simulationSteps = [
    {
      location: "Terminal Entrance",
      distance: 1.0,
      text: "Follow Mode started."
    },

    {
      location: "Check-in Area",
      distance: 1.3,
      text: "SmartCase is following behind the traveler."
    },

    {
      location: "Security",
      distance: 1.8,
      text: "Adjusting speed to remain close."
    },

    {
      location: "Concourse A",
      distance: 1.1,
      text: "Continuing through the airport."
    },

    {
      location: "Gate A12",
      distance: 0.8,
      text: "Traveler has reached the gate."
    }
  ];

  function startSimulation() {
    if (simulationRunning) return;

    if (!bluetoothConnected) {
      connectPhone();
    }

    if (isOpen) {
      closeSuitcase();
    }

    travelMode = false;
    followMode = true;

    simulationRunning = true;
    simulationStep = 0;

    applySimulationStep(0);

    notice = "Airport Follow Mode simulation started.";

    clearInterval(simulationTimer);

    simulationTimer = setInterval(() => {
      simulationStep += 1;

      if (simulationStep >= simulationSteps.length) {
        finishSimulation();
        return;
      }

      applySimulationStep(simulationStep);

      battery = Math.max(battery - 1, 0);
    }, 1400);
  }

  function applySimulationStep(step) {
    const data = simulationSteps[step];

    location = data.location;
    bagDistance = data.distance;
    simulationText = data.text;
  }

  function finishSimulation() {
    clearInterval(simulationTimer);

    simulationStep = simulationSteps.length - 1;

    simulationRunning = false;
    followMode = false;

    simulationText = "Simulation complete — traveler reached Gate A12.";

    notice = "Airport simulation complete.";
  }

  function stopSimulation() {
    clearInterval(simulationTimer);

    simulationRunning = false;
    followMode = false;

    simulationText = "Simulation stopped.";

    notice = "Airport simulation stopped.";
  }


  onDestroy(() => {
    clearInterval(simulationTimer);
    clearTimeout(findTimer);
    clearTimeout(usbTimer);
  });
</script>


<main class="page">

  <!-- =====================================================
       DEVICE UI
  ====================================================== -->

  <section class="device-section">

    <div class="section-heading">
      <p class="eyebrow">DEVICE UI</p>
      <h2>SmartCase</h2>
    </div>


    <div class="suitcase">

      <div class="handle"></div>


      <!-- TOP WEIGHT DISPLAY -->

      <div
        class="weight-display"
        class:overweight-display={isOverweight}
      >
        <span>WEIGHT</span>

        <strong>
          {weight.toFixed(1)} kg / {maxWeight} kg
        </strong>
      </div>


      <!-- MAIN FRONT DISPLAY -->

      <div
        class="main-display"
        class:finding-screen={isFinding}
      >

        <div class="screen-header">

          <div>
            <p class="screen-brand">SmartCase</p>

            {#if travelMode}
              <span class="mode-badge">
                TRAVEL MODE
              </span>
            {/if}
          </div>

          <div class="battery-mini">
            {battery}%
          </div>

        </div>


        <!-- MAIN STATUS -->

        {#if isOpen && travelMode}

          <p class="status danger-status">
            ⚠ Bag Open in Travel Mode
          </p>

        {:else if isOverweight}

          <p class="status danger-status">
            ⚠ Overweight
          </p>

        {:else if hasMissingItems}

          <p class="status warning-status">
            ⚠ Packing Incomplete
          </p>

        {:else if !isLocked}

          <p class="status warning-status">
            ⚠ Suitcase Unlocked
          </p>

        {:else}

          <p class="status ready-status">
            ✓ Trip Ready
          </p>

        {/if}


        <!-- WEIGHT -->

        <div class="data-row">
          <span>Weight</span>

          <strong>
            {weight.toFixed(1)} kg
          </strong>
        </div>

        <div class="weight-bar">

          <div
            class="weight-bar-fill"
            class:bar-danger={isOverweight}
            style={`width: ${Math.min((weight / maxWeight) * 100, 100)}%`}
          ></div>

        </div>

        {#if isOverweight}

          <p class="small-alert danger-text">
            {(weight - maxWeight).toFixed(1)} kg over the limit
          </p>

        {:else}

          <p class="small-alert safe-text">
            {(maxWeight - weight).toFixed(1)} kg remaining
          </p>

        {/if}


        <!-- SECURITY / SENSOR STATUS -->

        <div class="status-grid">

          <div class="status-card">
            <span>Lock</span>

            <strong class:good-text={isLocked} class:danger-text={!isLocked}>
              {isLocked ? "Locked" : "Unlocked"}
            </strong>
          </div>


          <div class="status-card">
            <span>Suitcase</span>

            <strong class:good-text={!isOpen} class:danger-text={isOpen}>
              {isOpen ? "Open" : "Closed"}
            </strong>
          </div>


          <div class="status-card">
            <span>Battery</span>

            <strong>
              {battery}%
            </strong>
          </div>


          <div class="status-card">
            <span>Follow</span>

            <strong class:good-text={followMode}>
              {followMode ? "Active" : "Off"}
            </strong>
          </div>

        </div>


        <!-- PACKING -->

        <div class="divider"></div>

        <div class="packing-heading">

          <h4>Packing</h4>

          <span>
            {packedCount}/{items.length}
          </span>

        </div>


        <div class="packing-list">

          {#each items as item}

            <div
              class="packing-item"
              class:missing-item={!item.packed}
            >

              <span>
                {item.packed ? "✓" : "⚠"}
              </span>

              <span>
                {item.name}
              </span>

            </div>

          {/each}

        </div>


        <!-- CONNECTION -->

        <div class="screen-footer">

          <div
            class="bluetooth-status"
            class:disconnected={!bluetoothConnected}
          >

            <svg
              class="bluetooth-icon"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <path d="M7 7l10 10-5 4V3l5 4L7 17" />
            </svg>

            {bluetoothConnected
              ? "Phone Connected"
              : "Phone Disconnected"}

          </div>


          <div class="location-mini">
            ⌖ {location}
          </div>

        </div>

      </div>


      <!-- PHYSICAL SIDE CONTROLS -->

      <div class="side-panel">

        <button
          class="physical-button"
          title={isLocked ? "Unlock" : "Lock"}
          onclick={toggleLock}
        >
          {isLocked ? "🔒" : "🔓"}
        </button>


        <button
          class="physical-button"
          class:active-control={travelMode}
          title="Travel Mode"
          onclick={toggleTravelMode}
        >
          ✈
        </button>


        <button
          class="physical-button find-button"
          class:find-active={isFinding}
          title="Find My Bag"
          onclick={findBag}
        >
          ⌖
        </button>


        <div
          class="usb-port"
          class:usb-active={usbCharging}
          title="USB-C Port"
        >
          USB-C
        </div>

      </div>


      <!-- WHEELS -->

      <div class="wheel wheel-left"></div>
      <div class="wheel wheel-right"></div>

    </div>

  </section>


  <!-- =====================================================
       PROJECT + TESTING UI
  ====================================================== -->

  <section class="testing-section">

    <header class="project-header">

      <div>

        <p class="eyebrow">
          PROJECT 1 — INTERFACE TO A SMART OBJECT
        </p>

        <h1>
          SmartCase
        </h1>

        <p class="subtitle">
          Smart Suitcase Interface
        </p>

        <p class="student-name">
          Maria Malik
        </p>

      </div>


<div class="project-links">

  <a
    class="writeup-link"
    href={writeupUrl}
    target="_blank"
    rel="noreferrer"
  >
    Project Write-Up ↗
  </a>

  <a
    class="writeup-link"
    href={demoUrl}
    target="_blank"
    rel="noreferrer"
  >
    Demo Video ▶
  </a>

</div>

    </header>


    <p class="project-description">
      SmartCase is a smart suitcase designed to make travel easier
      through automatic weight monitoring, packing reminders,
      security controls, location tracking, phone connectivity,
      and hands-free movement.
    </p>


    <div class="notice">
      <strong>System:</strong>
      {notice}
    </div>


    <div class="right-grid">

      <!-- TESTING CONTROLS -->

      <div class="testing-controls">


        <!-- SCENARIOS -->

        <div class="control-section">

          <div class="control-heading">
            <h3>Test Scenarios</h3>
            <span>Sensor profiles</span>
          </div>

          <div class="scenario-grid">

            <button
              class:scenario-active={selectedScenario === "ready"}
              onclick={() => loadScenario("ready")}
            >
              Ready to Fly
            </button>

            <button
              class:scenario-active={selectedScenario === "overweight"}
              onclick={() => loadScenario("overweight")}
            >
              Overweight Bag
            </button>

            <button
              class:scenario-active={selectedScenario === "missing"}
              onclick={() => loadScenario("missing")}
            >
              Missing Essentials
            </button>

            <button
              class:scenario-active={selectedScenario === "separated"}
              onclick={() => loadScenario("separated")}
            >
              Separated Bag
            </button>

          </div>

        </div>


        <!-- WEIGHT / SENSOR -->

        <div class="control-section">

          <div class="control-heading">
            <h3>Weight & Sensors</h3>
          </div>

          <div class="button-grid">

            <button onclick={addWeight}>
              + Add 1 kg
            </button>

            <button onclick={removeWeight}>
              − Remove 1 kg
            </button>

            <button
              onclick={openSuitcase}
              disabled={isOpen}
            >
              Open Suitcase
            </button>

            <button
              onclick={closeSuitcase}
              disabled={!isOpen}
            >
              Close Suitcase
            </button>

            <button onclick={moveBagFarther}>
              Move Bag Farther
            </button>

            <button onclick={moveBagCloser}>
              Move Bag Closer
            </button>

          </div>

        </div>


        <!-- PACKING -->

        <div class="control-section">

          <div class="control-heading">
            <h3>Packing Items</h3>

            <span>
              Click an item to toggle it
            </span>
          </div>

          <div class="item-buttons">

            {#each items as item, index}

              <button
                class:item-present={item.packed}
                class:item-missing={!item.packed}
                onclick={() => toggleItem(index)}
              >
                {item.packed ? "✓" : "⚠"}
                {item.name}
              </button>

            {/each}

          </div>


          <div class="mini-button-row">

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

          </div>

        </div>


        <!-- SECURITY -->

        <div class="control-section">

          <div class="control-heading">
            <h3>Security & Connectivity</h3>
          </div>

          <div class="button-grid">

            <button
              onclick={lockSuitcase}
              disabled={isLocked}
            >
              Lock
            </button>

            <button
              onclick={unlockSuitcase}
              disabled={!isLocked}
            >
              Unlock
            </button>

            <button
              onclick={connectPhone}
              disabled={bluetoothConnected}
            >
              Connect Phone
            </button>

            <button
              onclick={disconnectPhone}
              disabled={!bluetoothConnected}
            >
              Disconnect Phone
            </button>

            <button
              class:active-button={travelMode}
              onclick={toggleTravelMode}
            >
              {travelMode
                ? "Disable Travel Mode"
                : "Enable Travel Mode"}
            </button>

            <button onclick={findBag}>
              Find My Bag
            </button>

          </div>

        </div>


        <!-- BATTERY / USB -->

        <div class="control-section">

          <div class="control-heading">
            <h3>Battery & USB-C</h3>
          </div>

          <div class="button-grid">

            <button onclick={drainBattery}>
              Drain 10%
            </button>

            <button onclick={chargeSuitcase}>
              Charge 10%
            </button>

            <button onclick={chargePhone}>
              Charge Phone via USB-C
            </button>

          </div>

        </div>


        <!-- TIME SIMULATION -->

        <div class="control-section">

          <div class="control-heading">
            <h3>Airport Follow Simulation</h3>

            <span>
              Time-based simulation
            </span>
          </div>


          <div class="simulation-box">

            <p>
              {simulationText}
            </p>

            <div class="simulation-progress">

              <div
                class="simulation-progress-fill"
                style={`width: ${
                  simulationRunning || simulationStep > 0
                    ? Math.min(
                        ((simulationStep + 1) /
                          simulationSteps.length) *
                          100,
                        100
                      )
                    : 0
                }%`}
              ></div>

            </div>

            <small>
              Step {simulationStep + 1}
              of {simulationSteps.length}
            </small>

          </div>


          <div class="mini-button-row">

            <button
              onclick={startSimulation}
              disabled={simulationRunning}
            >
              ▶ Run Simulation
            </button>

            <button
              onclick={stopSimulation}
              disabled={!simulationRunning}
            >
              ■ Stop
            </button>

          </div>

        </div>


        <!-- INFO -->

        <button
          class="info-button"
          onclick={() => showInfo = !showInfo}
        >
          ℹ How to Test SmartCase
        </button>


        {#if showInfo}

          <div class="info-box">

            <strong>
              Testing Instructions
            </strong>

            <p>
              Use the controls above to simulate sensor readings
              and user actions. Weight controls change the suitcase
              weight, packing controls change the checklist,
              security controls affect the lock and Bluetooth
              connection, and the four scenarios load different
              traveler situations.
            </p>

            <p>
              The mock phone can also control Find My, locking,
              and Follow Mode. The Airport Follow Simulation shows
              SmartCase moving with a traveler over time.
            </p>

          </div>

        {/if}

      </div>


      <!-- =================================================
           MOCK PHONE UI
      ================================================== -->

      <aside class="phone-area">

        <p class="phone-label">
          SECONDARY DEVICE UI
        </p>


        <div class="phone">

          <div class="phone-speaker"></div>


          <div class="phone-screen">

            <div class="phone-status-bar">

              <span>9:41</span>

              <span>
                {phoneBattery}% ▰
              </span>

            </div>


            <div class="phone-app-header">

              <div>

                <small>
                  SMARTCASE APP
                </small>

                <h3>
                  My SmartCase
                </h3>

              </div>

              <div
                class="connection-dot"
                class:connection-off={!bluetoothConnected}
              ></div>

            </div>


            <div
              class="phone-connection"
              class:phone-disconnected={!bluetoothConnected}
            >

              <svg
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
              >
                <path d="M7 7l10 10-5 4V3l5 4L7 17" />
              </svg>

              {bluetoothConnected
                ? "Connected"
                : "Disconnected"}

            </div>


            <div
              class="map-card"
              class:map-finding={isFinding}
            >

              <div class="map-grid"></div>

              <div class="map-pin">
                ●
              </div>

              <div class="map-info">

                <strong>
                  {location}
                </strong>

                <span>
                  {bagDistance.toFixed(1)} m away
                </span>

              </div>

            </div>


            <div class="phone-stat-grid">

              <div>
                <span>Weight</span>

                <strong>
                  {weight.toFixed(1)}
                  kg
                </strong>
              </div>

              <div>
                <span>Battery</span>

                <strong>
                  {battery}%
                </strong>
              </div>

            </div>


            <div class="phone-controls">

              <button onclick={toggleLock}>
                {isLocked ? "🔓 Unlock" : "🔒 Lock"}
              </button>

              <button onclick={findBag}>
                ⌖ Find Bag
              </button>

              <button
                class:follow-active={followMode}
                onclick={toggleFollowMode}
                disabled={!bluetoothConnected}
              >
                {followMode
                  ? "■ Stop Following"
                  : "➜ Follow Mode"}
              </button>

            </div>


            {#if followMode}

              <div class="follow-card">

                <div class="follow-pulse"></div>

                <div>
                  <strong>
                    Following You
                  </strong>

                  <span>
                    Maintaining approximately
                    {bagDistance.toFixed(1)} m distance
                  </span>
                </div>

              </div>

            {/if}


            {#if usbCharging}

              <div class="charging-banner">
                ⚡ Charging phone through SmartCase USB-C
              </div>

            {/if}

          </div>

        </div>

      </aside>

    </div>

  </section>

</main>