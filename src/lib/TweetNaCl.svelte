<script lang="js">
  import { secretbox, randomBytes } from "tweetnacl"
  import {
    decodeUTF8,
    encodeUTF8,
    encodeBase64,
    decodeBase64,
  } from "tweetnacl-util"

  const newNonce = () => randomBytes(secretbox.nonceLength)
  const newKey = () => (key = generateKey())

  let key = ""

  export const generateKey = () =>
    encodeBase64(randomBytes(secretbox.keyLength))

  export const encrypt = (json, key) => {
    const keyUint8Array = decodeBase64(key)

    const nonce = newNonce()
    const messageUint8 = decodeUTF8(JSON.stringify(json))
    const box = secretbox(messageUint8, nonce, keyUint8Array)

    const fullMessage = new Uint8Array(nonce.length + box.length)
    fullMessage.set(nonce)
    fullMessage.set(box, nonce.length)

    const base64FullMessage = encodeBase64(fullMessage)
    return base64FullMessage
  }

  export const decrypt = (messageWithNonce, key) => {
    const keyUint8Array = decodeBase64(key)
    const messageWithNonceAsUint8Array = decodeBase64(messageWithNonce)
    const nonce = messageWithNonceAsUint8Array.slice(0, secretbox.nonceLength)
    const message = messageWithNonceAsUint8Array.slice(
      secretbox.nonceLength,
      messageWithNonce.length
    )

    const decrypted = secretbox.open(message, nonce, keyUint8Array)

    if (!decrypted) {
      throw new Error("Could not decrypt message")
    }

    const base64DecryptedMessage = encodeUTF8(decrypted)
    return JSON.parse(base64DecryptedMessage)
  }

  let input = "Hello world"
  let output = ""
  let decrypted = ""

  function encryptMessage() {
    output = ""
    const obj = { msg: input }
    output = encrypt(obj, key)
  }

  function decryptMessage() {
    decrypted = ""
    // const obj = { msg: output }
    const obj = decrypt(output, key)
    decrypted = obj.msg
  }
</script>

<h1>TweetNaCl.js - Secretbox (Single key)</h1>

<input type="text" bind:value={key} />
<button on:click={newKey}>Generate new key</button>

<span>Input</span>
<textarea name="" id="" cols="30" rows="10" bind:value={input} />

<button on:click={encryptMessage}>Encrypt Message</button>

<span>Output</span>
<textarea name="" id="" cols="30" rows="10" bind:value={output} />

<button on:click={decryptMessage}>Decrypt Message</button>

<span>Decrypt</span>
<textarea name="" id="" cols="30" rows="10" bind:value={decrypted} />
