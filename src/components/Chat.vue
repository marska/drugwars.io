<template>
	<div>
				<div class="black p-0">
					<button class="button button-black">Main</button>
					<button class="button button-black">
						Gang
						<span class="text-red">0</span>
					</button>
					<button class="button button-black">
						Private
						<span class="text-red">0</span>
					</button>
					<button class="button button-black">Users 		<span class="text-red">{{members.length}}</span></button>
				</div>
		<div class="chat">
			<div :key="response.id" v-for="response in responses" class="border-bottom">
				<div class="text-left item width-full">
					<div class="columns">
					<div class="column pl-1 col-3">
										<router-link
					v-if="response.sender != user.nickname"
					:to="`/fight?target=${response.sender}`">
					<Avatar :size="40" :username="response.sender" :picture="response.picture"/>
						</router-link>
											<Avatar v-else :size="40" :username="response.sender" :picture="response.picture"/>
						<h5 class="mt-1 sender">{{response.sender}} {{response.gang}}</h5>
					</div>
						<div class="column col-7 message text-left" v-if="response.text" v-html="checkUrl(response.text)"></div>
											<div class="column date pr-0 col-1">{{response.date}}</div>
					</div>
				</div>
			</div>
		</div>
		<div class="p-4 reply text-center">
			<Loading v-if="isLoading"/>
			<div v-else>
				<div class="columns">
					<h5>your message</h5>
										<div class="column pl-0 col-10">
					<textarea
						type="text"
						class="input input-block mb-2"
						placeholder="Write here."
						v-model="message" v-on:keyup.13="handleSubmit"
						maxlength="280"
					></textarea>
					</div>
					<div class="column pr-0 col-2">
					<button
						@click="handleSubmit()"
						class="button button-green btn-block mb-2"
						:disabled="isLoading"
					>Send</button>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import { mapActions } from 'vuex';
import Promise from 'bluebird';
import client from '@/helpers/client';
import io from 'socket.io-client';

if (socket) socket.disconnect();
const socket = new io.connect('https://drugwars-chat.herokuapp.com/');
// const socket = new io.connect('http://localhost:8082');

export default {
  data() {
    return {
      id: this.$route.params.id,
      isInit: false,
      isLoading: false,
      gang: null,
      members: [],
      message: null,
      user: this.$store.state.game.user.user,
      applies: null,
      info: null,
      responses: [],
    };
  },
  created() {
    const token = localStorage.getItem('access_token');
    const self = this;
    socket.emit('add-user', { token });
    socket.on('read-msg', message => {
      self.responses.push({
        text: message.text,
        sender: message.sender,
        gang: message.gang,
        picture: message.picture,
        date: message.date,
      });
    });

    socket.on('user-connected', userId => {
      self.members.push(userId);
    });

    socket.on('init-chat', messages => {
      self.responses = messages;
    });

    socket.on('update-chat', messages => {
      self.responses = messages;
    });

    socket.on('blocked', messages => {
      self.responses = messages;
      self.responses.push({
        text: 'Please wait before sending another message',
        sender: 'DrugWars',
        gang: 'government',
        picture: message.picture,
        date: new Date().toString(),
      });
    });

    socket.on('update-users', users => {
      self.members = users;
    });
  },
  updated() {
    this.scrollToEnd();
  },
  methods: {
    ...mapActions(['init', 'notify', 'refresh_gang_buildings']),
    handleSubmit() {
      if (this.message.length > 0) {
        this.isLoading = true;
        const self = this;
        const message = this.message;
        const token = localStorage.getItem('access_token');
        socket.emit('send-msg', {
          token,
          message,
        });
        this.message = null;
        this.isLoading = false;
      }
    },
    scrollToEnd() {
      const container = this.$el.querySelector('.chat');
      container.scrollTop = container.scrollHeight;
    },
    checkUrl(url) {
      try {
        url = url.replace(
          /(https?:\/\/.*?\.(?:png|jpe?g|gif)(.*))(\w|$)/gi,
          "<br><img width='100%' src='$1'>",
        );
        return url;
      } catch (e) {
        return url;
      }
    },
  },
};
</script>

<style scoped lang="less">
@import '../vars.less';

.message {
  color: whitesmoke;
  overflow-wrap: break-word;
}

img {
  max-width: 100% !important;
}
.chat {
  overflow-y: scroll;
  overflow-x: hidden;
  height: calc(100vh - 360px);
  width: 100%;
}

.sender {
  padding-top: 10px;
  padding-bottom: 0px;
  left: 20px !important;
  font-size: 14px;
}

.reply {
  bottom: 0px;
}

.black {
  background: black;
  color: gray !important;
}
.black button {
  background: black;
  color: rgb(51, 51, 51) !important;
}
</style>
