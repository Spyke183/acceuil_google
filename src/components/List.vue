<template>
  <div class="Title">
    <span>G</span><span>o</span><span>o</span><span>g</span><span>l</span><span>e</span>
  
  </div>
  <div class="btn-fav">  <button @click="openCreateModal">Créer un favori</button>
  </div>
    <div class="favoris">
    <div v-for="favori in displayedFavoris" :key="favori.id" class="container">
      <div class="Affichage">
        <button class="modif" @click="selectedFavori = favori">Modifier</button>
        <div class="link">
          <a :href="favori.fields.Lien" target="_blank">
            <div class="card_logo">
              <div class="logo">
                <img :src="favori.fields.Image" alt="Image du favori" />
              </div>
            </div>
            <div class="nom">
              {{ favori.fields.Nom.slice(0, 10) }} 
            </div>
          </a>
        </div>
        <div class="action-menu" v-if="selectedFavori === favori">
          <div class="close-button" @click="selectedFavori = null">✖</div>
          <button class="action-item" @click="handleUpdate(favori)">Modifier le raccourci</button>
          <button class="action-item" @click="deleteFavori(favori.id)">Supprimer</button>

        </div>
      </div>
    </div>
  </div>
  <div class="favoris2" v-if="additionalFavoris.length">
    <div v-for="favori in additionalFavoris" :key="favori.id" class="container">
      <div class="Affichage">
        <button class="modif" @click="selectedFavori = favori">Modifier</button>
        <div class="link">
          <a :href="favori.fields.Lien" target="_blank">
            <div class="card_logo">
              <div class="logo">
                <img :src="favori.fields.Image" alt="Image du favori" />
              </div>
            </div>
            <div class="nom">
              {{ favori.fields.Nom }} 
            </div>
          </a>
        </div>
        <div class="action-menu" v-if="selectedFavori === favori">
          <div class="close-button" @click="selectedFavori = null">✖</div>

          <button class="action-item" @click="handleUpdate(favori)">Modifier le raccourci</button>
          <button class="action-item" @click="deleteFavori(favori.id)">Supprimer</button>

        </div>
      </div>
    </div>
  </div>
  
    <form @submit.prevent="updateFavori(selectedId)">
      <!-- Modale pour la modification des Favoris -->
      <div v-if="showModal" class="modal-overlay" @click.self="closeModal">
      <div class="modal-content">
        <h3>Modifier le raccourci</h3>
        <form @submit.prevent="updateFavori(selectedId)">
          <div class="form-group">
            <label for="nom">Nom</label>
            <input type="text" id="nom" v-model="selectedFavori.fields.Nom" maxlength="10" />
          </div>
          <div class="form-group">
            <label for="lien">Lien</label>
            <input type="text" id="lien" v-model="selectedFavori.fields.Lien" />
          </div>
          <div class="modal-actions">
            <button type="button" @click="closeModal">Annuler</button>
            <button type="submit">OK</button>
          </div>
        </form>
      </div>
    </div></form>
  
  <!-- Modal pour la création d'un nouveau Favori -->
  <div v-if="showCreateModal" class="modal-overlay" @click.self="closeCreateModal">
    <div class="modal-content">
      <h3>Créer un nouveau favori</h3>
      <form @submit.prevent="createNewFavori">
        <div class="form-group">
          <label for="newNom">Nom</label>
          <input type="text" id="newNom" v-model="newFavori.Nom" />
        </div>
        <div class="form-group">
          <label for="newLien">Lien</label>
          <input type="text" id="newLien" v-model="newFavori.Lien" />
        </div>
        <div class="form-group">
          <label for="newImage">Image URL</label>
          <input type="text" id="newImage" v-model="newFavori.Image" />
        </div>
        <div class="modal-actions">
          <button type="button" @click="closeCreateModal">Annuler</button>
          <button type="submit">Créer</button>
        </div>
      </form>
    </div>
  </div>
    <!-- Bouton pour ouvrir le menu de personnalisation -->
    <div class="customization-menu">
      <button @click="toggleCustomizationMenu">Personnaliser</button>
      <div v-if="showCustomizationMenu" class="menu">
        <div class="color-picker">
          <label for="bg-color">Couleur de fond:</label>
          <input type="color" id="bg-color" v-model="backgroundColor" @change="changeBackgroundColor">
        </div>
        <div class="image-picker">
          <label for="bg-image">Image de fond (URL):</label>
          <input type="text" id="bg-image" v-model="backgroundImage" @change="changeBackgroundImage">
        </div>
      </div>
    </div>
  </template>
  
  <script>
  const BASE_ID = import.meta.env.VITE_BASE_ID;
  const TABLE_NAME = "CRM";
  const VIEW_NAME = "Grid view";
  const API_TOKEN = import.meta.env.VITE_TOKKEN;
  
  export default {
    data() {
      return {
        Favoris: [],
        displayedFavoris: [],
        additionalFavoris: [],
        nom: "",
        lien: "",
        image: "",
        selectedId: null,
        edit: false,
        showModal: false,
        selectedFavori: null,
        showCreateModal: false,
        newFavori: { Nom: '', Lien: '', Image: '' },
        showCustomizationMenu: false,
        backgroundColor: '#FFFFFF',
        backgroundImage: '',
      };
    },
    methods: {
      handleResetForm() {
        this.nom = "";
        this.lien = "";
        this.image = "";
      },
      toggleCustomizationMenu() {
        this.showCustomizationMenu = !this.showCustomizationMenu;
      },
      changeBackgroundColor() {
        document.body.style.backgroundColor = this.backgroundColor;
      },
      changeBackgroundImage() {
        document.body.style.backgroundImage = `url(${this.backgroundImage})`;
        document.body.style.backgroundSize = 'cover';
      },
      openModal(favori) {
        this.selectedFavori = favori;
        this.showModal = true;
      },
      closeModal() {
        this.showModal = false;
      },
      openMenu(favori) {
        this.selectedFavori = favori;
      },
      getFavoris() {
        fetch(`https://api.airtable.com/v0/${BASE_ID}/${TABLE_NAME}?view=${VIEW_NAME}`, {
          headers: {
            Authorization: `Bearer ${API_TOKEN}`,
          },
        })
        .then((response) => response.json())
        .then((data) => {
          this.Favoris = data.records;
          this.displayedFavoris = this.Favoris.slice(0, 5);
          this.additionalFavoris = this.Favoris.slice(5, 10);
        })
        .catch((error) => {
          console.log(error);
        });
      },
      deleteFavori(id) {
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
      updateFavori() {
    fetch(`https://api.airtable.com/v0/${BASE_ID}/${TABLE_NAME}/${this.selectedId}`, {
      headers: {
        Authorization: `Bearer ${API_TOKEN}`,
        "Content-Type": "application/json",
      },
      method: "PATCH",
      body: JSON.stringify({
        fields: {
          Nom: this.selectedFavori.fields.Nom,
          Lien: this.selectedFavori.fields.Lien,
          Image: this.selectedFavori.fields.Image,
        },
      }),
    })
    .then(response => response.json())
    .then(data => {
    this.getFavoris();  // Cette méthode doit recharger les favoris depuis l'API
    this.showModal = false;
  })
    .catch(error => {
      console.error("Erreur lors de la mise à jour du favori:", error);
    });
  },
      handleUpdate(favori) {
    this.selectedFavori = JSON.parse(JSON.stringify(favori));
    this.selectedId = favori.id;
    this.showModal = true;
  },
      createFavori() {
        // Logique pour la création de Favori, limitée à 10
        if (this.Favoris.length < 10) {
          fetch(`https://api.airtable.com/v0/${BASE_ID}/${TABLE_NAME}`, {
            headers: {
              Authorization: `Bearer ${API_TOKEN}`,
              "Content-Type": "application/json",
            },
            method: "POST",
            body: JSON.stringify({
              fields: {
                Nom: this.nom,
                Lien: this.lien,
                Image: this.image,
              },
            }),
          })
          .then((response) => response.json())
          .then((data) => {
            this.handleResetForm();
            this.Favoris.push(data.record);
            this.displayedFavoris = this.Favoris.slice(0, 5);
            this.additionalFavoris = this.Favoris.slice(5, 10);
          })
          .catch((error) => {
            console.log(error);
          });
        } else {
          alert("La limite de Favoris a été atteinte.");
        }
      },
      openModal(favori) {
        this.selectedFavori = favori;
        this.showModal = true;
      },
      closeModal() {
        this.showModal = false;
      },
      closeActionMenu(event) {
        // Vérifie si le clic s'est produit en dehors du menu d'actions
    if (this.selectedFavori && (!event.target.closest('.action-menu') && !event.target.closest('.modif'))) {
      this.selectedFavori = null;
        }
      },
      openCreateModal() {
        this.showCreateModal = true;
      },
  
      closeCreateModal() {
        this.showCreateModal = false;
      },
      createNewFavori() {
      // Vérification des données du nouveau Favori
      if (!this.newFavori.Nom || !this.newFavori.Lien) {
        alert("Veuillez remplir tous les champs requis.");
        return;
      }
  
      // Préparation de la requête POST
      const newFavoriData = {
        fields: {
          Nom: this.newFavori.Nom,
          Lien: this.newFavori.Lien,
          Image: this.newFavori.Image
        }
      };
  
      // Envoi de la requête à l'API
      fetch(`https://api.airtable.com/v0/${BASE_ID}/${TABLE_NAME}`, {
        headers: {
          Authorization: `Bearer ${API_TOKEN}`,
          "Content-Type": "application/json"
        },
        method: "POST",
        body: JSON.stringify(newFavoriData)
      })
      .then(response => response.json())
      .then(data => {
        // Ajout du nouveau Favori à la liste des Favoris
        this.Favoris.push(data.record);
        // Réinitialisation des champs du formulaire
        this.newFavori = { Nom: '', Lien: '', Image: '' };
        // Fermeture du modal
        this.showCreateModal = false;
        window.location.reload();

      })
      .catch(error => {
        console.error("Erreur lors de la création du favori:", error);
        alert("Erreur lors de la création du favori.");
      });
    },
    },
    mounted() {
      this.getFavoris();
      window.addEventListener('click', this.handleOutsideClick);
  
    },
    watch: {
      edit() {
        this.getFavoris();
      },
      beforeDestroy() {
      // Assurez-vous de nettoyer l'écouteur d'événements lorsque le composant est détruit
      window.removeEventListener('click', this.closeActionMenu);
    },
    },
  };
  </script>
  
  <style>
  :root {
    --icon-size: 24px;
  }
  .container{
    padding: 1em;
    background-color: white;
    margin: 10px;
    border-radius: 8px;
  }
  a{
    text-decoration: none;
      color: black;
      display: flex;
      flex-direction: column;
      align-items: center;
  }
  .nom{
    text-align: center;
    margin: 8px;
  }
  .modif{
    padding: 10px 20px;
    margin: 7px;
    background-color: #f1f1f1;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  .modif:hover {
    background-color: #e1e1e1; 
  }
  
  .link{
    display: flex;
      flex-direction: column;
      align-items: center;
  }
  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
  }
  
  .modal-content {
    background: #fff;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    width: 400px; /* ou la largeur que vous préférez */
  }
  
  .form-group {
    margin-bottom: 20px;
    display: flex;
    flex-direction: column;
  }
  
  .modal-actions {
    text-align: right;
  }
  
  .modal-actions  {
    margin-left: 10px;
  }
  .favoris {
    display: flex;
    flex-direction: row;
    justify-content: center;
  }
  .favoris2 {
    display: flex;
    flex-direction: row;
    justify-content: center;
  }
  .Affichage {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  
  .card_logo {
    background-color: #dbdbdb;
    border-radius: 50%;
    overflow: hidden;
    height: calc(var(--icon-size) + 16px);
    width: calc(var(--icon-size) + 16px);
    margin: 8px 15px 0px 15px;;
  }
  
  .logo {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
  }
  
  .logo img {
    height: var(--icon-size);
    width: var(--icon-size);
    object-fit: cover;
    border-radius: 50%;
  }
  
  .action-menu {
    display: flex;
    flex-direction: column;
    position: absolute;
    background-color: #d1d1d1;
    box-shadow: 0 2px 5px rgba(0,0,0,0.2);
    border-radius: 4px;
    z-index: 10;
    padding: 23px 0px 10px 0px;
  }
  
  .action-item {
    display: block;
    padding: 10px;
    cursor: pointer;
    border: none;
    transition: background-color 0.3s;
    background-color: #d1d1d1;
  }
  
  .action-item:hover {
    background-color: #a3a3a3;
  }
  button {
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    background-color: #f1f1f1;
    margin-left: 5px;
    /* Ajoutez d'autres styles de boutons ici */
  }
  
  button:hover {
    background-color: #e1e1e1;
  }
  .customization-menu {
      position: fixed;
      top: 10px;
      right: 10px;
      z-index: 1000;
    }
  
    .menu {
      background: #f1f1f1;
      padding: 10px;
      border: 1px solid #ccc;
      border-radius: 5px;
      margin-top: 5px;
    }
  
    .color-picker, .image-picker {
      margin-bottom: 10px;
    }
    .Title{
      display: flex;
      justify-content: center;
      font-size: 10em;
      font-weight: bold;
      padding-top: 0.5em;
    }
  .Title span:nth-child(1) { color: #4285F4; }
  .Title span:nth-child(2) { color: #DB4437; }
  .Title span:nth-child(3) { color: #F4B400; }
  .Title span:nth-child(4) { color: #4285F4; }
  .Title span:nth-child(5) { color: #0f9d58; }
  .Title span:nth-child(6) { color: #DB4437; }
  
  .btn-fav{
    display: flex;
      justify-content: center;
      padding: 2em;
  }
  .close-button {
  position: absolute;
  top: 5px;
  right: 5px;
  cursor: pointer;
  font-size: 1.2em;
  color: #555;
}
  </style>