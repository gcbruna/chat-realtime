<template>
  <div class="chat-container">
    <div class="row g-0 h-100">

      <!-- Lado esquerdo: Chat dentro do celular -->
      <div class="col-md-6 d-flex align-items-center justify-content-center bg-light h-100 chat-left">

        <!-- Mockup do celular -->
        <div class="cellphone-frame shadow-lg">
          <div class="cellphone-screen">

            <!-- Tela de escolher usuário -->
            <div v-if="!usuarioEscolhido"
              class="d-flex flex-column p-4 align-items-center justify-content-center flex-fill">

              <h3 class="fw-bold mb-4">Escolha seu nome</h3>

              <input v-model="usuarioTemp" placeholder="Seu nome..."
                class="form-control form-control-lg input-nome mb-3" />

              <button class="btn btn-outline-primary btn-lg w-100 botao-entrar" @click="definirUsuario">Entrar</button>
            </div>

            <!-- Chat -->
            <div v-else class="d-flex flex-column h-100">

              <div id="areaMensagens" class="area-mensagens flex-grow-1 p-3 bg-paper">
                <div v-for="(m, i) in mensagens" :key="i" :class="[
                  'p-2 mb-2 rounded message-bubble',
                  m.usuario === usuario ? 'msg-mine' : 'msg-other'
                ]">
                  <small class="fw-bold" v-if="m.usuario !== usuario">{{ m.usuario }}</small><br />
                  {{ m.texto }}
                </div>
              </div>

             <!-- INPUT -->
              <div class="area-input glass p-2 mb-2 m-1 w-90">
                <div class="input-group">
                  <input v-model="texto" @keyup.enter="enviar" placeholder="Digite uma mensagem..." class="form-control input-msg">
                  
                  <button class="btn btn-primary rounded-circle d-flex m-1 align-items-center justify-content-center" @click="enviar">
                    <i class="bi bi-cursor-fill fs-5"></i>
                  </button>
                </div>
              </div>

            </div>

          </div>
        </div>
      </div>

      <!-- Painel direito -->
      <div
        class="col-md-6 d-flex flex-column align-items-center justify-content-center text-white text-center position-relative painel-direito">
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
    this.ably = new Ably.Realtime(
      "WA9TGA.nuLYhw:kYidV4M_-rg5drc1oRmo3lOdKwbxD9Noux8CzwGbkys"
    );

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
/* ===========================
   CONTAINER GERAL
=========================== */
.chat-container {
  width: 100%;
  height: 100%;
  position: fixed;
  inset: 0;
  border-radius: 30px;
  overflow: hidden;
  backdrop-filter: blur(6px);
}

/* ===========================
   CHAT PAPER BACKGROUND
=========================== */
.bg-paper {
  background-color: #ecf0fc;
  background-size: cover;
  background-repeat: repeat;
  filter: contrast(1.05);
}

/* ===========================
   MENSAGENS
=========================== */
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

/* Força a área de mensagens a ocupar todo o espaço disponível */
.area-mensagens {
  flex: 1 !important;
  height: auto !important;
  min-height: 0 !important;
  /* crucial para flex funcionar */
}

/* Ajuste do container interno do celular */
.cellphone-screen {
  display: flex;
  flex-direction: column;
  height: 100%;
}


/* Scroll bonito */
#areaMensagens::-webkit-scrollbar {
  width: 6px;
}

#areaMensagens::-webkit-scrollbar-thumb {
  background: rgba(0, 0, 0, 0.25);
  border-radius: 10px;
}

/* ============================================
   INPUT
============================================ */
.area-input {
  position: absolute;
  bottom: 1rem;
  left: 50%;
  transform: translateX(-50%);
  width: 94%;
}

.glass {
  background: rgba(255, 255, 255, 0.45);
  backdrop-filter: blur(10px);
  border-radius: 35px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.input-msg {
  border-radius: 25px !important;
}

/* ===========================
   MOCKUP iPHONE X
=========================== */
.cellphone-frame {
  width: 390px;
  height: 800px;
  background: #000;
  border-radius: 55px;
  padding: 22px;
  position: relative;
  box-shadow: 0 0 40px rgba(0, 0, 0, 0.45);
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Notch superior */
.cellphone-frame::before {
  content: "";
  position: absolute;
  top: 18px;
  left: 50%;
  transform: translateX(-50%);
  width: 210px;
  height: 32px;
  background: #000;
  border-bottom-left-radius: 18px;
  border-bottom-right-radius: 18px;
  z-index: 10;
}

/* Barrinha inferior estilo iPhone */
.cellphone-frame::after {
  content: "";
  position: absolute;
  bottom: 27px;
  left: 50%;
  transform: translateX(-50%);
  width: 120px;
  height: 5px;
  background: #333;
  border-radius: 3px;
  opacity: 0.8;
  z-index: 10;
}

.cellphone-screen {
  background: #ffffff;
  width: 100%;
  height: 100%;
  border-radius: 45px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  position: relative;
  z-index: 2;
}

/* Responsivo */
@media(max-width: 768px) {
  .cellphone-frame {
    width: 95vw;
    height: 90vh;
    border-radius: 45px;
  }

  .cellphone-frame::before {
    width: 45%;
  }

  /* Ajuste da barra inferior para mobile */
  .cellphone-frame::after {
    width: 30%;
  }

  .cellphone-screen {
    border-radius: 38px;
  }
}


/* Responsivo */
@media(max-width: 768px) {
  .cellphone-frame {
    width: 95vw;
    height: 90vh;
    border-radius: 45px;
  }

  .cellphone-frame::before {
    width: 45%;
  }

  .cellphone-screen {
    border-radius: 38px;
  }
}

/* ===========================
   PAINEL DIREITO
=========================== */
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
