<script setup>
import { ref } from 'vue'

const mostrar3D = ref(false)
</script>

<template>
  <div style="text-align: center; padding: 50px;">
    <h1>Explorador de Modelos 3D</h1>
    <p>Haz clic en el botón para cargar el diseño técnico (Lazy Loading).</p>

    <button 
      v-if="!mostrar3D" 
      @click="mostrar3D = true"
      style="padding: 10px 20px; font-size: 1.1rem; cursor: pointer; background: #333; color: white; border: none; border-radius: 5px;"
    >
      Cargar Modelo 3D
    </button>

    <LazyVisor3D v-if="mostrar3D" />

  </div>
</template>
