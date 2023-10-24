<script setup lang="ts">
import { ref } from "vue";

defineProps<{ msg: string }>();

const count = ref(0);
const checkCredentialAvailable = () => {
  // Availability of `window.PublicKeyCredential` means WebAuthn is usable.
  // `isUserVerifyingPlatformAuthenticatorAvailable` means the feature detection is usable.
  // `​​isConditionalMediationAvailable` means the feature detection is usable.
  if (
    window.PublicKeyCredential &&
    PublicKeyCredential.isUserVerifyingPlatformAuthenticatorAvailable &&
    PublicKeyCredential.isConditionalMediationAvailable
  ) {
    // Check if user verifying platform authenticator is available.
    return Promise.all([
      PublicKeyCredential.isUserVerifyingPlatformAuthenticatorAvailable(),
      PublicKeyCredential.isConditionalMediationAvailable(),
    ]).then((results) => {
      if (results.every((r) => r === true)) {
        return true;
      } else {
        return false;
      }
    });
  } else {
    return false;
  }
};
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
const passkeyLogin = async () => {
  console.info("login by passkey");
  let challenge = fetchChallenge();
  let options: PublicKeyCredentialRequestOptions = {
    challenge: Uint8Array.from(challenge, (c) => c.charCodeAt(0)),
    // allowCredentials: [
    //   {
    //     id: credentialId, // Uint8Array.from(credentialId, (c) => c.charCodeAt(0)),
    //     type: "public-key",
    //     transports: ["hybrid", "internal", "usb", "ble", "nfc"],
    //   },
    // ],
    timeout: 60000,
  };
  try {
    let credential = await navigator.credentials.get({
      publicKey: options,
    });
    let assertion = (credential as PublicKeyCredential)
      .response as AuthenticatorAssertionResponse;
    var clientDataJSON = JSON.parse(
      new TextDecoder("UTF-8").decode(assertion.clientDataJSON)
    );
    clientDataJSON.challenge = atob(clientDataJSON.challenge);
    console.info({
      assertion: assertion,
      clientDataJSON: clientDataJSON,
      signature: buf2hex(assertion.signature),
      userHandle: new TextDecoder("UTF-8").decode(assertion.userHandle ?? new ArrayBuffer(0)),
    });


    // Create a DataView to work with the ArrayBuffer
    const dataView = new DataView(assertion.authenticatorData);

    // Parse the authenticatorData fields
    const rpIdHash = new Uint8Array(assertion.authenticatorData.slice(0, 32)); // RP ID Hash
    const flags = dataView.getUint8(32); // Flags
    const signatureCounter = dataView.getUint32(33, false); // Signature Counter

    // If there's an attestation data, you can extract it as well
    const attestationData = assertion.authenticatorData.slice(37); // Attestation Data

    console.log('RP ID Hash:', rpIdHash);
    console.log('Flags:', flags);
    console.log('Signature Counter:', signatureCounter);
    console.log('Attestation Data:', attestationData);

  } catch (error) {
    console.warn(
      "the error caught when getting login credential",
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
