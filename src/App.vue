<template>
  <div id="app">
    <header>
      <img src="https://cdn-icons-png.freepik.com/512/5592/5592147.png" alt="" />
    </header>

    <h1>Historial de movimientos</h1>
    <table>
      <thead>
        <tr>
          <th>Título</th>
          <th>Valor</th>
          <th>Tipo</th>
          <th>Descripción</th>
        </tr>
      </thead>
      <tbody v-if="movements.length > 0">
        <tr v-for="(movement, index) in movements" :key="index">
          <td>{{ movement.title }}</td>
          <td>${{ Number(movement.value).toLocaleString() }}</td>
          <td>{{ movement.type }}</td>
          <td>{{ movement.description }}</td>
        </tr>
      </tbody>
      <tbody v-else>
        <tr>
          <td colspan="4" style="text-align: center">No hay movimientos registrados</td>
        </tr>
      </tbody>
    </table>

    <div class="buttons-saldo">
      <h2>Saldo: ${{ Number(total).toLocaleString() }}</h2>
      <button @click="addMovement" class="btn">Agregar Movimiento</button>
    </div>

    <section
      style="
        width: 100vw;
        display: flex;
        align-items: center;
        justify-content: center;
        flex-direction: column;
      "
    >
      <h3>Grafica</h3>
      <div class="grafica">
        <Line id="lineChart" :key="chartKey" :data="dataGrafico" :options="chartOptions" />
      </div>
    </section>
  </div>
</template>

<script>
import Swal from 'sweetalert2'
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
} from 'chart.js'
import { Line } from 'vue-chartjs'
// Registro de elementos necesarios para el gráfico
ChartJS.register(CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend)
export default {
  data() {
    // Inicialización de datos de la gráfica y clave del gráfico
    ;(this.dataGrafico = {
      labels: [],
      datasets: [
        {
          label: 'Movimientos',
          backgroundColor: '#f87979',
          data: []
        }
      ]
    }),
      (this.chartKey = 0),
      (this.chartOptions = {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          y: {
            beginAtZero: true
          }
        },
        elements: {
          line: {
            borderWidth: 2, // Ancho de la línea
            borderColor: '#ffffff' // Color de la línea
          },
          point: {
            backgroundColor: '#5e5df0' // Color de los puntos
          }
        }
      })
    this.totalHistory = []
    return {
      movements: []
    }
  },
  components: {
    Line
  },
  computed: {
    // Cálculo del saldo total
    total() {
      return this.movements.reduce((acc, mov) => {
        return mov.type === 'Ingreso' ? acc + mov.value : acc - mov.value
      }, 0)
    }
  },
  methods: {
    // Método para agregar un nuevo movimiento y actualizar la grafica
    addMovement() {
      Swal.fire({
        title: 'Agregar Movimiento',
        html: `
          <div style="width:100%;display: flex;align-items: center;justify-content: center;flex-direction: column; gap: 1rem;">
            <input type="text" id="title" class="swal2-input" placeholder="Título" style="width:80%">
            <input type="number" id="value" class="swal2-input" placeholder="Valor" style="width:80%">
            <select id="type" class="swal2-input" style="width:80%">
              <option>Ingreso</option>
              <option>Gasto</option>
            </select>
            <input type="text" id="description" class="swal2-input" placeholder="Descripción" style="width:80%">
          </div>
        `,
        focusConfirm: false,
        showCancelButton: true,
        cancelButtonText: 'Cancelar',
        confirmButtonText: 'Agregar',
        preConfirm: () => {
          const title = Swal.getPopup().querySelector('#title').value
          const value = Swal.getPopup().querySelector('#value').value
          const type = Swal.getPopup().querySelector('#type').value
          const description = Swal.getPopup().querySelector('#description').value

          if (!title || !value || !type || !description) {
            Swal.showValidationMessage(`Por favor, completa todos los campos`)
            return null
          }

          const parsedValue = parseFloat(value)
          if (type === 'Gasto' && this.total - parsedValue < 0) {
            Swal.showValidationMessage(`No tienes suficientes fondos para este gasto`)
            return null
          }

          return {
            title: title,
            value: parsedValue,
            type: type,
            description: description
          }
        }
      }).then((result) => {
        if (result.isConfirmed) {
          this.movements.push(result.value)
          const nuevoLabel = 'Total: #' + (this.dataGrafico.labels.length + 1)
          const total = this.total

          if (this.dataGrafico.labels.length >= 5) {
            this.dataGrafico.labels.shift()
            this.totalHistory.shift()
          }

          this.totalHistory.push(total)
          this.dataGrafico.labels.push(nuevoLabel)
          this.dataGrafico.datasets[0].data = this.totalHistory
          this.chartKey++
          Swal.fire('El movimiento se ha agregado correctamente', '', 'success')
        }
      })
    }
  }
}
</script>

<style scoped>
header {
  width: 100vw;
  justify-content: center;
  align-items: center;
  display: flex;
  padding: 1rem;
}

header img {
  width: 10vw;
}

.grafica {
  width: 50vw;
  padding-bottom: 1rem;
}
#app {
  display: flex;
  width: 100vw;
  height: 100vh;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}

.buttons-saldo {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  justify-content: center;
  margin-top: 1rem;
  padding: 1rem;
}

.btn {
  background: #5e5df0;
  border-radius: 999px;
  box-shadow: #5e5df0 0 10px 20px -10px;
  box-sizing: border-box;
  color: #ffffff;
  cursor: pointer;
  font-family: 'Poppins', sans-serif;
  font-size: 16px;
  font-weight: 700;
  line-height: 24px;
  opacity: 1;
  outline: 0 solid transparent;
  padding: 8px 18px;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  width: fit-content;
  word-break: break-word;
  border: 0;
  display: inline;
  margin-left: 1rem;
}

table {
  margin-top: 2rem;
  border-collapse: collapse;
  width: 80vw;
  max-width: 100vw;
}

th,
td {
  border: 1px solid #dddddd;
  text-align: left;
  padding: 8px;
}

th {
  background: #5e5df0;
  color: #ffffff;
}

tr:nth-child(even) {
  background-color: #3c3b3b;
}

tr:hover {
  background-color: #4a4848;
}

@media (max-width: 1024px) {
  header img {
    width: 20vw;
    padding: 1rem;
  }

  .grafica {
    width: 70vw;
  }
}

@media (max-width: 600px) {
  header img {
    width: 30vw;
    padding: 1.5rem;
  }

  .grafica {
    width: 90vw;
  }
}

@media (max-width: 400px) {
  header img {
    width: 50vw;
    padding: 1.5rem;
  }

  .buttons-saldo {
    flex-direction: column;
  }

  .buttons-saldo h2 {
    padding-bottom: 1rem;
  }

  h1 {
    font-size: 23px;
  }
  h3 {
    font-size: larger;
  }

  .grafica {
    width: 90vw;
  }
}
</style>
