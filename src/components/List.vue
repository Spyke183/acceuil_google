<template>
  <h1>Liste</h1>
  <div class="favoris">
    <div v-for="contact in contacts" :key="contact.id" class="container">
      <div class="Affichage">
        <div class="modif" @click="openModal(contact)">+</div>
        <div class="card_logo">
          <div class="logo">
            <img :src="contact.fields.Image" alt="Image du contact" />
          </div>
        </div>
        <div class="nom">
          {{ contact.fields.Nom }} {{ contact.fields.Prenom }}
        </div>
      </div>
    </div>
  </div>

  <label for="nom">Nom</label>
  <input type="text" name="nom" id="nom" v-model="nom" />
  <label for="prenom">Prénom</label>
  <input type="text" name="prenom" id="prenom" v-model="prenom" />
  <label for="email">Email</label>
  <input type="email" name="email" id="email" v-model="email" />
  <button v-if="edit" @click="updateContact(selectedId)">Modifier</button>
  <button v-if="edit" @click="edit = false">Annuler</button>
  <button v-else @click="createContact">Créer un contact</button>

  <div class="modal" v-if="showModal">
    <div class="modal-content">
      <button @click="handleUpdate(selectedContact)">Modifier</button>
      <button @click="deleteContact(selectedContact.id)">Supprimer</button>
      <button @click="closeModal">Fermer</button>
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
  },
  mounted() {
    this.getContacts();
  },
  watch: {
    edit() {
      this.getContacts();
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
  margin-top: 16px;
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

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.24);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 10px;
}
</style>
