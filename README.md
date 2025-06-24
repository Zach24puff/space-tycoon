<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
  <title>Space Station Tycoon 3D</title>
  <style>
    html, body {
      margin: 0;
      padding: 0;
      overflow: hidden;
      touch-action: none;
      -webkit-user-select: none;
      user-select: none;
      -webkit-tap-highlight-color: transparent;
      background-color: #000;
    }

    #ui {
      position: fixed;
      top: 10px;
      left: 10px;
      background: rgba(0, 0, 0, 0.75);
      padding: 15px;
      border-radius: 10px;
      box-shadow: 0 0 10px #00ffcc;
      z-index: 10;
      color: white;
      font-family: Arial, sans-serif;
      width: 90vw;
      max-width: 300px;
      font-size: 1.2em;
    }

    .circle-trigger {
      position: absolute;
      width: 40px;
      height: 40px;
      border-radius: 50%;
      background: rgba(0, 255, 204, 0.6);
      box-shadow: 0 0 10px rgba(0, 255, 204, 0.9);
      pointer-events: none;
    }

    #languageSelect {
      position: fixed;
      top: 10px;
      right: 10px;
      z-index: 11;
      font-size: 1em;
      background: rgba(0, 0, 0, 0.7);
      color: white;
      border: 1px solid #00ffcc;
      border-radius: 5px;
      padding: 5px;
    }
  </style>
</head>
<body>
  <select id="languageSelect">
    <option value="de">Deutsch</option>
    <option value="en">English</option>
    <option value="fr">Français</option>
    <option value="es">Español</option>
  </select>

  <div id="ui">
    <h1 id="title" style="font-size: 1.5em;">🚀 Space Station Tycoon</h1>
    <div>💰 <span id="labelCredits">Credits</span>: <span id="credits">200</span></div>
    <div>🔬 <span id="labelLabor">Labor</span>: <span id="labor">0</span></div>
    <div>🏪 <span id="labelHandel">Handelsstation</span>: <span id="handel">0</span></div>
    <div>🔋 <span id="labelSolar">Solarpanels</span>: <span id="solar">0</span></div>
    <div>🪐 <span id="labelPlatform">Plattformen</span>: <span id="platform">1</span> (100x100 gehört dir)</div>
  </div>

  <script>
    const uiFields = ['credits', 'labor', 'handel', 'solar', 'platform'];
    const state = {
      credits: 200,
      labor: 0,
      handel: 0,
      solar: 0,
      platform: 1
    };

    const translations = {
      de: {
        title: '🚀 Space Station Tycoon',
        credits: 'Credits',
        labor: 'Labor',
        handel: 'Handelsstation',
        solar: 'Solarpanels',
        platform: 'Plattformen'
      },
      en: {
        title: '🚀 Space Station Tycoon',
        credits: 'Credits',
        labor: 'Lab',
        handel: 'Trade Hub',
        solar: 'Solar Panels',
        platform: 'Platforms'
      },
      fr: {
        title: '🚀 Tycoon de Station Spatiale',
        credits: 'Crédits',
        labor: 'Laboratoire',
        handel: 'Centre Commercial',
        solar: 'Panneaux Solaires',
        platform: 'Plates-formes'
      },
      es: {
        title: '🚀 Magnate de la Estación Espacial',
        credits: 'Créditos',
        labor: 'Laboratorio',
        handel: 'Centro Comercial',
        solar: 'Paneles Solares',
        platform: 'Plataformas'
      }
    };

    function updateUI() {
      uiFields.forEach(id => {
        const el = document.getElementById(id);
        if (el) el.innerText = state[id];
      });
    }

    function setLanguage(lang) {
      const t = translations[lang] || translations['de'];
      document.getElementById('title').innerText = t.title;
      document.getElementById('labelCredits').innerText = t.credits;
      document.getElementById('labelLabor').innerText = t.labor;
      document.getElementById('labelHandel').innerText = t.handel;
      document.getElementById('labelSolar').innerText = t.solar;
      document.getElementById('labelPlatform').innerText = t.platform;
    }

    document.getElementById('languageSelect').addEventListener('change', (e) => {
      setLanguage(e.target.value);
    });

    setInterval(() => {
      state.credits += 10;
      updateUI();
    }, 1000);

    updateUI();
    setLanguage('de');
  </script>
</body>
</html>
