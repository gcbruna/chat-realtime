<template>
  <div class="chat-container">
    <div class="row g-0 h-100">

      <!-- HEADER SUPERIOR -->
      <div class="top-header d-flex align-items-center justify-content-between px-4 py-3 w-100">
        <div class="d-flex align-items-center gap-3">
          <img v-if="me.avatar" :src="me.avatar" class="rounded-circle" width="36" height="36" />
          <strong>{{ me.name || "Visitante" }}</strong>
          <small v-if="connected" class="text-success">• conectado</small>
          <small v-else class="text-muted">• desconectado</small>
        </div>

        <div class="d-flex align-items-center gap-3">
          <i class="bi bi-bell-fill bell-icon-header" @click="requestNotificationPermission"></i>
          <button v-if="me.name" class="btn btn-outline-secondary btn-sm" @click="logout">Sair</button>
        </div>
      </div>

      <!-- LADO ESQUERDO (Chat dentro do celular) -->
      <div class="col-md-6 d-flex align-items-center justify-content-center bg-light h-100 chat-left">
        <div class="cellphone-frame shadow-lg">
          <div class="cellphone-screen">

            <!-- TELA DE ESCOLHER NOME -->
            <div v-if="!me.name" class="d-flex flex-column p-4 align-items-center justify-content-center flex-fill">
              <h3 class="fw-bold mb-4">Escolha seu nome</h3>
              <input v-model="tempName" placeholder="Seu nome..." class="form-control form-control-lg input-nome mb-3" />
              <div class="d-flex gap-2 w-100">
                <button class="btn btn-outline-primary btn-lg w-100" @click="enterAs(tempName)">Entrar</button>
                <button class="btn btn-outline-secondary btn-lg" @click="enterAs(guestName)">Visitante</button>
              </div>
            </div>

            <!-- TELA DE CHAT -->
            <div v-else class="d-flex h-100">

              <!-- LISTA DE USUÁRIOS -->
              <div class="users-list p-3 border-end" style="width: 36%;">
                <h6 class="mb-2">Contatos</h6>
                <div v-if="loadingUsers" class="text-muted">Carregando...</div>
                <div v-else>
                  <div v-for="u in users" :key="u.id"
                       class="d-flex align-items-center justify-content-between py-2 user-row"
                       :class="{ active: currentPeer && currentPeer.id === u.id }" @click="openChatWith(u)">
                    <div class="d-flex align-items-center gap-2 user-item">
                      <img :src="u.avatar" class="rounded-circle" width="36" height="36" />
                      <div>
                        <div class="fw-bold">{{ u.name }}</div>
                        <small class="text-muted">{{ u.presence }}</small>
                      </div>
                    </div>
                    <span v-if="unread[u.id]" class="badge bg-danger">{{ unread[u.id] }}</span>
                  </div>
                  <div v-if="users.length === 0" class="text-muted mt-3">Nenhum usuário online</div>
                </div>
              </div>

              <!-- JANELA DE MENSAGENS -->
              <div class="d-flex flex-column flex-fill">
                <div class="chat-header d-flex align-items-center gap-2 p-2 border-bottom">
                  <img :src="currentPeer ? currentPeer.avatar : defaultAvatar" class="rounded-circle" width="44" height="44" />
                  <div>
                    <h6 class="fw-bold m-0">{{ currentPeer ? currentPeer.name : "Selecione um usuário" }}</h6>
                    <small v-if="currentPeer">{{ currentPeer.presence }}</small>
                  </div>
                </div>

                <div id="areaMensagens" class="area-mensagens flex-grow-1 p-3 bg-paper overflow-auto">
                  <div v-for="(m, idx) in mensagens" :key="m.id || idx" class="d-flex align-items-start mb-3"
                       :class="{ 'justify-content-end': m.usuario === me.name }">
                    <div v-if="m.usuario !== me.name" class="avatar me-2" :style="{ backgroundColor: corDoNome(m.usuario) }">
                      {{ m.usuario.charAt(0).toUpperCase() }}
                    </div>
                    <div :class="['p-2 rounded message-bubble', m.usuario === me.name ? 'msg-mine' : 'msg-other']" style="max-width: 75%;">
                      <div v-if="m.usuario !== me.name" class="d-flex justify-content-between mb-1">
                        <small class="fw-bold">{{ m.usuario }}</small>
                        <small class="text-muted">{{ m.hora }}</small>
                      </div>
                      <div v-else class="text-end">
                        <small class="text-muted">{{ m.hora }}</small>
                      </div>
                      <div>{{ m.texto }}</div>
                    </div>
                  </div>
                </div>

                <!-- INPUT -->
                <div class="area-input glass p-2 mb-2 m-1 w-90">
                  <div class="input-group">
                    <input v-model="texto" @keydown.enter.prevent="enviar" placeholder="Digite uma mensagem..." class="form-control input-msg" />
                    <button class="btn btn-primary rounded-circle d-flex m-1 align-items-center justify-content-center" @click="enviar">
                      <i class="bi bi-cursor-fill fs-5"></i>
                    </button>
                  </div>
                </div>
              </div>

            </div>

          </div>
        </div>
      </div>

      <!-- PAINEL DIREITO -->
      <div class="col-md-6 d-flex flex-column align-items-center justify-content-center text-white text-center position-relative painel-direito">
        <div class="painel-overlay"></div>
        <div class="position-relative px-5">
          <h1 class="fw-bold display-5 mb-3">Chat Privado</h1>
          <p class="fs-5 opacity-75">Escolha alguém para conversar em tempo real.</p>
          <p class="small opacity-75 mt-3">Conectado ao Ably.</p>
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
      texto: "",
      tempName: "",
      guestName: "Visitante",
      me: { id: null, name: null, avatar: null },
      connected: false,
      loadingUsers: true,
      ably: null,
      lobbyChannel: null,
      users: [],
      unread: {},
      mensagens: [],
      currentPeer: null,
      defaultAvatar: "https://i.pravatar.cc/100",
    };
  },

  methods: {
    async enterAs(name) {
      if (!name || name.trim() === "") return;
      this.me.name = name;
      await this.initAbly(name);
    },

    async initAbly(name) {
      try {
        this.ably = new Ably.Realtime({
          clientId: name,
          authUrl: `http://127.0.0.1:8000/api/ably-token?name=${encodeURIComponent(name)}`
        });

        this.ably.connection.on('connected', () => {
          this.connected = true;
          this.me.id = name;
          this.me.avatar = `https://ui-avatars.com/api/?background=random&name=${encodeURIComponent(name)}`;
          this.initPresence();
        });
      } catch (e) {
        console.error('Erro ao iniciar Ably:', e);
      }
    },

    async initPresence() {
      this.lobbyChannel = this.ably.channels.get('lobby-presence');

      // Entra no presence
      this.lobbyChannel.presence.enter({
        name: this.me.name,
        avatar: this.me.avatar
      });

      // Inscreve eventos de presença
      this.lobbyChannel.presence.subscribe('enter', () => this.updateUserList());
      this.lobbyChannel.presence.subscribe('update', () => this.updateUserList());
      this.lobbyChannel.presence.subscribe('leave', () => this.updateUserList());

      // Atualiza lista inicialmente
      const pres = await this.lobbyChannel.presence.get();
      this.processPresenceList(pres);
      this.loadingUsers = false;
    },

    updateUserList() {
      this.lobbyChannel.presence.get().then((list) => this.processPresenceList(list));
    },

    processPresenceList(list) {
      this.users = list
        .filter(p => p.clientId !== this.me.id)
        .map(p => ({
          id: p.clientId,
          name: p.data?.name || p.clientId,
          avatar: p.data?.avatar || this.defaultAvatar,
          presence: 'online'
        }));
    },

    openChatWith(user) {
      this.currentPeer = user;
      this.mensagens = [];
    },

    enviar() {
      if (!this.texto.trim()) return;
      this.mensagens.push({
        usuario: this.me.name,
        texto: this.texto,
        hora: new Date().toLocaleTimeString("pt-BR", { hour: "2-digit", minute: "2-digit" }),
      });
      this.texto = "";
    },

    corDoNome(nome) {
      const cores = ["#ff7675", "#74b9ff", "#55efc4", "#ffeaa7", "#a29bfe"];
      let sum = 0;
      for (let i = 0; i < nome.length; i++) sum += nome.charCodeAt(i);
      return cores[sum % cores.length];
    },

    logout() {
      location.reload();
    },
  },
};
</script>

