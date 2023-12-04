<template>
  <h1>Liste</h1>
  <div class="favoris">
    <div v-for="contact in contacts" :key="contact.id" class="container">
      <div class="Affichage">
        <!-- Bouton pour ouvrir le menu d'actions -->
        <div class="modif" @click="selectedContact = contact">+</div>
        <div class="card_logo">
          <div class="logo">
            <img :src="contact.fields.Image" alt="Image du contact" />
          </div>
        </div>
        <div class="nom">
          {{ contact.fields.Nom }} {{ contact.fields.Prenom }}
        </div>
        <!-- Menu d'actions -->
        <div class="action-menu" v-if="selectedContact === contact">
          <button class="action-item" @click="handleUpdate(contact)">Modifier le raccourci</button>
          <button class="action-item" @click="deleteContact(contact.id)">Supprimer</button>
        </div>
      </div>
    </div>
  </div>

  <!-- Formulaire pour modifier ou créer un contact -->
  <div v-if="edit" class="contact-form">
    <label for="nom">Nom</label>
    <input type="text" name="nom" id="nom" v-model="nom" />
    <label for="prenom">Prénom</label>
    <input type="text" name="prenom" id="prenom" v-model="prenom" />
    <label for="email">Email</label>
    <input type="email" name="email" id="email" v-model="email" />
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
      prenom: "",
      email: "",
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
      this.prenom = "";
      this.email = "";
      this.image = "";
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
            Prenom: this.prenom,
            Email: this.email,
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
      this.prenom = contact.fields.Prenom;
      this.email = contact.fields.Email;
      this.image = contact.fields.Image;
      this.selectedId = contact.id;
      this.edit = true;
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
            Prenom: this.prenom,
            Email: this.email,
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
  background-color: #000000;
  box-shadow: 0 2px 5px rgba(0,0,0,0.2);
  border-radius: 4px;
  z-index: 10;
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
</style>
