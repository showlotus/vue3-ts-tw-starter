<script setup lang="ts">
import HelloWorld from '@/components/HelloWorld.vue'
import { useStore } from '@/store'

const store = useStore()
</script>
<template>
  <header>
    <h1>Vite + Vue 3 + TypeScript + Tailwind Starter Template v{{ store.appMeta.version }}</h1>
    <p>
      Opinionated, production ready template for Vite and Vue 3. MIT licensed,
      <a href="https://github.com/Uninen/vite-ts-tailwind-starter">available on GitHub</a>.
    </p>
  </header>

  <main>
    <HelloWorld msg="Hello World Component" />

    <h2>Project setup and usage</h2>

    <h4>Install dependencies</h4>

    <pre>pnpm i</pre>

    <h4>Run development server</h4>

    <pre>pnpm dev</pre>

    <h4>Run unit + component tests (Vitest)</h4>

    <pre>pnpm test</pre>

    <h4>Build for production</h4>

    <pre>pnpm build</pre>

    <h4>Other</h4>

    <p>See <code>package.json</code> for all available commands.</p>

    <h2>Notes and further documentation</h2>

    <h3>Typed ENV Variables</h3>

    <p>
      Vite exposes a special <code>meta.env</code> object for ENV variables (see
      <a href="https://vitejs.dev/guide/env-and-mode.html">official docs</a>). This template extends that object and
      adds custom typed variables which you can easily use and modify to your needs.
    </p>

    <p>
      See <code>vite.config.ts</code> and <code>src/env.d.ts</code> for the configuration and
      <code>src/App.vue</code> for usage example.
    </p>
  </main>
</template>
