
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Roblox Audio Downloader</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #1c1c1c;
      color: white;
      text-align: center;
      padding: 40px;
    }
    input, button {
      padding: 10px;
      margin: 10px;
      font-size: 1rem;
    }
    #audioList {
      margin-top: 30px;
    }
    a {
      color: #4ef;
    }
  </style>
</head>
<body>
  <h1>Roblox Audio Downloader</h1>
  <p>Enter a Roblox Place ID to find and download its public audios:</p>
  <input type="text" id="placeIdInput" placeholder="Enter Place ID">
  <button onclick="fetchAudios()">Fetch Audios</button>

  <div id="audioList"></div>

  <script>
    async function fetchAudios() {
      const placeId = document.getElementById('placeIdInput').value;
      const audioListDiv = document.getElementById('audioList');
      audioListDiv.innerHTML = 'Fetching...';

      try {
        // Step 1: Get universe ID from place ID
        const placeRes = await fetch(`https://games.roblox.com/v1/games/multiget-place-details?placeIds=${placeId}`);
        const placeData = await placeRes.json();
        const universeId = placeData[0]?.universeId;
        if (!universeId) throw new Error("Invalid Place ID or game not found.");

        // Step 2: Get all audio assets for the universe
        const assetsRes = await fetch(`https://develop.roblox.com/v1/universes/${universeId}/assets?assetTypes=Audio`);
        const assetsData = await assetsRes.json();

        if (!assetsData.data.length) {
          audioListDiv.innerHTML = 'No public audio assets found.';
          return;
        }

        // Step 3: Display downloadable links
        audioListDiv.innerHTML = '<h3>Audio Files:</h3>';
        assetsData.data.forEach(async asset => {
          const deliveryRes = await fetch(`https://assetdelivery.roblox.com/v1/asset?id=${asset.id}`);
          const deliveryUrl = deliveryRes.url;

          const audioLink = document.createElement('a');
          audioLink.href = deliveryUrl;
          audioLink.download = '';
          audioLink.textContent = `${asset.name} [Download]`;
          audioLink.target = '_blank';

          const div = document.createElement('div');
          div.appendChild(audioLink);
          audioListDiv.appendChild(div);
        });
      } catch (err) {
        audioListDiv.innerHTML = 'Error: ' + err.message;
      }
    }
  </script>
</body>
</html>

