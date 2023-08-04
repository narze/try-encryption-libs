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
  let elapsed = 0

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

    const start = performance.now()

    output = encrypt(obj, key)

    const end = performance.now()
    elapsed = end - start
  }

  function decryptMessage() {
    decrypted = ""
    // const obj = { msg: output }
    const start = performance.now()

    const obj = decrypt(output, key)
    decrypted = obj.msg

    const end = performance.now()
    elapsed = end - start
  }
</script>

<h1>TweetNaCl.js - Secretbox (Single key)</h1>

<div>
  <button on:click={newKey}>Generate new key</button>
  <input type="text" bind:value={key} />
</div>

<div>
  <span>Input</span>
  <input bind:value={input} />
</div>

<div>
  <button on:click={encryptMessage}>Encrypt Message</button>

  <span>Output</span>
  <input bind:value={output} />
</div>
<div>
  <button on:click={decryptMessage}>Decrypt Message</button>

  <span>Decrypt</span>
  <input bind:value={decrypted} />
</div>
<div>Time: {elapsed} ms</div>
