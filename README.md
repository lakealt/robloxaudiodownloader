<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Roblox Asset Inspector</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #111;
      color: #fff;
      margin: 0;
      padding: 20px;
      max-width: 900px;
      margin-left: auto;
      margin-right: auto;
    }
    input, button {
      padding: 10px;
      margin: 8px 0;
      font-size: 1rem;
      border-radius: 6px;
      border: none;
      width: 100%;
      max-width: 400px;
    }
    button {
      background: #4e8cff;
      color: white;
      cursor: pointer;
      transition: background-color 0.3s;
    }
    button:hover {
      background: #3c70d6;
    }
    .section {
      background: #222;
      padding: 20px;
      margin-top: 20px;
      border-radius: 10px;
    }
    .result {
      margin-top: 12px;
      word-wrap: break-word;
    }
    img {
      max-width: 120px;
      border-radius: 8px;
      margin-top: 10px;
      display: block;
    }
    hr {
      border: 1px solid #444;
      margin: 15px 0;
    }
  </style>
</head>
<body>
  <h1>Roblox Asset Inspector</h1>

  <div class="section">
    <h2>Fetch Asset Info</h2>
    <input type="number" id="assetId" placeholder="Asset ID" />
    <button onclick="getAssetInfo()">Get Info</button>
    <div class="result" id="assetInfo"></div>
  </div>

  <div class="section">
    <h2>Convert Place ID to Universe ID</h2>
    <input type="number" id="placeIdInput" placeholder="Place ID" />
    <button onclick="getUniverseId()">Convert</button>
    <div class="result" id="universeIdResult"></div>
  </div>

  <div class="section">
    <h2>Fetch Subplaces</h2>
    <input type="number" id="universeIdSubplaces" placeholder="Universe ID" />
    <button onclick="getSubplaces()">Get Subplaces</button>
    <div class="result" id="subplacesResult"></div>
  </div>

  <div class="section">
    <h2>Fetch Universe Badges</h2>
    <input type="number" id="universeBadgeId" placeholder="Universe ID" />
    <button onclick="getUniverseBadges()">Get Badges</button>
    <div class="result" id="badgeResult"></div>
  </div>

  <div class="section">
    <h2>Fetch Badge Info</h2>
    <input type="number" id="badgeId" placeholder="Badge ID" />
    <button onclick="getBadgeInfo()">Get Badge Info</button>
    <div class="result" id="badgeInfo"></div>
  </div>

  <div class="section">
    <h2>Get Badge Award Info</h2>
    <input type="number" id="userIdBadge" placeholder="User ID" />
    <input type="number" id="badgeIdUser" placeholder="Badge ID" />
    <button onclick="getBadgeAward()">Get Award Info</button>
    <div class="result" id="badgeAwardResult"></div>
  </div>

  <div class="section">
    <h2>Get User Info</h2>
    <input type="number" id="userIdInfo" placeholder="User ID" />
    <button onclick="getUserInfo()">Get User Info</button>
    <div class="result" id="userInfo"></div>
  </div>

  <script>
    // Replace this with your actual Cloudflare Worker URL:
    const proxy = "https://your-worker.subdomain.workers.dev";

    async function proxiedFetch(url) {
      const encoded = encodeURIComponent(url);
      const response = await fetch(`${proxy}?url=${encoded}`);
      if (!response.ok) throw new Error(`Request failed: ${response.status}`);
      return response.json();
    }

    async function getAssetInfo() {
      try {
        const id = document.getElementById("assetId").value.trim();
        if (!id) return alert("Please enter an Asset ID");

        const details = await proxiedFetch(`https://economy.roblox.com/v2/assets/${id}/details`);
        const thumb = await proxiedFetch(`https://thumbnails.roblox.com/v1/assets?assetIds=${id}&format=Png&size=420x420`);
        const img = thumb.data?.[0]?.imageUrl || "";

        document.getElementById("assetInfo").innerHTML = `
          <strong>Name:</strong> ${details.name || "N/A"}<br>
          <strong>Description:</strong> ${details.description || "N/A"}<br>
          <strong>Creator:</strong> ${details.creator?.name || "N/A"}<br>
          <img src="${img}" alt="Asset Thumbnail" />
          <br><a href="https://assetdelivery.roblox.com/v1/asset?id=${id}" target="_blank" rel="noopener">Download Asset</a>
        `;
      } catch (e) {
        document.getElementById("assetInfo").textContent = "Failed to fetch asset info.";
      }
    }

    async function getUniverseId() {
      try {
        const placeId = document.getElementById("placeIdInput").value.trim();
        if (!placeId) return alert("Please enter a Place ID");

        const data = await proxiedFetch(`https://apis.roblox.com/universes/v1/places/${placeId}/universe`);
        document.getElementById("universeIdResult").textContent = `Universe ID: ${data.universeId || "Not Found"}`;
      } catch {
        document.getElementById("universeIdResult").textContent = "Failed to fetch universe ID.";
      }
    }

    async function getSubplaces() {
      try {
        const universeId = document.getElementById("universeIdSubplaces").value.trim();
        if (!universeId) return alert("Please enter a Universe ID");

        const data = await proxiedFetch(`https://develop.roblox.com/v1/universes/${universeId}/places?limit=100`);
        if (!data.data || data.data.length === 0) {
          document.getElementById("subplacesResult").textContent = "No subplaces found.";
          return;
        }
        document.getElementById("subplacesResult").innerHTML = data.data
          .map(p => `Place ID: ${p.placeId} - Name: ${p.name}`)
          .join("<br>");
      } catch {
        document.getElementById("subplacesResult").textContent = "Failed to fetch subplaces.";
      }
    }

    async function getUniverseBadges() {
      try {
        const universeId = document.getElementById("universeBadgeId").value.trim();
        if (!universeId) return alert("Please enter a Universe ID");

        const data = await proxiedFetch(`https://badges.roblox.com/v1/universes/${universeId}/badges?limit=100`);
        if (!data.data || data.data.length === 0) {
          document.getElementById("badgeResult").textContent = "No badges found.";
          return;
        }
        document.getElementById("badgeResult").innerHTML = data.data
          .map(b => `<img src="${b.imageUrl}" alt="${b.name}" /><br><strong>${b.name}</strong><br>${b.description}<hr>`)
          .join("");
      } catch {
        document.getElementById("badgeResult").textContent = "Failed to fetch badges.";
      }
    }

    async function getBadgeInfo() {
      try {
        const badgeId = document.getElementById("badgeId").value.trim();
        if (!badgeId) return alert("Please enter a Badge ID");

        const data = await proxiedFetch(`https://badges.roblox.com/v1/badges/${badgeId}`);
        document.getElementById("badgeInfo").innerHTML = `
          <strong>${data.name}</strong><br>
          <img src="${data.imageUrl}" alt="${data.name}" /><br>
          ${data.description}<br>
          Enabled: ${data.enabled}<br>
          Created: ${new Date(data.created).toLocaleString()}
        `;
      } catch {
        document.getElementById("badgeInfo").textContent = "Failed to fetch badge info.";
      }
    }

    async function getBadgeAward() {
      try {
        const userId = document.getElementById("userIdBadge").value.trim();
        const badgeId = document.getElementById("badgeIdUser").value.trim();
        if (!userId || !badgeId) return alert("Please enter both User ID and Badge ID");

        const data = await proxiedFetch(`https://badges.roblox.com/v1/users/${userId}/badges/awarded-dates?badgeIds=${badgeId}`);
        const badge = data.data?.[0];
        document.getElementById("badgeAwardResult").textContent = badge ? `Awarded at: ${new Date(badge.awardedDate).toLocaleString()}` : "Badge not awarded or invalid ID";
      } catch {
        document.getElementById("badgeAwardResult").textContent = "Failed to fetch badge award info.";
      }
    }

    async function getUserInfo() {
      try {
        const userId = document.getElementById("userIdInfo").value.trim();
        if (!userId) return alert("Please enter a User ID");

        const data = await proxiedFetch(`https://users.roblox.com/v1/users/${userId}`);
        const thumbData = await proxiedFetch(`https://thumbnails.roblox.com/v1/users/avatar-headshot?userIds=${userId}&size=150x150&format=Png`);
        const img = thumbData.data?.[0]?.imageUrl || "";

        document.getElementById("userInfo").innerHTML = `
          <strong>${data.displayName} (@${data.name})</strong><br>
          <img src="${img}" alt="User Avatar" /><br>
          Description: ${data.description || "N/A"}<br>
          Created: ${new Date(data.created).toLocaleString()}<br>
          Is Banned: ${data.isBanned ? "Yes" : "No"}
        `;
      } catch {
        document.getElementById("userInfo").textContent = "Failed to fetch user info.";
      }
    }
  </script>
</body>
</html>
