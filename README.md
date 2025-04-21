<!DOCTYPE html>
<html lang="pl">
<head>
  <meta charset="UTF-8">
  <title>MeteoQuiz</title>
  <style>
    body {
      font-family: sans-serif;
      background: #f0f8ff;
      padding: 40px;
      display: flex;
      flex-direction: row;
      justify-content: center;
      gap: 40px;
      position: relative;
    }
    .main-container {
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    h1 {
      color: #006064;
    }
    .quiz-box {
      background: white;
      border-radius: 12px;
      padding: 20px;
      max-width: 500px;
      width: 100%;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
      margin-top: 20px;
    }
    label {
      font-weight: bold;
      display: block;
      margin-top: 10px;
    }
    select {
      width: 100%;
      padding: 8px;
      margin-top: 5px;
      margin-bottom: 15px;
      border-radius: 5px;
      border: 1px solid #ccc;
      font-size: 1em;
    }
    button {
      padding: 10px 20px;
      background-color: #00796b;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      font-size: 1em;
    }
    .feedback {
      margin-top: 15px;
      padding: 10px;
      border-radius: 5px;
    }
    .correct {
      background: #e0f2f1;
      color: #00796b;
    }
    .incorrect {
      background: #ffebee;
      color: #c62828;
    }
    ul.result-list {
      padding-left: 20px;
    }
    .log-box {
      max-width: 300px;
      width: 100%;
      background: #ffffff;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
      overflow-y: auto;
      max-height: 600px;
    }
    .log-box h2 {
      font-size: 1.2em;
      margin-bottom: 10px;
    }
    .log-entry {
      font-size: 0.95em;
      margin-bottom: 10px;
      border-bottom: 1px solid #ccc;
      padding-bottom: 5px;
    }
    #fake-login {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.7);
      color: white;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 2em;
      z-index: 9999;
    }
    #fake-login.hidden {
      display: none;
    }
  </style>
