<template>
  <div class="app">
    <h1 class="titulo">PokéDex Vue</h1>

    <div class="buscador">
      <input
        type="text"
        v-model="apicode"
        placeholder="Escribe un nombre de Pokémon"
      />
      <button @click="buscarPokemon">Buscar</button>
    </div>

    <div v-if="error" class="error">{{ error }}</div>

    <div v-if="pokemon && pokemon.sprites" class="container">
      <!-- Header con número y nombre -->
      <div class="header" :style="backcolor()">
        <div class="numero">#{{ pokemon.id }}</div>
        <h2 class="nombre">{{ pokemon.name }}</h2>
      </div>

      <!-- Imagen del Pokémon -->
      <div class="imagen-container">
        <div class="circulo-imagen" :style="backcolor()">
          <img :src="pokemon.sprites.other.showdown.front_default" alt="pokemon" />
        </div>
      </div>

      <!-- Información básica -->
      <div class="info-basica">
        <div class="medidas">
          <div class="medida">
            <span class="valor">{{ (pokemon.height / 10).toFixed(1) }} m</span>
            <span class="etiqueta">Altura</span>
          </div>
          <div class="medida">
            <span class="valor">{{ (pokemon.weight / 10).toFixed(1) }} kg</span>
            <span class="etiqueta">Peso</span>
          </div>
        </div>
      </div>

      <!-- Tipos -->
      <div class="tipos-container">
        <h3>Tipo</h3>
        <div class="tipos">
          <span
            v-for="(it, i) in pokemon.types"
            :key="i"
            :style="color(it.type.name)"
            class="tipo"
          >
            {{ it.type.name }}
          </span>
        </div>
      </div>

      <!-- Debilidades -->
      <div class="debilidades-container" v-if="weak.length > 0">
        <h3>Debilidades</h3>
        <div class="debilidades">
          <span
            v-for="(d, i) in weak"
            :key="i"
            :style="color(d.name)"
            class="debilidad"
          >
            {{ d.name }}
          </span>
        </div>
      </div>

      <!-- Estadísticas -->
      <div class="stats-container">
        <h3>Estadísticas</h3>
        <div class="stats">
          <div
            v-for="(it, i) in pokemon.stats"
            :key="i"
            class="stat"
          >
            <div class="stat-info">
              <span class="stat-nombre">{{ formatStatName(it.stat.name) }}</span>
              <span class="stat-valor">{{ it.base_stat }}</span>
            </div>
            <div class="stat-barra-bg">
              <div
                class="stat-barra-fill"
                :style="barraColor(it.base_stat)"
              ></div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="pokeball" class="pokeball activo">
      <div class="line"></div>
      <div class="button"></div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";

const tipoColor = {
  normal: "#A8A77A",
  fire: "#EE8130",
  water: "#6390F0",
  electric: "#F7D02C",
  grass: "#7AC74C",
  ice: "#96D9D6",
  fighting: "#C22E28",
  poison: "#A33EA1",
  ground: "#E2BF65",
  flying: "#A98FF3",
  psychic: "#F95587",
  bug: "#A6B91A",
  rock: "#B6A136",
  ghost: "#735797",
  dragon: "#6F35FC",
  dark: "#705746",
  steel: "#B7B7CE",
  fairy: "#D685AD",
};

const pokeball = ref(false);
const apicode = ref("");
const pokemon = ref(null);
const error = ref("");
const url = ref(null);
const weak = ref([]);
const mainColor = ref("#ccc");
const secondaryColor = ref(null);

async function buscarPokemon() {
  try {
    error.value = "";
    pokemon.value = null;
    weak.value = [];

    if (!apicode.value) {
      error.value = "Por favor ingresa el nombre de un Pokémon";
      return;
    }

    const { data } = await axios.get(
      `https://pokeapi.co/api/v2/pokemon/${apicode.value.toLowerCase()}`
    );
    pokemon.value = data;

    const tipos = pokemon.value.types.map((t) => t.type.name);
    mainColor.value = tipoColor[tipos[0]] || "#999";
    secondaryColor.value = tipos[1] ? tipoColor[tipos[1]] : null;

    url.value = pokemon.value.types[0].type.url;
    const resWeak = await axios.get(url.value);
    weak.value = resWeak.data.damage_relations.double_damage_from;
    pokeball.value = true;
  } catch (err) {
    error.value = "Pokémon no encontrado o error en la petición";
    console.error(err);
  }
}

function color(typeName) {
  const colorBase = tipoColor[typeName];
  return {
    backgroundColor: colorBase ? colorBase : "#888",
    color: "white",
    padding: "6px 12px",
    borderRadius: "10px",
    textShadow: "1px 1px 3px black",
    margin: "4px",
    display: "inline-block",
    fontSize: "0.7rem",
    fontWeight: "bold",
  };
}

function barraColor(statValue) {
  const percent = Math.min((statValue / 150) * 100, 100);
  const bg =
    secondaryColor.value
      ? `linear-gradient(90deg, ${mainColor.value}, ${secondaryColor.value})`
      : mainColor.value;
  return {
    width: `${percent}%`,
    background: bg,
    borderRadius: "4px",
    height: "100%",
    transition: "width 0.8s ease-out",
  };
}