<style scoped>
.chat-container { width:100%; height:100%; position:fixed; inset:0; background:#f0f2f5; }
.bg-paper { background: linear-gradient(180deg, #f7f9fc, #edf0f7); }
.cellphone-frame { width:420px; height:830px; background:#000; border-radius:34px; padding:18px; box-shadow:0px 12px 40px rgba(0,0,0,0.35);}
.cellphone-screen { background:#fff; border-radius:26px; height:100%; position:relative; overflow:hidden; border:1px solid #dcdcdc; }
.msg-mine { background:#d3ffe0; border-radius:18px 18px 0 18px; }
.msg-other { background:#fff; border:1px solid #eee; border-radius:18px 18px 18px 0; }
.area-mensagens { padding:15px 18px; }
.area-input { position:absolute; bottom:12px; left:50%; transform:translateX(-50%); width:92%; background:rgba(255,255,255,0.8); backdrop-filter:blur(10px); border-radius:30px; padding:10px 12px; box-shadow:0px 5px 18px rgba(0,0,0,0.15);}
.input-msg { background:transparent!important; border:none!important; box-shadow:none!important;}
.bell-icon-header { font-size:22px; cursor:pointer; }
.painel-direito { background: linear-gradient(135deg, #6366f1, #8b5cf6, #ec4899);}
.painel-overlay { position:absolute; inset:0; background: rgba(0,0,0,0.2); }
</style>
