<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Roblox Inspector</title>
  <style>
    :root {
      --bg: #0f0f0f;
      --fg: #fff;
      --accent: #4e8cff;
      --section-bg: #1a1a1a;
      --border-radius: 12px;
      --padding: 16px;
    }
    body {
      background: var(--bg);
      color: var(--fg);
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      margin: 0;
      padding: var(--padding);
    }
    h1, h2 {
      color: var(--accent);
    }
    input, button {
      padding: 10px;
      font-size: 1rem;
      border-radius: 8px;
      border: none;
      margin: 5px 0;
    }
    input {
      width: calc(100% - 24px);
      background: #222;
      color: var(--fg);
    }
    button {
      background: var(--accent);
      color: white;
      cursor: pointer;
    }
    button:hover {
      background: #3c70d6;
    }
    .container {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
      gap: 20px;
    }
    .section {
      background: var(--section-bg);
      padding: var(--padding);
      border-radius: var(--border-radius);
      box-shadow: 0 0 10px #0004;
    }
    .result {
      margin-top: 10px;
      font-size: 0.95rem;
    }
    img {
      max-width: 100px;
      border-radius: 8px;
    }
    audio {
      width: 100%;
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <h1>🎧 Roblox Asset Inspector</h1>
  <div class="container">
    <div class="section">
      <h2>Asset Info</h2>
      <input type="text" id="assetId" placeholder="Asset ID">
      <button onclick="getAssetInfo()">Get Info</button>
      <div class="result" id="assetInfo"></div>
    </div>
    <div class="section">
      <h2>Place to Universe</h2>
      <input type="text" id="placeIdInput" placeholder="Place ID">
      <button onclick="getUniverseId()">Convert</button>
      <div class="result" id="universeIdResult"></div>
    </div>
    <div class="section">
      <h2>Subplaces</h2>
      <input type="text" id="universeIdSubplaces" placeholder="Universe ID">
      <button onclick="getSubplaces()">Get Subplaces</button>
      <div class="result" id="subplacesResult"></div>
    </div>
    <div class="section">
      <h2>Badges</h2>
      <input type="text" id="universeBadgeId" placeholder="Universe ID">
      <button onclick="getUniverseBadges()">Get Badges</button>
      <div class="result" id="badgeResult"></div>
    </div>
    <div class="section">
      <h2>Badge Info</h2>
      <input type="text" id="badgeId" placeholder="Badge ID">
      <button onclick="getBadgeInfo()">Fetch</button>
      <div class="result" id="badgeInfo"></div>
    </div>
    <div class="section">
      <h2>User Badge Time</h2>
      <input type="text" id="userIdBadge" placeholder="User ID">
      <input type="text" id="badgeIdUser" placeholder="Badge ID">
      <button onclick="getBadgeAward()">Check</button>
      <div class="result" id="badgeAwardResult"></div>
    </div>
    <div class="section">
      <h2>User Info</h2>
      <input type="text" id="userIdInfo" placeholder="User ID">
      <button onclick="getUserInfo()">Fetch</button>
      <div class="result" id="userInfo"></div>
    </div>
  </div>
  <script>
    const proxy = "https://old-dust-fd75.devrahsanko.workers.dev";
    async function proxiedFetch(url) {
      const encoded = encodeURIComponent(url);
      const res = await fetch(`${proxy}?url=${encoded}`);
      if (!res.ok) throw new Error("Request failed");
      return res.json();
    }
    async function getAssetInfo() {
      const id = document.getElementById("assetId").value;
      const details = await proxiedFetch(`https://economy.roblox.com/v2/assets/${id}/details`);
      const thumb = await proxiedFetch(`https://thumbnails.roblox.com/v1/assets?assetIds=${id}&format=Png&size=420x420`);
      const audioPreview = details.AssetTypeId === 3 ? `<audio controls src='https://assetdelivery.roblox.com/v1/asset?id=${id}'></audio>` : "";
      document.getElementById("assetInfo").innerHTML = `
        <strong>Name:</strong> ${details.Name}<br>
        <strong>Desc:</strong> ${details.Description}<br>
        <strong>Creator:</strong> ${details.Creator.Name}<br>
        <strong>Is For Sale:</strong> ${details.IsForSale}<br>
        <strong>Is Public Domain:</strong> ${details.IsPublicDomain}<br>
        <img src="${thumb.data[0]?.imageUrl}"/><br>
        ${audioPreview}<br>
        <a href="https://assetdelivery.roblox.com/v1/asset?id=${id}" target="_blank">Download Asset</a>
      `;
    }
    async function getUniverseId() {
      const id = document.getElementById("placeIdInput").value;
      const data = await proxiedFetch(`https://apis.roblox.com/universes/v1/places/${id}/universe`);
      document.getElementById("universeIdResult").textContent = `Universe ID: ${data.universeId}`;
    }
    async function getSubplaces() {
      const id = document.getElementById("universeIdSubplaces").value;
      const data = await proxiedFetch(`https://develop.roblox.com/v1/universes/${id}/places?limit=100`);
      document.getElementById("subplacesResult").innerHTML = data.data.map(p => `Place ID: ${p.placeId} - ${p.name}`).join('<br>');
    }
    async function getUniverseBadges() {
      const id = document.getElementById("universeBadgeId").value;
      const data = await proxiedFetch(`https://badges.roblox.com/v1/universes/${id}/badges?limit=100`);
      document.getElementById("badgeResult").innerHTML = data.data.map(b => `<img src="${b.imageUrl}"/><br>${b.name}: ${b.description}`).join('<hr>');
    }
    async function getBadgeInfo() {
      const id = document.getElementById("badgeId").value;
      const b = await proxiedFetch(`https://badges.roblox.com/v1/badges/${id}`);
      document.getElementById("badgeInfo").innerHTML = `<strong>${b.name}</strong><br><img src="${b.imageUrl}"/><br>${b.description}<br>Created: ${b.created}`;
    }
    async function getBadgeAward() {
      const user = document.getElementById("userIdBadge").value;
      const badge = document.getElementById("badgeIdUser").value;
      const data = await proxiedFetch(`https://badges.roblox.com/v1/users/${user}/badges/awarded-dates?badgeIds=${badge}`);
      document.getElementById("badgeAwardResult").textContent = data.data[0]?.awardedDate || "Not awarded";
    }
    async function getUserInfo() {
      const id = document.getElementById("userIdInfo").value;
      const user = await proxiedFetch(`https://users.roblox.com/v1/users/${id}`);
      const thumb = await proxiedFetch(`https://thumbnails.roblox.com/v1/users/avatar-headshot?userIds=${id}&size=150x150&format=Png`);
      const friends = await proxiedFetch(`https://friends.roblox.com/v1/users/${id}/friends`);
      document.getElementById("userInfo").innerHTML = `
        <strong>${user.displayName} (@${user.name})</strong><br>
        <img src="${thumb.data[0]?.imageUrl}"/><br>
        Description: ${user.description}<br>
        Created: ${user.created}<br>
        Friends (${friends.data.length}):<br>
        ${friends.data.slice(0, 5).map(f => f.name).join(', ')}...
      `;
    }
  </script>
</body>
</html>
