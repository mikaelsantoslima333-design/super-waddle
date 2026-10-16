
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="theme-color" content="#10131b">
  <title>Nexa AI</title>
  <style>
    * { box-sizing: border-box; }
    body {
      margin: 0; background: #10131b; color: #f3f5fa;
      font-family: system-ui, sans-serif;
    }
    header {
      padding: 18px; border-bottom: 1px solid #292e3b;
      display: flex; align-items: center; gap: 12px;
    }
    .logo {
      background: #7c6cff; width: 42px; height: 42px;
      border-radius: 14px; display: grid; place-items: center;
      font-size: 22px; font-weight: bold;
    }
    h1 { font-size: 19px; margin: 0; }
    .sub { color: #a6aec2; font-size: 12px; margin-top: 3px; }
    main {
      max-width: 760px; margin: auto; padding: 18px;
    }
    #messages {
      display: flex; flex-direction: column; gap: 12px;
      min-height: 55vh; padding-bottom: 20px;
    }
    .msg {
      padding: 13px 15px; border-radius: 16px;
      max-width: 90%; white-space: pre-wrap; overflow-wrap: anywhere;
      line-height: 1.5;
    }
    .bot { background: #202635; align-self: flex-start; }
    .user { background: #5546ca; align-self: flex-end; }
    .composer {
      position: sticky; bottom: 0; background: #10131b;
      padding: 10px 0 16px;
    }
    textarea {
      width: 100%; resize: vertical; min-height: 54px;
      border: 1px solid #343b4b; border-radius: 14px;
      padding: 14px; background: #191e2a; color: white;
      font: inherit;
    }
    button {
      border: 0; border-radius: 12px; padding: 12px 15px;
      color: white; background: #6b5ce7; font: inherit;
      cursor: pointer;
    }
    .actions { display: flex; gap: 8px; margin-top: 9px; flex-wrap: wrap; }
    .secondary { background: #292f3e; }
    .note { color: #9ba5ba; font-size: 12px; line-height: 1.5; }
  </style>
</head>
<body>
  <header>
    <div class="logo">N</div>
    <div>
      <h1>Nexa AI</h1>
      <div class="sub">Seu projeto pessoal de inteligência artificial</div>
    </div>
  </header>

  <main>
    <div id="messages" aria-live="polite"></div>

    <div class="composer">
      <textarea id="prompt" placeholder="Escreva sua mensagem..."
        aria-label="Mensagem"></textarea>
      <div class="actions">
        <button id="send">Enviar</button>
        <button id="voice" class="secondary">🔊 Ler resposta</button>
        <button id="clear" class="secondary">Limpar conversa</button>
      </div>
      <p class="note">
        Protótipo inicial: as respostas ainda são demonstrativas.
        O histórico fica salvo neste navegador.
      </p>
    </div>
  </main>

  <script>
    const messages = document.querySelector("#messages");
    const promptBox = document.querySelector("#prompt");
    const storageKey = "nexa-ai-chat-v1";
    let history = [];

    function render() {
      messages.replaceChildren();

      history.forEach(item => {
        const bubble = document.createElement("div");
        bubble.className = "msg " +
          (item.role === "user" ? "user" : "bot");
        bubble.textContent = item.text;
        messages.appendChild(bubble);
      });

      messages.scrollTop = messages.scrollHeight;
      window.scrollTo(0, document.body.scrollHeight);
    }

    function save() {
      try {
        localStorage.setItem(storageKey, JSON.stringify(history));
      } catch {
        alert("Não foi possível salvar o histórico neste navegador.");
      }
    }

    function replyTo(text) {
      const t = text.toLowerCase();

      if (t.includes("seu nome") || t.includes("quem é você")) {
        return "Sou a Nexa AI, um projeto pessoal de chatbot. " +
          "Ainda estou na fase inicial de desenvolvimento.";
      }

      if (t.includes("quem criou")) {
        return "Este projeto está sendo desenvolvido por você, " +
          "com ajuda para programar e construir suas funções.";
      }

      return "Recebi sua mensagem: \"" + text + "\"\n\n" +
        "Minha inteligência real ainda será conectada. " +
        "O próximo passo é integrar um modelo de linguagem.";
    }

    function send() {
      const text = promptBox.value.trim();
      if (!text) return;

      history.push({ role: "user", text });
      history.push({ role: "assistant", text: replyTo(text) });
      promptBox.value = "";
      save();
      render();
    }

    document.querySelector("#send").addEventListener("click", send);

    promptBox.addEventListener("keydown", event => {
      if (event.key === "Enter" && !event.shiftKey) {
        event.preventDefault();
        send();
      }
    });

    document.querySelector("#voice").addEventListener("click", () => {
      const last = [...history].reverse()
        .find(item => item.role === "assistant");
      if (!last || !("speechSynthesis" in window)) {
        alert("A leitura de voz não está disponível neste navegador.");
        return;
      }
      speechSynthesis.cancel();
      const utterance = new SpeechSynthesisUtterance(last.text);
      utterance.lang = "pt-BR";
      speechSynthesis.speak(utterance);
    });

    document.querySelector("#clear").addEventListener("click", () => {
      if (!confirm("Deseja apagar o histórico desta conversa?")) return;
      history = [];
      save();
      render();
    });

    try {
      const saved = JSON.parse(localStorage.getItem(storageKey) || "[]");
      if (Array.isArray(saved)) history = saved;
    } catch {
      history = [];
    }

    if (!history.length) {
      history.push({
        role: "assistant",
        text: "Olá! Sou a Nexa AI. Vamos construir minha inteligência " +
          "passo a passo. Por enquanto, minhas respostas são demonstrativas."
      });
    }

    render();
  </script>
</body>
</html>
