<script setup>
import { ref } from 'vue';

const logged = ref(false);
const email = ref('');
const password = ref('');
const username = ref('');
const user = ref();
const chatLog = ref('');
const input = ref('');
const channels = ref();
const channelId = ref('');
const chatSelected = ref(false);
const signUp = ref(false);

async function handleLogin() {
  try {
    const data = {
      email: email.value,
      password: password.value
    };

    const req = await fetch('http://localhost:3000/api/accounts/login', {
      method: 'POST',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data),
    });

    if (req.status >= 500) {
      console.log('server error');
    } else if (req.status >= 400) {
      console.log('bad request');
    } else {
      const res = await req.json();

      logged.value = true;
      user.value = res.user;

      await handleGetChannel();

      console.log(res);
      console.log('success');
    }

  } catch (error) {
    console.log(error);
  }
}

async function handleLogout() {
  try {
    const req = await fetch('http://localhost:3000/api/accounts/logout', {
      method: 'POST',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
    });

    if (req.status >= 500) {
      console.log('server error');
    } else if (req.status >= 400) {
      console.log('bad request');
    } else {
      logged.value = false;
      chatLog.value = '';
      chatSelected.value = false;
      console.log('success');
    }
  } catch (error) {
    console.log(error);
  }
}

async function handleGetChannel() {
  try {
    const req = await fetch('http://localhost:3000/api/channels/', {
      method: 'GET',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
    });

    if (req.status >= 500) {
      console.log('server error');
    } else if (req.status >= 400) {
      console.log('bad request');
    } else {
      const res = await req.json();
      channels.value = res.docs;
      console.log(res);
      console.log('success');
    }
  } catch (error) {
    console.log(error);
  }
}

async function handleFindChannel(channelID) {
  try {
    channelId.value = channelID;
    chatSelected.value = true;
    const req = await fetch(`http://localhost:3000/api/channels/${channelID}`, {
      method: 'GET',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
    });

    if (req.status >= 500) {
      console.log('server error');
    } else if (req.status >= 400) {
      console.log('bad request');
    } else {
      const res = await req.json();
      chatLog.value = res.chatHistory;
      console.log(res);
      console.log('success');
    }
  } catch (error) {
    console.log(error);
  }
}

async function handleSendText() {
  try {
    const newChatEntry = {
      sender: user.value.username,
      message: input.value,
    };

    const data = {
      chatHistory: [...chatLog.value, newChatEntry],
    };

    const req = await fetch(`http://localhost:3000/api/channels/${channelId.value}`, {
      method: 'PATCH',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data),
    });

    if (req.status >= 500) {
      console.log('server error');
    } else if (req.status >= 400) {
      console.log('bad request');
    } else {
      console.log('success');
      await handleFindChannel(channelId.value);
    }

  } catch (error) {
    console.log(error);
  }
}

async function handleCreateChannel() {
  try {
    const data = {
      authorID: user.value.id,
    };

    const req = await fetch(`http://localhost:3000/api/channels/`, {
      method: 'POST',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data),
    });

    if (req.status >= 500) {
      console.log('server error');
    } else if (req.status >= 400) {
      console.log('bad request');
    } else {
      console.log('success');
      await handleGetChannel();
    }

  } catch (error) {
    console.log(error);
  }
}

async function handleSignUp() {
  try {
    const data = {
      email: email.value,
      password: password.value,
      username: username.value
    };

    const req = await fetch('http://localhost:3000/api/accounts/', {
      method: 'POST',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data),
    });

    if (req.status >= 500) {
      console.log('server error');
    } else if (req.status >= 400) {
      console.log('bad request');
    } else {
      const res = await req.json();

      logged.value = true;
      user.value = res.doc;

      await handleGetChannel();

      console.log(res);
      console.log('success');
    }
  } catch (error) {
    console.log(error);
  }
}

function handleSignUpState() {
  signUp.value = !signUp.value;
}

</script>

<template>
  <div v-if="!logged">
    <div v-if="!signUp">
      <h1>Login</h1>
      <form @submit.prevent="">
        <label for="Email">Email </label>
        <input type="email" id="Email" v-model="email"><br>
        <label for="Password">Password </label>
        <input type="password" id="Password" v-model="password">
      </form>
      <button @click="handleLogin">Login</button>
      <button @click="handleSignUpState">goto sign-up</button>
    </div>
    <div v-if="signUp">
      <h1>Sign Up</h1>
      <form @submit.prevent="">
        <label for="Email">Email </label>
        <input type="email" id="Email" v-model="email"><br>
        <label for="Password">Password </label>
        <input type="password" id="Password" v-model="password"><br>
        <label for="Username">Username </label>
        <input type="username" id="Username" v-model="username">
      </form>
      <button @click="handleSignUp">Sign Up</button>
      <button @click="handleSignUpState">goto login</button>
    </div>
  </div>

  <div v-if="logged">
    <h1>Chat App</h1>

    <div>
      <h1>Your Profile</h1>
      <p>ID: {{ user.id }}</p>
      <p>Username: {{ user.username }}</p>
      <p>Email: {{ user.email }}</p>
      <button @click="handleLogout">Logout</button>
    </div>

    <h1>Available Channel</h1>
    <button @click="handleCreateChannel">create new channel</button>
    <div v-for="channel in channels" :key="channel.id">
      <p>channel {{ channel.id }}</p>
      <button @click="handleFindChannel(channel.id)">join</button>
    </div>

    <div v-if="chatSelected">
      <h1>Chat log</h1>
      <div v-for="log in chatLog" :key="log.id">
        <p>From {{ log.sender }}: {{ log.message }}</p>
      </div>
      <input type="text" v-model="input">
      <button @click="handleSendText">send</button>
    </div>

  </div>
</template>

<style scoped></style>