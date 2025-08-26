<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Mini IA – Clasificador de gustos</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-slate-50 min-h-screen flex flex-col">
  <header class="bg-white shadow p-4 text-center">
    <h1 class="text-xl font-bold">Mini IA – Clasificador de gustos</h1>
    <p class="text-sm text-slate-500">Clasifica textos en "me gusta" o "no me gusta" y aprende de tu feedback</p>
  </header>

  <main class="flex-1 container mx-auto p-4 max-w-2xl">
    <label class="block text-sm font-medium text-slate-700">Escribe o pega un texto</label>
    <textarea id="inputText" class="mt-2 w-full h-40 p-3 rounded-xl border focus:outline-none focus:ring-2 focus:ring-indigo-500/50" placeholder="Ej.: Invitación a una fiesta el sábado por la noche"></textarea>
    <div class="mt-3 flex gap-2">
      <button onclick="predict()" class="px-4 py-2 rounded-xl bg-indigo-600 text-white shadow">Analizar</button>
      <button onclick="clearText()" class="px-4 py-2 rounded-xl border bg-white hover:bg-slate-50">Limpiar</button>
    </div>

    <div id="result" class="mt-6 p-4 rounded-xl border bg-white hidden"></div>
    <div id="feedback" class="mt-3 flex gap-2 hidden">
      <button onclick="feedback('like')" class="px-3 py-2 rounded-xl border bg-white hover:bg-slate-50">👍 Me gusta</button>
      <button onclick="feedback('dislike')" class="px-3 py-2 rounded-xl border bg-white hover:bg-slate-50">👎 No me gusta</button>
    </div>
  </main>

  <footer class="text-center text-xs text-slate-500 p-4">Demo local: todo se guarda en tu navegador</footer>

  <script>
    const STORAGE_KEY = "mini-ia-model-v1";
    const seedWeights = { fiesta: 2, invitación: 2, gatitos: 2, perritos: 2, factura: -2, trabajo: -1, examen: -1.5 };

    function tokenize(text) {
      return text.toLowerCase().normalize("NFKD").replace(/[^a-z0-9áéíóúñü\s]/gi, " ").split(/\s+/).filter(Boolean);
    }

    function loadModel() {
      try { return { ...seedWeights, ...JSON.parse(localStorage.getItem(STORAGE_KEY)) }; } catch { return { ...seedWeights }; }
    }

    function saveModel(model) { localStorage.setItem(STORAGE_KEY, JSON.stringify(model)); }

    function scoreText(model, text) {
      const tokens = tokenize(text);
      let score = 0;
      tokens.forEach(t => { score += model[t] ?? 0 });
      const norm = tokens.length ? score / Math.sqrt(tokens.length) : 0;
      const prob = 1 / (1 + Math.exp(-norm));
      return { prob };
    }

    let model = loadModel();

    function predict() {
      const text = document.getElementById("inputText").value;
      if (!text.trim()) return;
      const { prob } = scoreText(model, text);
      const res = document.getElementById("result");
      res.innerHTML = `<strong>${prob >= 0.5 ? 'Me gusta' : 'No me gusta'}</strong><br>Confianza ${(prob*100).toFixed(1)}%`;
      res.classList.remove("hidden");
      document.getElementById("feedback").classList.remove("hidden");
    }

    function feedback(label) {
      const text = document.getElementById("inputText").value;
      if (!text.trim()) return;
      const tokens = tokenize(text);
      const delta = label === "like" ? 1 : -1;
      tokens.forEach(t => { model[t] = (model[t] ?? 0) + 0.6 * delta; });
      saveModel(model);
      predict();
    }

    function clearText() {
      document.getElementById("inputText").value = "";
      document.getElementById("result").classList.add("hidden");
      document.getElementById("feedback").classList.add("hidden");
    }
  </script>
</body>
</html>
