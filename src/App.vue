<template>
  <div>
    <header>
      <nav class="navbar navbar-inverse navbar-static-top">
        <div class="container">
          <div class="navbar-header">
            <a class="navbar-brand" href="">Jogo de Memória</a>
          </div>
          <form class="navbar-form navbar-left" role="search">
            <div class="form-group">
              <input type="text" placeholder="Insira o seu Nome..." class="form-control" name="nome" v-on:input="nome = $event.target.value" :disabled="desabilitado" />
            </div>
            <button type="button" @click="comecar()" :disabled="desabilitado || nome == ''" class="btn btn-primary">Jogar!</button>
            <button type="button" @click="reiniciar()" :disabled="!desabilitado" class="btn btn-default">Começar Novo Jogo!</button>
          </form>
          <div class="navbar-right">
            <p class="navbar-text">Jogador: <b>{{ this.nome }}</b></p>
            <p class="navbar-text">Tempo: <b>{{ this.tempo }}</b></p>
            <p class="navbar-text">Rodadas: <b>{{ this.rodadas }}</b></p>
          </div>
        </div>
      </nav>
    </header>
    <section class="mesa container-fluid">
      <div class="cartas container" v-if="!pararJogo">
        <div class="carta" v-for="carta in cartas" :class="{ virada: carta.virada, correta: carta.correta}" @click="virarCarta(carta)">
          <div class="costas"></div>
          <div class="frente" v-bind:style="{ backgroundImage: 'url(' + carta.url + ')' }"></div>
        </div>
        <footer>
          As ilustrações são do <a href="https://dribbble.com/isaac317" target="_blank">Isaac Montemayor</a>.
        </footer>
      </div>
      <div class="info container" v-if="mostrarResultado">
        <div class="jumbotron" v-if="mostrarFinal">
          <div class="well">
            <div class="page-header">
              <h1>Parabéns!! Você ganhou.</h1>
            </div>
            <div class="alert alert-success" role="alert">Tempo: {{ tempo }}</div>
            <div class="alert alert-success" role="alert">Pontuação: {{ rodadas }}</div>
            <div class="form-group">
              <button @click="reiniciar()" class="btn btn-primary btn-block btn-lg">Jogar novamente!</button>
            </div>
          </div>
          <div class="resultados">
            <table class="table table-striped table-bordered">
              <thead>
                <tr>
                  <th>Jogador</th>
                  <th>Tempo</th>
                  <th>Rodadas</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="jogador in ordenarResultados" class="success">
                  <td>{{ jogador.nome }}</td>
                  <td>{{ jogador.tempo }}</td>
                  <td>{{ jogador.rodadas }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>
<script>

let cartas = [
  {
    url: './src/assets/cartas/1.jpg',
    name: 1
  },
  {
    url: './src/assets/cartas/2.jpg',
    name: 2
  },
  {
    url: './src/assets/cartas/3.jpg',
    name: 3
  },
  {
    url: './src/assets/cartas/4.jpg',
    name: 4
  },
  {
    url: './src/assets/cartas/5.jpg',
    name: 5
  },
  {
    url: './src/assets/cartas/6.jpg',
    name: 6
  },
  {
    url: './src/assets/cartas/7.jpg',
    name: 7
  },
  {
    url: './src/assets/cartas/8.jpg',
    name: 8
  },
  {
    url: './src/assets/cartas/9.jpg',
    name: 9
  },
  {
    url: './src/assets/cartas/10.jpg',
    name: 10
  }
];

let embaralhar = () => {

  let baralho = [].concat(_.cloneDeep(cartas), _.cloneDeep(cartas));

  return _.shuffle(baralho);

}

export default {

  data() {

    return {

      nome: '',
      pararJogo: true,
      desabilitado: false,
      cronometro: 0,
      tempo: 0,
      cartas: [],
      rodadas: 0,
      combinacoes: 0,
      desvirarTempo: 0,
      mostrarResultado: false,
      mostrarFinal: false,
      jogadores: []
    }
	},
	
	methods: {

		reiniciar() {

      this.mostrarResultado = false;

      this.mostrarFinal = false;

      this.pararJogo = true;

      this.desabilitado = false;

      this.tempo = 0;

      this.rodadas = 0;

      clearInterval(this.cronometro);

      this.cronometro = 0;

      let cartas = embaralhar();   

      _.each(cartas, (carta) => {

          carta.virada = false;

          carta.correta = false;

      });      

      this.cartas = cartas;	

		},
		
		comecar() {

      this.mostrarResultado = false;

      this.mostrarFinal = false;

      this.pararJogo = false;

      this.desabilitado = true;

      this.cronometro = setInterval(() => {

          this.tempo++;

      }, 1000);

		},
    
    fim() {

      this.pararJogo = true;

      clearInterval(this.cronometro);

      this.mostrarResultado = true;

      this.mostrarFinal = true;

      let jogador = {
        nome: this.nome,
        tempo: this.tempo,
        rodadas: this.rodadas
      };

      this.jogadores.push(jogador);

    },

    cartasViradas() {

      return _.filter(this.cartas, carta => carta.virada);

    },

    mesmaCartaVirada() {

      let cartasViradas = this.cartasViradas();

      if (cartasViradas.length == 2) {

        if (cartasViradas[0].name == cartasViradas[1].name) {

            return true;

        }

      }

    },

    deixarCartaVirada() {

      _.each(this.cartas, (carta) => {

        if (carta.virada) {

          carta.correta = true;

        }

      });

    },

    verificarTodasCorretas() {

      let cartasEncontradas = _.filter(this.cartas, carta => carta.correta);

      this.combinacoes = cartasEncontradas.length;

      if (cartasEncontradas.length == this.cartas.length) {

          return true;

      }

    },
    
    virarCarta(carta) {

      if (carta.correta || carta.virada) return;
      
      let contador = this.cartasViradas().length;

      if (contador == 0) {

          carta.virada = !carta.virada;

      } else if (contador == 1) {

        carta.virada = !carta.virada;

        this.rodadas++;

        if (this.mesmaCartaVirada()) {

          this.flipBackTimer = setTimeout( ()=> {

            this.limparDesvirarTempo();

            this.deixarCartaVirada();

            this.desvirar();

            if (this.verificarTodasCorretas()) {

                this.fim();

            }   

          }, 200);

        } else {

          this.desvirarTempo = setTimeout( ()=> {

              this.limparDesvirarTempo();

              this.desvirar();

          }, 1000);

        }

      }

    },

    limparDesvirarTempo() {

      clearTimeout(this.desvirarTempo);

      this.desvirarTempo = null;

    },
    
    desvirar() {

        _.map(this.cartas, carta => carta.virada = false);

    },
      
	},

  computed: {

    ordenarResultados: function() {

      function compare(a, b) {

        if (a.rodadas < b.rodadas) {

          return -1;

        }

        if (a.rodadas > b.rodadas) {

          return 1;

        }

        return 0;

      }

      return this.jogadores.sort(compare);

    }

  },

	created() {

		this.reiniciar();

	}

}
</script>

<style scoped>

  * {
    box-sizing: border-box;
  }

  .navbar {
    margin: 0;
  }

  .mesa {
    background-color: #2B3736;
    padding: 20px;
    min-height: 100%;
    display: block;
    position: absolute;
    width: 100%;
    height: 100%;
  }

  .cartas .carta {
    position: relative;
    display: inline-block;
    width: 218px;
    height: 127px;
    margin: 5px;
    transition: opacity 0.3s;
  }

  .cartas .carta .frente, .cartas .carta .costas {
    border-radius: 10px;
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    width: 100%;
    height: 100%;
    backface-visibility: hidden;
    transform: translateZ(0);
    transition: transform 0.5s;
    transform-style: preserve-3d;
  }

  .cartas .carta .costas {
    background-color: #c93756;
    background-image: url('/src/assets/logo.png');
    background-size: 75%;
    background-position: center;
    background-repeat: no-repeat;
  }

  .cartas .carta .frente {
    transform: rotateY(-180deg);
    background-size: cover;
    background-repeat: no-repeat;
    background-position: center;
  }

  .cartas .carta.virada .costas, .cartas .carta.correta .costas {
    transform: rotateY(180deg);
  }

  .cartas .carta.virada .frente, .cartas .carta.correta .frente {
    transform: rotateY(0deg);
  }

  .cartas .carta.correta {
    opacity: 0.3;
  }

  footer {
    color: #FFF;
    padding: 10px;
  }

</style>
