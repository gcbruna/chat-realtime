<template>
  <div class="chat-container">

    <div class="row g-0 h-100">

      <!-- Lado esquerdo: Chat -->
      <div class="col-md-6 d-flex align-items-center justify-content-center bg-light h-100 chat-left">

        <!-- Header minimalista -->
        <div class="header-app d-flex align-items-center px-3">
          <h5 class="m-0 fw-bold">SyncChat</h5>
        </div>

        <!-- Caixa do chat -->
        <div class="col-md-6 d-flex flex-column m-2 chat-box bg-white shadow glass">


          <!-- Tela de escolher usuário -->
          <div
            v-if="!usuarioEscolhido"
            class="d-flex flex-column p-4 align-items-center justify-content-center flex-fill"
          >
            <h3 class="fw-bold mb-4">Escolha seu nome</h3>

            <input
              v-model="usuarioTemp"
              placeholder="Seu nome..."
              class="form-control form-control-lg input-nome mb-3"
            />

            <button class="btn btn-outline-primary btn-lg w-100 botao-entrar" @click="definirUsuario">
              Entrar
            </button>
          </div>

          <!-- Chat -->
          <div v-else class="d-flex flex-column h-100">

            <!-- Área das mensagens -->
            <div id="areaMensagens" class="area-mensagens flex-grow-1 p-3 bg-paper">

              <div
                v-for="(m, i) in mensagens"
                :key="i"
                :class="[
                  'p-2 mb-2 rounded message-bubble',
                  m.usuario === usuario ? 'msg-mine' : 'msg-other'
                ]"
              >
                <small class="fw-bold" v-if="m.usuario !== usuario">{{ m.usuario }}</small><br />
                {{ m.texto }}
              </div>

            </div>

            <!-- Input -->
            <div class="area-input p-2 glass">
              <div class="input-group">
                <input
                  v-model="texto"
                  @keyup.enter="enviar"
                  placeholder="Digite uma mensagem..."
                  class="form-control input-msg"
                />

                <button class="btn btn-outline-primary btn-lg w-5 btn-enviar" @click="enviar">
                  Enviar
                </button>
              </div>
            </div>

          </div>
        </div>
      </div>

      <!-- Painel direito -->
      <div class="col-md-6 d-flex flex-column align-items-center justify-content-center text-white text-center position-relative painel-direito">

        <div class="painel-overlay"></div>

        <div class="position-relative px-5">
          <h1 class="fw-bold display-5 mb-3">Chat em Tempo Real</h1>
          <p class="fs-5 opacity-75">
            Ably + Laravel + Vue<br />
            Comunicação instantânea e moderna.
          </p>
        </div>

      </div>

    </div>
  </div>
</template>

<script>
import * as Ably from "ably";

export default {
  data() {
    return {
      usuario: "",
      usuarioTemp: "",
      usuarioEscolhido: false,
      texto: "",
      mensagens: [],
      ably: null,
      canal: null,
    };
  },

  mounted() {
    this.ably = new Ably.Realtime("WA9TGA.nuLYhw:kYidV4M_-rg5drc1oRmo3lOdKwbxD9Noux8CzwGbkys");

    this.canal = this.ably.channels.get("chat-geral");

    this.canal.subscribe("nova-mensagem", (msg) => {
      this.mensagens.push(msg.data);

      this.$nextTick(() => {
        const box = document.getElementById("areaMensagens");
        box.scrollTop = box.scrollHeight;
      });
    });
  },

  methods: {
    definirUsuario() {
      if (!this.usuarioTemp.trim()) return;
      this.usuario = this.usuarioTemp.trim();
      this.usuarioEscolhido = true;
    },

    async enviar() {
      if (!this.texto.trim()) return;

      await fetch("http://127.0.0.1:8000/api/mensagens", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          usuario: this.usuario,
          texto: this.texto,
        }),
      });

      this.texto = "";
    },
  },
};
</script>

<style scoped>
/* Tela toda moderna */
.chat-container {
  width: 100%;
  height: 100%;
  position: fixed;
  inset: 0;
  border-radius: 30px;
  overflow: hidden;
  backdrop-filter: blur(6px);
}
.chat-box {
  height: 80vh; /* ajuste a altura que quiser */
  max-height: 80vh;
  border-radius: 20px;
}

/* Header minimalista */
.header-app {
  position: absolute;
  top: 20px;
  left: 20px;
}
.header-app h5 {
  background: linear-gradient(135deg, #6366f1, #8b5cf6);
  padding: 10px 20px;
  border-radius: 15px;
  color: #fff;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

/* Efeito glassmorphism */
.glass {
  background: rgba(255, 255, 255, 0.45) !important;
  backdrop-filter: blur(12px);
}

/* Papel de fundo no chat */
.bg-paper {
  background-color:#f0dcf5;
  background-size: cover;
  background-repeat: repeat;
  filter: contrast(1.05);
}

/* Mensagens */
.msg-mine {
  margin-left: auto;
  background: #d1ffd6;
  border-radius: 20px 20px 0 20px;
  max-width: 75%;
}

.msg-other {
  margin-right: auto;
  background: #fff;
  border: 1px solid #eee;
  border-radius: 20px 20px 20px 0;
  max-width: 75%;
}

/* Área das mensagens */
.area-mensagens {
  overflow-y: auto;
  border-radius: 0 0 30px 30px;
}

/* Scroll bonito */
#areaMensagens::-webkit-scrollbar {
  width: 8px;
}

#areaMensagens::-webkit-scrollbar-thumb {
  background: rgba(0, 0, 0, 0.2);
  border-radius: 10px;
}

/* Input moderno */
.input-msg {
  border-radius: 25px !important;
  padding: 14px 18px;
}

/* Painel direito gradiente */
.painel-direito {
  background: linear-gradient(135deg, #6366f1, #8b5cf6, #ec4899);
  position: relative;
}

.painel-overlay {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.15);
}
</style>
