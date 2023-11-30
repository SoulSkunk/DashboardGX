<template>
  <h1>DashboardGX</h1>
  <h2>Mes raccourcis :</h2>
  <ul>
    <li v-for="raccourci in raccourcis" :key="raccourci.id">
      {{ raccourci.fields.New_racc }}
      <button @click="handleUpdate(raccourci)">Modifier</button>
      <button @click="deleteRaccourci(raccourci.id)">Supprimer</button>
    </li>
  </ul>

  <!-- FORM NEW RACC -->
  <!-- Button open modal -->
  <button @click="toggleModalBackground" id="btn_open_modal_input_background">
    Changer le background
  </button>
  <!-- From changement background -->
  <div v-if="displayModalBackground">
    <form>
      <label for="input_background">Entrez l'url de l'image background :</label>
      <input
        type="url"
        id="input_background"
        name="input_background"
        v-model="input_background"
        required
      />
      <input type="submit" value="Changer" />
    </form>
  </div>

  <!-- FORM NEW RACC -->
  <!-- Button open modal -->
  <button @click="toggleModalNewRacc" id="btn_open_modal_input_new_racc">
    Ajouter un raccourci
  </button>
  <!-- Form ajouter une url -->
  <div v-if="displayModalNewRacc">
    <form @submit.prevent="getRaccourcis">
      <label for="input_new_racc">Entrez une URL :</label>
      <input
        type="url"
        id="input_new_racc"
        name="input_new_racc"
        v-model="input_new_racc"
        required
      />
      <input
        type="submit"
        value="Ajouter"
      /><!-- A la validation du form la function se lance -->
    </form>
  </div>

  <!-- FORM MODIFY RACC -->
  <!-- Button open modal -->
  <button @click="toggleModalModifyRacc" id="btn_open_modal_input_modify_racc">
    Modifier un raccourci
  </button>
  <!-- Form modifier le raccourci -->
  <div v-if="displayModalModifyRacc">
    <form>
      <label for="input_modify_racc">Entrez nouvelle URL :</label>
      <input
        type="url"
        id="input_modify_racc"
        name="input_modify_racc"
        v-model="input_modify_racc"
        required
      />
      <input type="submit" value="Modifier" />
    </form>
  </div>

  <button v-if="edit" @click="updateRaccourci(selectedId)">Modifier</button>
  <button v-if="edit" @click="edit = false">Annuler</button>
</template>
<script>
//On met dans des constantes les infos de notre bdd airtable pour lier
const BASE_ID = "app5LMxubeFsTGn4z";
const TABLE_NAME = "UrlStock";
const VIEW_NAME = "Grid view";
const API_TOKEN =
  "pattD266cz4nD4L4F.2bcc09e878c5c7373283936460027dc15685331c08a6903d6f56c31214b3a89c";

export default {
  data() {
    return {
      raccourcis: [],
      input_background: "",
      input_new_racc: "",
      input_modify_racc: "",
      selectedId: null,
      edit: false,
      displayModalBackground: false,
      displayModalNewRacc: false,
      displayModalModifyRacc: false,
    };
  },
  methods: {
    handleResetForm() {
      this.input_background = "";
      this.input_new_racc = "";
      this.input_modify_racc = "";
    },
    // OPEN/CLOSE MODALS
    // background
    toggleModalBackground() {
      this.displayModalBackground = !this.displayModalBackground;
    },
    // newRacc
    toggleModalNewRacc() {
      this.displayModalNewRacc = !this.displayModalNewRacc;
    },
    // modifyRacc
    toggleModalModifyRacc() {
      this.displayModalModifyRacc = !this.displayModalModifyRacc;
    },

    // Recup des data dans airtable
    getRaccourcis() {
      fetch(
        `https://api.airtable.com/v0/${BASE_ID}/${TABLE_NAME}?view=${VIEW_NAME}`,
        {
          headers: {
            Authorization: `Bearer ${API_TOKEN}`,
          },
        }
      )
        .then((response) => response.json())
        .then((data) => {
          this.raccourcis = data.records;
          console.log(data.records);
        })
        .catch((error) => {
          console.log(error);
        });
    },
    // Je comprend pas
    createRaccourci() {
      fetch(`https://api.airtable.com/v0/${BASE_ID}/${TABLE_NAME}`, {
        headers: {
          Authorization: `Bearer ${API_TOKEN}`,
          "Content-Type": "application/json",
        },
        method: "POST",
        body: JSON.stringify({
          fields: {
            Background: this.input_background,
            New_racc: this.input_new_racc,
            Modify_racc: this.input_modify_racc,
          },
        }),
      })
        .then((response) => response.json())
        .then((data) => {
          this.handleResetForm();
        })
        .catch((error) => {
          console.log(error);
        });
    },
    // Supprimer le raccourci
    deleteRaccourci(id) {
      fetch(`https://api.airtable.com/v0/${BASE_ID}/${TABLE_NAME}/${id}`, {
        headers: {
          Authorization: `Bearer ${API_TOKEN}`,
        },
        method: "DELETE",
      })
        .then((response) => response.json())
        .then((data) => {})
        .catch((error) => {
          console.log(error);
        });
    },
    // Modify le raccourci
    updateRaccourci(id) {
      fetch(`https://api.airtable.com/v0/${BASE_ID}/${TABLE_NAME}/${id}`, {
        headers: {
          Authorization: `Bearer ${API_TOKEN}`,
          "Content-Type": "application/json",
        },
        method: "PATCH",
        body: JSON.stringify({
          fields: {
            Background: this.input_background,
            New_racc: this.input_new_racc,
            Modify_racc: this.input_modify_racc,
          },
        }),
      })
        .then((response) => response.json())
        .then((data) => {
          this.handleResetForm();
          this.edit = false;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    handleUpdate(raccourci) {
      this.input_background = raccourci.fields.Background;
      this.input_new_racc = raccourci.fields.New_racc;
      this.input_modify_racc = raccourci.fields.Modify_racc;
      this.selectedId = raccourci.id;
      this.edit = true;
    },
  },

  // Execution de base
  mounted() {
    this.getRaccourcis();
  },
  watch: {
    edit() {
      this.getRaccourcis();
    },
  },
};
</script>

<style scoped>
/* Style buttons */
button {
  background-color: #4caf50;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  margin: 1%;
}

button:hover {
  background-color: #45a049;
}
/* Styles pour le formulaire */
form {
  max-width: 400px; /* Ajustez la largeur maximale selon vos besoins */
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

/* Styles pour les étiquettes */
label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
}

/* Styles pour les champs de saisie */
input[type="url"] {
  width: 100%;
  padding: 8px;
  margin-bottom: 16px;
  box-sizing: border-box;
  border: 1px solid #ccc;
  border-radius: 4px;
}

/* Styles pour le bouton de soumission */
input[type="submit"] {
  background-color: #4caf50;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

input[type="submit"]:hover {
  background-color: #45a049;
}

/* Styles pour rendre le formulaire responsive */
@media screen and (max-width: 600px) {
  form {
    max-width: 100%;
  }
}
</style>
