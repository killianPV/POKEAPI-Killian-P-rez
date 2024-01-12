<!DOCTYPE html>
<html>
<head>
  <title>Cartas de Pokémon</title>
  <style>
    /* Estilos para las cartas */
    body {
      font-family: Arial, sans-serif;
      background-color: #f0f0f0;
      margin: 0;
      padding: 20px;
      background-image: url('fotoFondo.jpg');
    }

    .controls {
      text-align: center;
      margin-bottom: 20px;
    }

    .controls input[type="number"],
    .controls input[type="text"],
    .controls button {
      padding: 8px;
      margin: 5px;
      border: none;
      border-radius: 4px;
      font-size: 16px;
    }

    .battle-button {
      background-color: #3498db;
      color: white;
      cursor: pointer;
    }

    .battle-button:disabled {
      background-color: #a0a0a0;
      cursor: not-allowed;
    }

    .card {
      border: 1px solid #ccc;
      padding: 10px;
      margin: 10px;
      display: inline-block;
      cursor: pointer;
      width: 200px;
      text-align: center;
      transition: border-color 0.3s ease;
      background-color: white;
      border-radius: 8px;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }

    .card.selected {
      border: 2px solid red;
    }

    .pokemon-image {
      width: 120px;
      height: 120px;
    }

    .result {
      font-weight: bold;
      margin-top: 20px;
      text-align: center;
      font-size: 18px;
    }
  </style>
</head>
<body>
  <div class="controls">
    <input type="number" id="pokemon-number" placeholder="Cantidad de Pokémon">
    <button id="random-pokemon-button">Generar Pokémon aleatorios</button>
    <input type="text" id="filter" placeholder="Filtrar por nombre o tipo">
    <button class="battle-button" disabled>Luchar</button>
  </div>
  <div id="pokemon-cards"></div>
  <div class="result"></div>

  <script>
    let selectedPokemons = [];
    let originalPokemonNumber = 10; // Guarda la cantidad original de Pokémon

    document.getElementById('random-pokemon-button').addEventListener('click', () => {
      originalPokemonNumber = document.getElementById('pokemon-number').value || 10;
      getPokemonData(originalPokemonNumber);
    });

    document.querySelector('.battle-button').addEventListener('click', () => {
      if (selectedPokemons.length === 2) {
        fight(selectedPokemons);
      } else {
        alert('Selecciona dos Pokémon para la batalla.');
      }
    });

    async function getPokemonData(numberOfPokemon) {
  const pokemonCards = document.getElementById('pokemon-cards');
  pokemonCards.innerHTML = ''; // Elimina los elementos hijos

  selectedPokemons = []; // Reinicia la variable

  for (let i = 0; i < numberOfPokemon; i++) {
    const randomId = Math.floor(Math.random() * 898) + 1;
    const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${randomId}`);
    const pokemonData = await response.json();

    createPokemonCard(pokemonData);
  }
}

    function createPokemonCard(pokemonData) {
      const pokemonCards = document.getElementById('pokemon-cards');
      const card = document.createElement('div');
      card.classList.add('card');

      const types = pokemonData.types.map(type => type.type.name).join(', ');

      card.innerHTML = `
        <h3 class="name">${pokemonData.name}</h3>
        <img class="pokemon-image" src="${pokemonData.sprites.front_default}" alt="Pokemon">
        <p class="type">Tipo: ${types}</p>
        <p>Ataque: <span class="attack">${pokemonData.stats[1].base_stat}</span></p>
        <p>Defensa: <span class="defense">${pokemonData.stats[2].base_stat}</span></p>
        <p>Vida: <span class="hp">${pokemonData.stats[0].base_stat}</span></p>
      `;

      card.addEventListener('click', () => {
        if (selectedPokemons.length < 2) {
          card.classList.toggle('selected');
          if (card.classList.contains('selected')) {
            selectedPokemons.push(pokemonData);
          } else {
            selectedPokemons = selectedPokemons.filter(pokemon => pokemon !== pokemonData);
          }
        } else {
          alert('Solo puedes seleccionar dos Pokémon para la batalla.');
        }

        if (selectedPokemons.length === 2) {
          document.querySelector('.battle-button').disabled = false;
        } else {
          document.querySelector('.battle-button').disabled = true;
        }
      });

      pokemonCards.appendChild(card);
    }

    function fight(selectedPokemons) {
      const attack1 = selectedPokemons[0].stats[1].base_stat;
      const defense2 = selectedPokemons[1].stats[2].base_stat;
      const name1 = selectedPokemons[0].name;
      const name2 = selectedPokemons[1].name;

      let result;

      if (attack1 > defense2) {
        result = `${name1} ha ganado.`;
      } else if (attack1 < defense2) {
        result = `${name2} ha ganado.`;
      } else {
        result = 'El combate termina en empate.';
      }

      displayResult(result);
      document.querySelector('.result').textContent = result;

      setTimeout(async () => {
  document.querySelector('.result').textContent = '';
  await getPokemonData(originalPokemonNumber); // Generar nuevos Pokémon
  document.querySelectorAll('.card.selected').forEach(card => card.classList.remove('selected')); // Desmarcar las cartas seleccionadas
  document.querySelector('.battle-button').removeAttribute('disabled'); // Habilitar el botón de batalla
}, 3000);

    }

    function displayResult(result) {
      const resultElement = document.querySelector('.result');
      resultElement.textContent = result;
    }
  </script>
</body>
</html>
