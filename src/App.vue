<template>
  <div id="app">
   <div v-if="initiated" class="video" ref="video" />
   <button v-if="!initiated" @click="init">asdf</button>
  </div>
</template>

<script>
import uneeq from 'uneeq-js';

export default {
  name: 'App',

  data() {
    return {
      uneeq: null,
      token: null,
      initiated: false
    }
  },

  methods: {
    async init() {
       const response = await fetch('https://em.staging.api.onereach.ai/http/668917f5-beec-4534-a7b4-b59ea7cca124/uneeq_jwt', {
        method: "POST"
      });

      const { token } = await response.json();

      this.token = token;
      this.initiated = true;
      await this.$nextTick();

      console.log('this.$refs.video', this.$refs.video)

      this.uneeq = new Uneeq({
        url: 'https://api.us.uneeq.io',
        conversationId: 'c5c2c432-874f-4d90-9ff2-f96d213cad63',
        avatarVideoContainerElement: this.$refs.video,
        sendLocalVideo: false,
        sendLocalAudio: false,
      });
      await this.uneeq.initWithToken(token);

      window.addEventListener('message', this.onMessage, false);
    },

    onMessage(e) {
      const { type, text } = e.data;

      if(type !== 'speak') return;

      fetch(`https://em.staging.api.onereach.ai/http/668917f5-beec-4534-a7b4-b59ea7cca124/uneeq_conversation_platform?sessionId=${this.uneeq.sessionId}&text=${text}`);
    },
  }
}
</script>

<style>
body, html {
  margin: 0;
}

.video,
#app {
  width: 100%;
  height: 100vh;
}
</style>
