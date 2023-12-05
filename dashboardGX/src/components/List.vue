<template>
  <div
    class="background-container"
    :style="{ backgroundImage: 'url(' + backgroundImageUrl + ')' }"
  >
    <div class="area_titles">
      <h1>DashboardGX</h1>
    </div>
    <button
      class="btn_principal"
      @click="toggleModalBackground"
      id="btn_open_modal_input_background"
    >
      Changer le background
    </button>
    <button
      class="btn_principal"
      @click="toggleModalNewRacc"
      id="btn_open_modal_input_new_racc"
    >
      Ajouter un raccourci
    </button>
    <ul>
      <li v-for="raccourci in raccourcis" :key="raccourci.id">
        <a
          :style="{ backgroundColor: raccourci.fields.Change_color }"
          v-if="
            raccourci &&
            raccourci.fields &&
            raccourci.fields.Title_racc &&
            raccourci.fields.New_racc
          "
          :href="raccourci.fields.New_racc"
          target="_blank"
          rel="noopener noreferrer"
        >
          {{ raccourci.fields.Title_racc }}
        </a>
        <div class="area_btns_mod_sup">
          <button
            class="btns_mod_sup"
            @click="toggleModalModifyRacc(raccourci)"
          >
            ⚙️
          </button>
          <button class="btns_mod_sup" @click="deleteRaccourci(raccourci.id)">
            ❌
          </button>
        </div>
      </li>
    </ul>

    <!-- FORM NEW RACC -->
    <!-- Button open modal -->

    <!-- From changement background -->
    <div v-if="displayModalBackground" class="area_modal">
      <form @submit.prevent="changeBackground">
        <label for="input_background"
          >Entrez l'url de l'image background :</label
        >
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

    <!-- Form ajouter une url -->
    <div v-if="displayModalNewRacc" class="area_modal">
      <form @submit.prevent="createRaccourci">
        <label for="input_title_racc">Entrez le titre :</label>
        <input
          type="text"
          id="input_title_racc"
          name="input_title_racc"
          v-model="input_title_racc"
          required
        />
        <label for="input_new_racc">Entrez une URL :</label>
        <input
          type="url"
          id="input_new_racc"
          name="input_new_racc"
          v-model="input_new_racc"
          required
        />
        <label for="input_change_color">Choisissez une couleur :</label>
        <input
          type="color"
          id="input_change_color"
          name="input_change_color"
          v-model="input_change_color"
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

    <!-- Form modifier le raccourci -->
    <div v-if="displayModalModifyRacc" class="area_modal">
      <form @submit.prevent="updateRaccourci">
        <label for="input_title_racc">Entrez nouveau titre :</label>
        <input
          type="text"
          id="input"
          name="input_title_racc"
          v-model="input_title_racc"
          required
        />
        <label for="input_new_racc">Entrez nouvelle URL :</label>
        <input
          type="url"
          id="input_new_racc"
          name="input_new_racc"
          v-model="input_new_racc"
          required
        />
        <label for="input_change_color">Choisissez nouvelle couleur :</label>
        <input
          type="color"
          id="input_change_color"
          name="input_change_color"
          v-model="input_change_color"
          required
        />
        <input type="submit" value="Modifier" />
      </form>
    </div>

    <button v-if="edit" @click="updateRaccourci(selectedId)">Modifier</button>
    <button v-if="edit" @click="edit = false">Annuler</button>
  </div>
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
      input_title_racc: "",
      input_new_racc: "",
      input_change_color: "",
      input_modify_racc: "",
      selectedId: null,
      edit: false,
      displayModalBackground: false,
      displayModalNewRacc: false,
      displayModalModifyRacc: false,
      backgroundImageUrl: "../../img/background.jpeg", //image par défaut
    };
  },
  methods: {
    handleResetForm() {
      this.input_background = "";
      this.input_new_racc = "";
      this.input_change_color = "";
      this.input_title_racc = "";
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
          console.log("URL envoyée : ", this.input_new_racc);
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
            Title_racc: this.input_title_racc,
            New_racc: this.input_new_racc,
            Change_color: this.input_change_color,
          },
        }),
      })
        .then((response) => response.json())
        .then((data) => {
          this.raccourcis.push(data); //Evite le rafraichissement de la page
          this.handleResetForm();
        })
        .catch((error) => {
          console.log(error);
        });
    },
    // Supprimer le raccourci
    deleteRaccourci(id) {
      console.log(id);
      fetch(`https://api.airtable.com/v0/${BASE_ID}/${TABLE_NAME}/${id}`, {
        headers: {
          Authorization: `Bearer ${API_TOKEN}`,
        },
        method: "DELETE",
      })
        .then((response) => response.json())

        .then((data) => {
          //Evite le rafraichissement de la page
          this.raccourcis = this.raccourcis.filter(
            (raccourcis) => raccourcis.id !== id
          );
        })
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
            Title_racc: this.input_title_racc,
            New_racc: this.input_new_racc,
            Change_color: this.input_change_color,
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
      this.input_title_racc = raccourci.fields.Title_racc;
      this.input_new_racc = raccourci.fields.New_racc;
      this.input_change_color = raccourci.fields.Change_color;
      this.input_modify_racc = raccourci.fields.Modify_racc;
      this.selectedId = raccourci.id;
      this.edit = true;
    },
    getBackgroundImage() {
      fetch(`https://api.airtable.com/v0/${BASE_ID}/Background`, {
        headers: {
          Authorization: `Bearer ${API_TOKEN}`,
        },
      })
        .then((response) => response.json())
        .then((data) => {
          this.raccourcis = data.records;
          localStorage.setItem("backgroundImageUrl", this.backgroundImageUrl); // stock dans le local storage
        });
    },
    changeBackground() {
      fetch(`https://api.airtable.com/v0/${BASE_ID}/Background`, {
        headers: {
          Authorization: `Bearer ${API_TOKEN}`,
          "Content-Type": "application/json",
        },
        method: "PATCH",
        body: JSON.stringify({
          fields: {
            Background: this.input_background,
          },
        }),
      })
        .then((response) => response.json())
        .then((data) => {
          this.backgroundImageUrl = this.input_background; // Mettez à jour l'URL de fond dans le composant

          this.handleResetForm();
        })
        .catch((error) => {
          console.log(error);
        });
    },
    displayTitleRacc() {
      if (raccourci.fields.New_racc.substring(0, 5) == "https") {
        return raccourci.fields.New_racc.substring(12, 13);
      } else {
        return raccourci.fields.New_racc.substring(11, 12);
      }
    },
  },

  // Execution de base
  mounted() {
    this.getRaccourcis();
    this.getBackgroundImage();
  },
  watch: {
    edit() {
      this.getRaccourcis();
    },
  },
};
</script>

