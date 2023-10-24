<script setup lang="ts">
import { ref } from "vue";

defineProps<{ msg: string }>();

const count = ref(0);
const passkeyRegister = async () => {
  console.info("register by passkey");
  let username = prompt("Enter your username", "username");
  if (username == null) return;
  let challenge = fetchChallenge();
  let options: PublicKeyCredentialCreationOptions = {
    challenge: Uint8Array.from(challenge, (c) => c.charCodeAt(0)),
    rp: {
      name: "Passkey Demo",
      id: "localhost",
    },
    user: {
      id: Uint8Array.from("UZSL85T9AFC", (c) => c.charCodeAt(0)),
      name: username,
      displayName: "@" + username,
    },
    pubKeyCredParams: [
      { alg: -7, type: "public-key" },
      // { alg: -257, type: "public-key" },
    ],
    excludeCredentials: [
      {
        id: Uint8Array.from("UZSL85T9AFC", (c) => c.charCodeAt(0)),
        type: "public-key",
        transports: ["internal", "usb", "ble", "nfc"],
      },
    ],
    authenticatorSelection: {
      // authenticatorAttachment: "platform", // not providing for all options
      requireResidentKey: true,
    },
    attestation: "direct",
  };

  try {
    let credential = await navigator.credentials.create({
      publicKey: options,
    });

    let attestation = (credential as PublicKeyCredential)
      .response as AuthenticatorAttestationResponse;
    let clientDataJSON = attestation.clientDataJSON;
    let json = JSON.parse(new TextDecoder("UTF-8").decode(clientDataJSON));
    let challengeInCredential = atob(json.challenge);

    console.info({
      attestation: attestation,
      json: json,
      challenge: challenge,
      challengeInCredential: challengeInCredential,
    });
  } catch (error) {
    console.warn(
      "the error caught in creation of registration credential",
      error,
      (error as Error).message
    );
  }
};
const fetchChallenge = () => {
  let random = Math.round(Math.random() * 10000);
  return "shanelic" + random;
};
const buf2hex = (buffer: ArrayBuffer) => { // buffer is an ArrayBuffer
  return [...new Uint8Array(buffer)]
      .map(x => x.toString(16).padStart(2, '0'))
      .join('');
}
</script>

<template>
  <h1>{{ msg }}</h1>

  <div class="card">
    <button type="button" @click="count++">count is {{ count }}</button>
    <p>
      Edit
      <code>components/HelloWorld.vue</code> to test HMR
    </p>
  </div>

  <p>
    Check out
    <a href="https://vuejs.org/guide/quick-start.html#local" target="_blank"
      >create-vue</a
    >, the official Vue + Vite starter
  </p>
  <p>
    Install
    <a href="https://github.com/vuejs/language-tools" target="_blank">Volar</a>
    in your IDE for a better DX
  </p>
  <p class="read-the-docs">Click on the Vite and Vue logos to learn more</p>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