</head>
<body>
  <div id="fake-login">🔒 Zaloguj się, aby kontynuować</div>

  <div class="main-container">
    <h1>MeteoQuiz – wybierz poprawne opcje!</h1>
    <div class="quiz-box">
      <label for="zjawisko">Zjawisko pogodowe:</label>
      <select id="zjawisko"></select>

      <label for="urzadzenie">Urządzenie pomiarowe:</label>
      <select id="urzadzenie"></select>

      <label for="jednostka">Jednostka pomiaru:</label>
      <select id="jednostka"></select>

      <button onclick="sprawdz()">Sprawdź</button>
      <div id="feedback" class="feedback"></div>
    </div>
  </div>

  <div class="log-box">
    <h2>Historia odpowiedzi</h2>
    <div id="log"></div>
  </div>

  <script>
    document.addEventListener('DOMContentLoaded', () => {
      let clickCount = 0;
      let clickTimer;
      const fakeLogin = document.getElementById('fake-login');

      document.body.addEventListener('click', () => {
        clickCount++;
        clearTimeout(clickTimer);
        clickTimer = setTimeout(() => clickCount = 0, 1000);
        if (clickCount >= 5) {
          fakeLogin.classList.add('hidden');
        }
      });

      const poprawne = {
        "temperatura powietrza": { urzadzenie: "termometr", jednostka: "stopnie celsjusza" },
        "ciśnienie atmosferyczne": { urzadzenie: "barometr", jednostka: "hektopaskale" },
        "wilgotność powietrza": { urzadzenie: "higrometr", jednostka: "procenty" },
        "opady atmosferyczne": { urzadzenie: "deszczomierz", jednostka: "milimetry" },
        "wiatr - kierunek": { urzadzenie: "wiatrowskaz", jednostka: "kierunki świata" },
        "wiatr - prędkość": { urzadzenie: "anemometr", jednostka: "metry na sekundę" },
        "zachmurzenie": { urzadzenie: "obserwacja wzrokowa", jednostka: "procenty" },
        "usłonecznienie": { urzadzenie: "heliograf", jednostka: "godziny" },
      };

      const urzadzeniaOpcje = [
        "termometr", "termometrr", "barometr", "barometrr", "higrometr", "hygrometr",
        "deszczomierz", "deszczomieszz", "wiatrowskaz", "wiatroskaz",
        "anemometr", "anemmetrr", "obserwacja wzrokowa", "obserwacja satelitarna",
        "heliograf", "heliometrr", "radar pogodowy", "czujnik ciśnienia"
      ];

      const jednostkiOpcje = [
        "stopnie celsjusza", "stopnie celcjusza", "hektopaskale", "hektopaskale",
        "procenty", "procent", "milimetry", "milimetrów",
        "kierunki świata", "kierunek świata", "metry na sekundę", "km/h",
        "godziny", "minuty", "punkty", "skala 0–8"
      ];

      const zjawiska = Object.keys(poprawne);
      const zjawiskaSelect = document.getElementById("zjawisko");
      const urzadzenieSelect = document.getElementById("urzadzenie");
      const jednostkaSelect = document.getElementById("jednostka");
      const logContainer = document.getElementById("log");
      const feedback = document.getElementById("feedback");

      function shuffleArray(array) {
        return array.sort(() => Math.random() - 0.5);
      }

      function odswiezListe() {
        zjawiskaSelect.innerHTML = '<option value="">-- wybierz --</option>';
        shuffleArray(zjawiska).forEach(z => {
          const option = document.createElement("option");
          option.value = z;
          option.textContent = z.charAt(0).toUpperCase() + z.slice(1);
          zjawiskaSelect.appendChild(option);
        });

        urzadzenieSelect.innerHTML = '<option value="">-- wybierz --</option>';
        shuffleArray(urzadzeniaOpcje).forEach(u => {
          const option = document.createElement("option");
          option.value = u;
          option.textContent = u.charAt(0).toUpperCase() + u.slice(1);
          urzadzenieSelect.appendChild(option);
        });

        jednostkaSelect.innerHTML = '<option value="">-- wybierz --</option>';
        shuffleArray(jednostkiOpcje).forEach(j => {
          const option = document.createElement("option");
          option.value = j;
          option.textContent = j.charAt(0).toUpperCase() + j.slice(1);
          jednostkaSelect.appendChild(option);
        });
      }

      odswiezListe();

      window.sprawdz = function () {
        const zjawisko = zjawiskaSelect.value;
        const urzadzenie = urzadzenieSelect.value.toLowerCase();
        const jednostka = jednostkaSelect.value.toLowerCase();

        if (zjawisko && poprawne[zjawisko]) {
          const poprawneUrz = poprawne[zjawisko].urzadzenie;
          const poprawneJedn = poprawne[zjawisko].jednostka;

          let wyniki = '<ul class="result-list">';
          let dobrze = true;

          if (urzadzenie === poprawneUrz) {
            wyniki += `<li>✅ Urządzenie poprawne: ${urzadzenie}</li>`;
          } else {
            wyniki += `<li>❌ Urządzenie błędne. Powinno być: ${poprawneUrz}</li>`;
            dobrze = false;
          }

          if (jednostka === poprawneJedn) {
            wyniki += `<li>✅ Jednostka poprawna: ${jednostka}</li>`;
          } else {
            wyniki += `<li>❌ Jednostka błędna. Powinno być: ${poprawneJedn}</li>`;
            dobrze = false;
          }

          wyniki += "</ul>";
          feedback.innerHTML = (dobrze ? "✅ Wszystko poprawnie!" : "🔎 Sprawdź swoje odpowiedzi:") + wyniki;
          feedback.className = "feedback " + (dobrze ? "correct" : "incorrect");

          const logEntry = document.createElement("div");
          logEntry.classList.add("log-entry");
          logEntry.innerHTML = `<strong>${zjawisko}</strong><br>${wyniki}`;
          logContainer.prepend(logEntry);

          if (dobrze) {
            const index = zjawiska.indexOf(zjawisko);
            if (index !== -1) {
              zjawiska.splice(index, 1);
              odswiezListe();
            }
          }
        } else {
          feedback.innerHTML = "❌ Wybierz zjawisko pogodowe.";
          feedback.className = "feedback incorrect";
        }
      }
    });
  </script>
</body>
</html>