<style scoped>
.area_titles {
  text-align: center;
  color: white;
  font-size: 1.5vw;
}
.area_btns_mod_sup {
  display: flex;
}
.background-container {
  /* Assurez-vous que le fond s'étend sur toute la page */
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1; /* Placez-le derrière le contenu */

  /* Assurez-vous que l'image de fond est responsive */
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;

  /* Ajoutez d'autres styles au besoin */
}
/*MODAL*/
.area_modal {
  position: absolute;
  display: flex;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

/* Style buttons */
.btn_principal {
  background-color: #59038a;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  margin: 1%;
}

button:hover {
  background-color: #370254;
}

.btns_mod_sup {
  background-color: transparent;
  border: none;
  margin: 1%;
  font-size: 1.5vw;
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
  background-color: #59038a;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

input[type="submit"]:hover {
  background-color: #370254;
}

label {
  font-size: 1vw;
  color: white;
  margin-top: 2%;
}

ul {
  display: flex;
  justify-content: space-around;
}

li {
  list-style: none;
  display: flex;
}

/* Affichage titles raccourcis */
a {
  font-size: 1.5vw;
  text-decoration: none;
  color: white;
  padding: 10%;
  border-radius: 5%;
}
a:hover {
  color: rgb(206, 196, 196);
}

/* Styles pour rendre le formulaire responsive */
@media screen and (max-width: 600px) {
  form {
    max-width: 100%;
  }
}
</style>
