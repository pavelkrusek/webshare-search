<script setup>
import { ref } from 'vue'

const searchTerm = ref('')
const files = ref([])
const loading = ref(false)

const searchFiles = async () => {
  loading.value = true
  try {
    const response = await fetch('https://webshare.cz/api/search/', {
      method: 'POST',
      headers: {
        Accept: 'text/xml; charset=UTF-8',
        'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
      },
      body: `what=${encodeURIComponent(searchTerm.value)}&limit=100&offset=0&sort=largest&category=video`
    })

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    const xmlText = await response.text()
    const parser = new DOMParser()
    const xmlDoc = parser.parseFromString(xmlText, 'text/xml')
    const fileNodes = xmlDoc.getElementsByTagName('file')

    files.value = Array.from(fileNodes).map((fileNode) => {
      const name = fileNode.getElementsByTagName('name')[0]?.textContent || 'Unknown'
      const size = parseInt(fileNode.getElementsByTagName('size')[0]?.textContent || '0')
      const ident = fileNode.getElementsByTagName('ident')[0]?.textContent || ''

      let sizeStr
      if (size >= 1073741824) {
        sizeStr = `${(size / 1073741824).toFixed(2)}GB`
      } else if (size >= 1048576) {
        sizeStr = `${(size / 1048576).toFixed(2)}MB`
      } else if (size >= 1024) {
        sizeStr = `${(size / 1024).toFixed(2)}KB`
      } else {
        sizeStr = `${size}B`
      }

      return {
        name,
        sizeStr,
        ident,
        link: `https://webshare.cz/#/file/${ident}`
      }
    })
  } catch (error) {
    console.error('Error searching files:', error)
    files.value = []
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div id="main">
    <h1>webshare.cz hledani</h1>
    <input v-model="searchTerm" @keyup.enter="searchFiles" placeholder="Hledej film/serial" />
    <button @click="searchFiles">Hledej</button>
    <div v-if="loading">Nahravam...</div>
    <ul v-else>
      <li v-for="file in files" :key="file.ident">
        {{ file.name }} | {{ file.sizeStr }} | <a :href="file.link" target="_blank">Download</a>
      </li>
    </ul>
  </div>
</template>

<style scoped>
#app {
  font-family: Arial, sans-serif;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

input,
button {
  margin-bottom: 20px;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  margin-bottom: 10px;
}
</style>
