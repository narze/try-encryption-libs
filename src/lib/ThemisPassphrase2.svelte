<script>
  import { StringToUTF8Array } from "$lib/utf8"
  import { onMount } from "svelte"

  import Themis from "wasm-themis"
  import libthemis from "wasm-themis/dist/libthemis.wasm?url"
  import { pbkdf2Async } from "@noble/hashes/pbkdf2"

  let cell, symmetricKey

  let input = "Hello world"
  let passphrase = "password"
  let themisInitialized = false
  let encrypted = ""
  let decrypted = ""
  let elapsed = 0

  onMount(() => {
    Themis.initialize(libthemis).then(() => {
      themisInitialized = true
    })
  })

  $: if (themisInitialized && symmetricKey) {
    cell = Themis.SecureCellSeal.withKey(symmetricKey)
  }

  import { sha256 } from "@noble/hashes/sha256"

  async function generateKey() {
    const start = performance.now()

    symmetricKey = await pbkdf2Async(sha256, passphrase, "salt", {
      c: 314110,
      dkLen: 32,
    })

    const end = performance.now()

    elapsed = end - start
  }

  function encryptMessage() {
    encrypted = ""

    const inputAsArray = StringToUTF8Array(input)
    const start = performance.now()
    encrypted = cell.encrypt(inputAsArray)
    const end = performance.now()
    elapsed = end - start
  }

  function decryptMessage() {
    decrypted = ""

    const start = performance.now()
    const decryptedAsArray = cell.decrypt(encrypted)
    const end = performance.now()
    elapsed = end - start

    decrypted = new TextDecoder().decode(decryptedAsArray)
  }
</script>

<h1>
  Themis (Secure Cell - Seal Mode with passphrase with 3rd party PBKDF2 library)
</h1>

<div>
  Passphrase: <input bind:value={passphrase} />
  <button on:click={generateKey}>Generate Key</button>
</div>

<div>
  Symmetric key:<input bind:value={symmetricKey} />
</div>
<div>
  Input: <input bind:value={input} />
</div>
<div>
  <button on:click={encryptMessage}>Encrypt</button>
  Encrypted:
  <input bind:value={encrypted} />
</div>
<div>
  <button on:click={decryptMessage}>Decrypt</button> Decrypted:
  <input bind:value={decrypted} />
</div>

<div>Time: {elapsed} ms</div>
