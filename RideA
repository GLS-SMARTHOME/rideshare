<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Riders – Amount + ETA</title>
  <style>
    body { font-family: sans-serif; background: #0f172a; color: #e2e8f0; margin: 0; padding: 20px; }
    h1 { font-size: 20px; margin-bottom: 16px; }
    ul { list-style: none; padding: 0; margin: 0; display: grid; gap: 12px; }
    li {
      display: flex; flex-direction: column; gap: 6px;
      padding: 14px 16px; border-radius: 12px;
      background: #1e293b; border: 1px solid #334155;
      cursor: pointer; transition: background 0.2s;
    }
    li:hover { background: #334155; }
    .amount { font-size: 25px; font-weight: bold; color: #4ade80; }
    .eta { font-size: 14px; color: #93c5fd; }
  </style>
</head>
<body>
  <h1>Riders – Live Info</h1>
  <ul id="riderList"></ul>

  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
    import { getDatabase, ref, set, onChildAdded, onChildChanged, onChildRemoved }
      from "https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

    // Firebase config
    const firebaseConfig = {
      apiKey: "AIzaSyAIlkvwyaF6FPPbXBJDqHwuTS5S3QKG3l8",
      authDomain: "xoxo-store.firebaseapp.com",
      databaseURL: "https://xoxo-store-default-rtdb.firebaseio.com",
      projectId: "xoxo-store",
      storageBucket: "xoxo-store.firebasestorage.app",
      messagingSenderId: "667290233046",
      appId: "1:667290233046:web:dbbc9130ff38b0900f8177",
      measurementId: "G-XMJVCK8KMD"
    };

    const app = initializeApp(firebaseConfig);
    const db = getDatabase(app);

    const riderList = document.getElementById("riderList");

    // Preset driver ID
    const DRIVER_ID = "Z0000001";

    // Google Distance service
    let distanceService;
    function ensureDistanceService() {
      if (!distanceService) distanceService = new google.maps.DistanceMatrixService();
      return distanceService;
    }

    function getETA(origin, destination, cb) {
      ensureDistanceService().getDistanceMatrix(
        {
          origins: [new google.maps.LatLng(origin.lat, origin.lng)],
          destinations: [new google.maps.LatLng(destination.lat, destination.lng)],
          travelMode: "DRIVING",
          unitSystem: google.maps.UnitSystem.METRIC
        },
        (response, status) => {
          if (status === "OK" && response.rows[0].elements[0].status === "OK") {
            const elem = response.rows[0].elements[0];
            cb(`${elem.duration.text} (${elem.distance.text})`);
          } else {
            cb(null);
          }
        }
      );
    }

    function renderRider(id, tripAmount, etaText, riderData) {
      let el = document.getElementById("rider-" + id);
      if (!el) {
        el = document.createElement("li");
        el.id = "rider-" + id;
        el.innerHTML = `
          <div class="amount"></div>
          <div class="eta"></div>
        `;
        // Make it clickable
        el.addEventListener("click", () => {
          const rider = el.riderData;  // always latest data
          if (!rider?.PICK_UP_LOCATION) {
            alert("Pickup location not available.");
            return;
          }
          // Get my location
          navigator.geolocation.getCurrentPosition(pos => {
            const myLoc = { lat: pos.coords.latitude, lng: pos.coords.longitude };
            const pickup = rider.PICK_UP_LOCATION;

            // Compute ETA from my location → pickup
            getETA(myLoc, pickup, etaToPickup => {
              const msg = etaToPickup
                ? `Do you want to accept Rider ${id}?\nETA to pickup: ${etaToPickup}`
                : `Do you want to accept Rider ${id}? (ETA unavailable)`;
              if (confirm(msg)) {
                alert(`✅ Rider ${id} accepted!`);
                // 🔥 Update RTDB with pickup, drop-off, name & phone for this driver
                if (rider.PICK_UP_LOCATION) {
                  set(ref(db, `Drivers/${DRIVER_ID}/PICK_UP_LOCATION`), rider.PICK_UP_LOCATION);
                }
                if (rider.DROP_OFF_LOCATION) {
                  set(ref(db, `Drivers/${DRIVER_ID}/DROP_OFF_LOCATION`), rider.DROP_OFF_LOCATION);
                }
                if (rider.NAME) {
                  set(ref(db, `Drivers/${DRIVER_ID}/NAME`), rider.NAME);
                }
                if (rider.PHONE) {
                  set(ref(db, `Drivers/${DRIVER_ID}/PHONE`), rider.PHONE);
                }
              }
            });
          }, () => alert("Could not get your location."));
        });
        riderList.appendChild(el);
      }

      const amtEl = el.querySelector(".amount");
      amtEl.textContent = tripAmount
        ? `+${new Intl.NumberFormat("en-US", { style: "currency", currency: "USD" }).format(tripAmount)}`
        : "+$0.00";
      el.querySelector(".eta").textContent = etaText ? `Durée: ${etaText}` : "Durée: --";

      // Attach latest rider data for click handler
      el.riderData = riderData;
    }

    async function handleRider(id, val) {
      const amount = val.TRIP_AMOUNT;
      if (val.PICK_UP_LOCATION && val.DROP_OFF_LOCATION) {
        getETA(val.PICK_UP_LOCATION, val.DROP_OFF_LOCATION, etaText => {
          renderRider(id, amount, etaText, val);
        });
      } else {
        renderRider(id, amount, null, val);
      }
    }

    const ridersRef = ref(db, "Riders");
    onChildAdded(ridersRef, snap => handleRider(snap.key, snap.val() || {}));
    onChildChanged(ridersRef, snap => handleRider(snap.key, snap.val() || {}));
    onChildRemoved(ridersRef, snap => {
      const el = document.getElementById("rider-" + snap.key);
      if (el) el.remove();
    });
  </script>

  <!-- Load Google Maps JS API with Distance Matrix -->
  <script async
    src="https://maps.googleapis.com/maps/api/js?key=AIzaSyDjdnkFWOnh1u2cWf5RFURytU8Zk3fmOhk&libraries=places&callback=initMap">
  </script>
  <script>
    function initMap() {
      console.log("Google Maps API loaded");
      distanceService = new google.maps.DistanceMatrixService();
    }
  </script>
</body>
</html>