function lightenColor(color, percent) {
  const num = parseInt(color.replace("#", ""), 16);
  const amt = Math.round(2.55 * percent);
  const R = (num >> 16) + amt;
  const G = ((num >> 8) & 0x00ff) + amt;
  const B = (num & 0x0000ff) + amt;

  return (
    "#" +
    (
      0x1000000 +
      (R < 255 ? (R < 1 ? 0 : R) : 255) * 0x10000 +
      (G < 255 ? (G < 1 ? 0 : G) : 255) * 0x100 +
      (B < 255 ? (B < 1 ? 0 : B) : 255)
    )
      .toString(16)
      .slice(1)
  );
}

function backcolor(){
  const back=
   secondaryColor.value
        ? `linear-gradient(135deg, ${mainColor.value}, ${secondaryColor.value})`
          : `linear-gradient(135deg, ${mainColor.value}, ${lightenColor(mainColor.value, 20)})`;
  return {
    background:`${back}`,
  };
}

function formatStatName(statName) {
  const statMap = {
    'hp': 'HP',
    'attack': 'Ataque',
    'defense': 'Defensa',
    'special-attack': 'At. Esp.',
    'special-defense': 'Def. Esp.',
    'speed': 'Velocidad'
  };
  return statMap[statName] || statName;
}
</script>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.app {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: 'Poppins', sans-serif;
  color: #333;
  padding: 15px;
  background: linear-gradient(to bottom, #254878 40%, #872b3a 100%);
  border-radius: 20px;
  box-shadow: 0 0 20px rgba(0,0,0,0.5);
  overflow: hidden;
  max-width: 400px;
  margin: auto;
  min-height: 100vh;
}

/* === Título === */
.titulo {
  font-family: "Press Start 2P", system-ui;
  text-align: center;
  font-size: 1.2rem;
  color: #ffcb05;
  text-shadow: 2px 2px #2a2a2a;
  margin: 10px 0 20px;
}

/* === Buscador === */
.buscador {
  display: flex;
  width: 100%;
  margin-bottom: 15px;
}

.buscador input {
  flex: 1;
  padding: 10px;
  border: none;
  border-radius: 10px 0 0 10px;
  font-size: 0.9rem;
}

.buscador button {
  padding: 10px 15px;
  background: #ffcb05;
  color: #2a2a2a;
  border: none;
  border-radius: 0 10px 10px 0;
  font-weight: bold;
  cursor: pointer;
}

/* === Error === */
.error {
  color: #ff6b6b;
  background: rgba(255, 255, 255, 0.8);
  padding: 10px;
  border-radius: 10px;
  margin-bottom: 15px;
  text-align: center;
  width: 100%;
}

/* === Container principal === */
.container {
  width: 100%;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 15px;
  overflow: hidden;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

/* === Header === */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  color: white;
}

.numero {
  font-size: 1rem;
  font-weight: bold;
  opacity: 0.8;
}

.nombre {
  font-size: 1.4rem;
  text-transform: capitalize;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
}

/* === Imagen === */
.imagen-container {
  display: flex;
  justify-content: center;
  padding: 10px;
}

.circulo-imagen {
  width: 180px;
  height: 180px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.3);
}

.circulo-imagen img {
  width: 140px;
  height: 140px;
  object-fit: contain;
  filter: drop-shadow(0 0 5px rgba(255, 255, 255, 0.5));
}

/* === Información básica === */
.info-basica {
  padding: 0 15px;
}

.medidas {
  display: flex;
  justify-content: space-around;
  margin: 10px 0;
}

.medida {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.valor {
  font-weight: bold;
  font-size: 1rem;
}

.etiqueta {
  font-size: 0.8rem;
  color: #666;
  margin-top: 5px;
}

/* === Tipos === */
.tipos-container, .debilidades-container, .stats-container {
  padding: 15px;
  border-top: 1px solid #eee;
}

.tipos-container h3, .debilidades-container h3, .stats-container h3 {
  margin-bottom: 10px;
  color: #444;
  font-size: 1rem;
}

.tipos, .debilidades {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.tipo, .debilidad {
  padding: 6px 12px;
  border-radius: 10px;
  font-size: 0.8rem;
  font-weight: bold;
  text-transform: capitalize;
  color: white;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
}

/* === Estadísticas === */
.stats {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.stat {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.stat-info {
  display: flex;
  justify-content: space-between;
  font-size: 0.8rem;
}

.stat-nombre {
  text-transform: capitalize;
}

.stat-valor {
  font-weight: bold;
}

.stat-barra-bg {
  height: 8px;
  background: #e0e0e0;
  border-radius: 4px;
  overflow: hidden;
}

.stat-barra-fill {
  height: 100%;
  border-radius: 4px;
}

/* === Pokéball loader (opcional) === */
.pokeball {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: white;
  position: relative;
  margin: 20px auto;
  border: 3px solid black;
  overflow: hidden;
}

.pokeball .line {
  position: absolute;
  top: 50%;
  left: 0;
  width: 100%;
  height: 3px;
  background: black;
  transform: translateY(-50%);
}

.pokeball .button {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 20px;
  height: 20px;
  background: white;
  border: 3px solid black;
  border-radius: 50%;
  transform: translate(-50%, -50%);
  z-index: 1;
}

/* === Responsive === */
@media (max-width: 450px) {
  .app {
    padding: 10px;
    max-width: 350px;
  }
  
  .titulo {
    font-size: 1rem;
  }
  
  .circulo-imagen {
    width: 150px;
    height: 150px;
  }
  
  .circulo-imagen img {
    width: 120px;
    height: 120px;
  }
  
  .nombre {
    font-size: 1.2rem;
  }
}
</style>