<script setup>
  import {ref, onMounted, reactive, computed} from 'vue'
  import Alerta from './components/Alerta.vue'
  import Spinner from './components/Spinner.vue'

  const monedas = ref([
      { codigo: 'USD', texto: 'Dolar de Estados Unidos'},
      { codigo: 'MXN', texto: 'Peso Mexicano'},
      { codigo: 'EUR', texto: 'Euro'},
      { codigo: 'GBP', texto: 'Libra Esterlina'},
      { codigo: 'COP', texto: 'Peso Colombiano'},
  ])
  const criptomonedas = ref([])
  const cotizar = reactive({
    moneda:'',
    criptomoneda:''
  })
  // objecto ref porque los valores se insertan en otra parte, porque si fuera con reactive tocaría iniciarlizar todos los
  // valores y son muchos 
  const cotizacion = ref({})
  const cargando = ref(false)
  const error = ref('')
  

  onMounted(()=>{
    fetch('https://min-api.cryptocompare.com/data/top/totalvolfull?limit=10&tsym=USD')
    // Primera promesa es la conexion y la segunda los datos
    .then(respuesta => respuesta.json())
    // datos
    .then(data => {
      criptomonedas.value = data.Data
    })
  })
  const cotizarCripto = () =>{
    // validar que cotizar este lleno
    // comprueba si algunos de los elementos esta vacios
    if(Object.values(cotizar).includes('')){
      error.value = 'Todos los campos son obligatorios'
      setTimeout(()=>{
        error.value = ''
      },3000)
      return 
    }
    error.value = ''
    obtenerCotizacion()
  }
  const obtenerCotizacion = async()=>{
    cargando.value = true
    const {moneda,criptomoneda} = cotizar
    const url = `https://min-api.cryptocompare.com/data/pricemultifull?fsyms=${criptomoneda}&tsyms=${moneda}`
    const respuesta = await fetch(url)
    const data = await respuesta.json()

    // Buscar en data/DISPLAY donde las variables criptomonedas y monedas se encuentren en el objecto
    cotizacion.value = data.DISPLAY[criptomoneda][moneda]
    cargando.value = false
  }

  const mostrarResultado = computed(()=>{
    // Si la longitud de los valores del objecto cotizacion es mayor a 0 es porque no esta vacio
    // esto retorna false o true
    return Object.values(cotizacion.value).length > 0

  })
</script>

<template>
  <div class="contenedor">
    <h1 class="titulo">Cotizador de <span>Criptomonedas</span></h1>
    <div class="contenido">
      <Alerta
      v-if="error"
      >
      {{ error }}
      </Alerta>
      <form class="formulario"
        @submit.prevent="cotizarCripto"
      >

        <div class="campo">
          <label for="moneda" class="">Moneda:</label>
            <select id="moneda"
            v-model="cotizar.moneda"
            >
              <option value="">-- Selecciona --</option>
              <option v-for="moneda in monedas" 
              :value="moneda.codigo">
              {{ moneda.texto }}
            </option>
            </select>
        </div>

        <div class="campo">
          <label for="crypto" class="">Criptomoneda:</label>
            <select id="crypto"
            v-model="cotizar.criptomoneda"
            
            >
              <option value="">-- Selecciona --</option>
              <option v-for="cripto in criptomonedas" 
              :value="cripto.CoinInfo.Name">
              {{ cripto.CoinInfo.FullName }}
            </option>
            </select>
        </div>
        <input type="submit" value="Cotizar" >
      </form>
      <Spinner
      v-if="cargando"
      />
      <div 
      v-if="mostrarResultado"
      class="contenedor-resultado">
        <h2>Cotización</h2>
        <div class="resultado">
          <img :src=" 'https://cryptocompare.com/'+cotizacion.IMAGEURL " alt="Imagen critp">
         <div>
          <p>El precio es de: <span>{{ cotizacion.PRICE }}</span></p>
          <p>Precio más alto del día: <span>{{ cotizacion.HIGHDAY }}</span></p>
          <p>Precio más bajo del día: <span>{{ cotizacion.LOWDAY }}</span></p>
          <p>Variación ultimas 24hrs <span>{{ cotizacion.CHANGEPCT24HOUR }}%</span></p>
          <p>Última Actualiazación: <span>{{ cotizacion.LASTUPDATE }}</span></p>
         </div> 
        </div>
      </div>

    </div>
  </div>
</template>

<style scoped>
</style>
