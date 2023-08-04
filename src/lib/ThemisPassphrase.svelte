<script>
  import { StringToUTF8Array } from "$lib/utf8"
  import { onMount } from "svelte"

  import Themis from "wasm-themis"
  import libthemis from "wasm-themis/dist/libthemis.wasm?url"

  let cell

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

  $: if (themisInitialized && passphrase) {
    cell = Themis.SecureCellSeal.withPassphrase(passphrase)
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

<h1>Themis (Secure Cell - Seal Mode with passphrase)</h1>

<div>Passphrase: <input bind:value={passphrase} /></div>
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
