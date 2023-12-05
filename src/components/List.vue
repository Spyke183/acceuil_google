<template>
  <h1>Liste</h1>
  <div class="favoris">
    <div v-for="contact in contacts" :key="contact.id" class="container">
      <div class="Affichage">
        <!-- Bouton pour ouvrir le menu d'actions -->
        <button1 class="modif" @click="selectedContact = contact">Modifier</button1>
        <div class="link">
  <a :href="contact.fields.Lien" target="_blank">
    <div class="card_logo">
      <div class="logo">
        <img :src="contact.fields.Image" alt="Image du contact" />
      </div>
    </div>
    <div class="nom">
      {{ contact.fields.Nom }} 
    </div>
  </a>
</div>


        <!-- Menu d'actions -->
        <div class="action-menu" v-if="selectedContact === contact">
          <button class="action-item" @click="handleUpdate(contact)">Modifier le raccourci</button>
          <button class="action-item" @click="deleteContact(contact.id)">Supprimer</button>
        </div>
      </div>
    </div>
  </div>
  <form @submit.prevent="updateContact(selectedId)">
    <!-- Modale pour la modification des contacts -->
    <div v-if="showModal" class="modal-overlay" @click.self="closeModal">
    <div class="modal-content">
      <h3>Modifier le raccourci</h3>
      <form @submit.prevent="updateContact(selectedId)">
        <div class="form-group">
          <label for="nom">Nom</label>
          <input type="text" id="nom" v-model="selectedContact.fields.Nom" />
        </div>
        <div class="form-group">
          <label for="url">URL</label>
          <input type="text" id="url" v-model="selectedContact.fields.URL" />
        </div>
        <div class="modal-actions">
          <button type="button" @click="closeModal">Annuler</button>
          <button type="submit">OK</button>
        </div>
      </form>
    </div>
  </div></form>

  <!-- Formulaire pour modifier ou créer un contact -->
  <div v-if="edit" class="contact-form">
    <label for="nom">Nom</label>
    <input type="text" name="nom" id="nom" v-model="nom" />
    <label for="lien">Lien</label>
    <input type="lien" name="lien" id="lien" v-model="lien" />
    <button @click="updateContact(selectedId)">Modifier</button>
    <button @click="edit = false">Annuler</button>
  </div>
  <button v-else @click="createContact">Créer un contact</button>
</template>

<script>
const BASE_ID = import.meta.env.VITE_BASE_ID;
const TABLE_NAME = "CRM";
const VIEW_NAME = "Grid view";
const API_TOKEN = import.meta.env.VITE_TOKKEN;

export default {
  data() {
    return {
      contacts: [],
      nom: "",
      lien: "",
      image: "",
      selectedId: null,
      edit: false,
      showModal: false,
      selectedContact: null,
    };
  },
  methods: {
    handleResetForm() {
      this.nom = "";
      this.lien = "";
      this.image = "";
    },
    openModal(contact) {
      this.selectedContact = contact;
      this.showModal = true;
    },
    closeModal() {
      this.showModal = false;
    },
    openMenu(contact) {
      this.selectedContact = contact;
    },
    getContacts() {
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
          this.contacts = data.records;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    deleteContact(id) {
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
    updateContact(id) {
      fetch(`https://api.airtable.com/v0/${BASE_ID}/${TABLE_NAME}/${id}`, {
        headers: {
          Authorization: `Bearer ${API_TOKEN}`,
          "Content-Type": "application/json",
        },
        method: "PATCH",
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
          this.edit = false;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    handleUpdate(contact) {
      this.nom = contact.fields.Nom;
      this.lien = contact.fields.Lien;
      this.image = contact.fields.Image;
      this.selectedId = contact.id;
      this.edit = true;
      this.openModal(contact);
    },
    createContact() {
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
        })
        .catch((error) => {
          console.log(error);
        });
    },
    openModal(contact) {
      this.selectedContact = contact;
      this.showModal = true;
    },
    closeModal() {
      this.showModal = false;
    },
    closeActionMenu(event) {
      // Vérifie si le clic s'est produit en dehors du menu d'actions
      if (this.selectedContact && (!event.target.closest('.action-menu') && !event.target.closest('.modif'))) {
        this.selectedContact = null;
      }
    },
  },
  mounted() {
    this.getContacts();
    window.addEventListener('click', this.closeActionMenu);
  },
  watch: {
    edit() {
      this.getContacts();
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
}
a{
  text-decoration: none;
    color: black;
}
.nom{
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
.Affichage {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.card_logo {
  background-color: black;
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
  padding: 10px 0px 10px 0px;
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
button1 {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  background-color: #f1f1f1;
  /* Ajoutez d'autres styles de boutons ici */
}

button:hover {
  background-color: #e1e1e1;
}
</style>
