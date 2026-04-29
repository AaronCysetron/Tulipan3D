<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import * as THREE from 'three'
// Importamos el cargador específico para archivos STL
import { STLLoader } from 'three/examples/jsm/loaders/STLLoader.js'
// Importamos los controles para poder rotar el objeto con el mouse
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

// Referencia al div donde inyectaremos el Canvas 3D
const contenedor3D = ref(null)

let escena, camara, renderizador, controles, animacionId

onMounted(() => {
  // 1. CREAR LA ESCENA (El "Mundo")
  escena = new THREE.Scene()
  escena.background = new THREE.Color(0xf4f4f9) // Color de fondo claro

  // 2. CREAR LA CÁMARA (Los "Ojos" del usuario)
  camara = new THREE.PerspectiveCamera(
    75, 
    contenedor3D.value.clientWidth / contenedor3D.value.clientHeight, 
    0.1, 
    1000
  )
  camara.position.set(0, 50, 100) // Posición inicial de la cámara

  // 3. CREAR EL RENDERIZADOR (El motor de dibujo)
  renderizador = new THREE.WebGLRenderer({ antialias: true }) // Antialias suaviza los bordes
  renderizador.setSize(contenedor3D.value.clientWidth, contenedor3D.value.clientHeight)
  contenedor3D.value.appendChild(renderizador.domElement)

  // 4. AGREGAR LUCES (Sin luz, el STL se vería totalmente negro)
  const luzAmbiental = new THREE.AmbientLight(0x404040, 2) // Luz base suave
  escena.add(luzAmbiental)
  
  const luzDireccional = new THREE.DirectionalLight(0xffffff, 2) // Luz principal simulando el sol
  luzDireccional.position.set(1, 1, 1).normalize()
  escena.add(luzDireccional)

  // 5. CARGAR EL MODELO STL
  const cargador = new STLLoader()
  // Apuntamos al archivo en la carpeta public/
  cargador.load('/modelo.stl', (geometria) => {
    
    // Centramos la geometría (los STL a veces traen su punto de origen descentrado)
    geometria.center() 

    // Le damos un material (color azul técnico, con un poco de brillo metálico)
    const material = new THREE.MeshStandardMaterial({ 
      color: 0x007bff, 
      metalness: 0.3, 
      roughness: 0.4 
    })

    // Juntamos la geometría y el material en una Malla (Mesh)
    const modelo = new THREE.Mesh(geometria, material)
    
    // Rotamos el modelo si viene acostado por defecto (muy común en STL de impresión 3D)
    modelo.rotation.x = -Math.PI / 2
    
    // Lo agregamos a la escena
    escena.add(modelo)
  })

  // 6. CONTROLES DEL MOUSE (Giro, Zoom, Desplazamiento)
  controles = new OrbitControls(camara, renderizador.domElement)
  controles.enableDamping = true // Le da un efecto de inercia/fricción muy elegante

  // 7. BUCLE DE ANIMACIÓN (Actualiza la pantalla 60 veces por segundo)
  const animar = () => {
    animacionId = requestAnimationFrame(animar)
    controles.update() // Necesario por el damping
    renderizador.render(escena, camara)
  }
  
  animar()
})

// LIMPIEZA: Es vital para el rendimiento liberar la memoria si el usuario cambia de página
onBeforeUnmount(() => {
  cancelAnimationFrame(animacionId)
  if (renderizador) renderizador.dispose()
})
</script>

<template>
  <div class="contenedor-visor">
    <div ref="contenedor3D" class="canvas-3d"></div>
  </div>
</template>

<style scoped>
.contenedor-visor {
  width: 100%;
  display: flex;
  justify-content: center;
  padding: 2rem 0;
}
.canvas-3d {
  width: 800px;
  height: 500px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 10px 20px rgba(0,0,0,0.2);
  background-color: #f4f4f9; /* Mismo color que la escena para que no parpadee al cargar */
}
</style>