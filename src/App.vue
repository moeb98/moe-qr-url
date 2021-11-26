<template>
  <div id="app">
    <div id="qrDisplayModal" class="modal" :hidden="!showQrDisplay">
      <div class="close-modal" @click="showQrDisplay = !showQrDisplay">X</div>
      <canvas id="qrDisplay"></canvas>
      <center>
        <pre>{{contactToDisplay}}</pre>
        <input type="button" id="downloadButton" @click="downloadQrCode()" value="Download" />
      </center>
    </div>
    <div id="header">
      Your URL to encode
    </div>

    <!-- own profile -->
    <div id="profile">
      <h6>Name</h6>
      <input type="text" placeholder="name" v-model="profile.name" />
      <h6>URL</h6>
      <input type="text" placeholder="url" v-model="profile.url" />
      <canvas id="myQrDisplay"></canvas>
    </div>

    <!-- toolbar -->
    <div id="toolbar">
      <button @click="saveQrCode">Save QR code</button>
    </div>

    <!-- list of contacts -->
    <div id="contacts">
      <h3>URLs</h3>
      <div id="no-contacts" v-if="contacts.length == 0">No URLs yet.</div>

      <div class="contact" v-for="c in contacts" :key="c.id">
        <div class="contact-name">{{c.name}}</div>
        <div class="contact-url"><a :href="c.url">{{c.url}}</a></div>
        <div class="contact-toolbar">
          <button @click="displayQrCode(c)">Share URL</button>
          <button @click="clickDelete(c.id)">Delete URL</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import QRCode from 'qrcode';
import md5 from 'md5';
import profileService from '@/services/profileService';
import contactService from '@/services/contactService';

export default {
  data() {
    return {
      // boolean variables to keep track of pop-up windows being shown
      showQrDisplay: false,
      showQrReader: false,

      // MD5 hash of the contact currently shown (only for validation)
      contactToDisplay: null,

      // own profile
      profile: {
        name: null,
        url: null,
      },

      // list of scanned contacts
      contacts: [],
    };
  },

  methods: {
    // object-destructuring to add functions of our services in 'methods'
    ...contactService,
    ...profileService,

    // load data from the contact and profile services into the view
    loadData() {
      this.contacts = this.getContacts().reverse();
      this.profile = this.getProfile();
    },

    downloadQrCode() {
      const myCanvas = document.getElementById('qrDisplay');
      const img = document.getElementById('qrDisplay').getElementsByTagName('img');
      img.src = myCanvas.toDataURL('image/jpg');
      const a = document.createElement('a');
      // Set the href attribute of the a tag (turn the canvas into a png image)
      a.href = img.src;
      a.download = 'qrcode';
      a.click();
    },

    // delete contact via our service
    clickDelete(contactId) {
      this.deleteContact(contactId);
      this.loadData();
    },

    // read QR code
    async saveQrCode() {
      this.createContact(this.getProfile());
      this.loadData();
    },

    // show QR code for url
    displayQrCode(contact) {
      contact = JSON.stringify({
        // name: contact.name,
        url: contact.url,
      });

      const qrDisplay = document.getElementById('qrDisplay');

      // show contact as QR code via the JSON string
      QRCode.toCanvas(qrDisplay, contact);
      this.contactToDisplay = md5(contact);
      this.showQrDisplay = true;
    },

    displayMyQrCode() {
      const myQrDisplay = document.getElementById('myQrDisplay');
      QRCode.toCanvas(myQrDisplay, JSON.stringify(this.profile.url));
    },
  },

  watch: {
    // whenever profile changes, save/display those changes
    profile: {
      handler() {
        this.setProfile(this.profile);
        this.displayMyQrCode();
      },
      deep: true,
    },
  },

  // load data when opening the page 
  mounted() {
    this.loadData();
  },
};
</script>

<style lang="scss">
* {
  box-sizing: border-box;
}

body,
html {
  padding: 0;
  margin: 0;
  height: 100%;
  font-family: Verdana, sans-serif;
  background-color:black;
  color: white;
}

#app {
  max-width: 450px;
  height: 100%;
  overflow: auto;
  margin: auto;
  background-color: #444;
  padding: 40px;

  .modal {
    position: fixed;
    background-color: #222;
    top: 0px;
    width: 100%;
    margin-left: -80px;
    max-width: 450px;
    height: 100%;
    .close-modal {
      cursor: pointer;
      padding: 10px;
      text-align: right;
      font-weight: bold;
      font-size: 24px;
    }
  }

  #toolbar {
    display: flex;
    justify-content: space-around;
    button {
      padding: 10px;
      margin: 10px;
    }
  }

  #qrDisplayModal {
    canvas {
      width: 100% !important;
      height: auto !important;
    }
  }

  #header {
    border: 1px solid #7a7a7a;
    width: 100% !important;
    height: 50px;
    background-color: #222;
    margin-top: -30px;
    margin-bottom: 20px;
    padding: 15px;
    text-align: center;
    font-weight: bold;
  }

  #profile {
    h3 {
      color: white;
    }
    h6 {
      margin: 10px 0px 0px 0px;
      font-size: 0.9em;
    }
    input {
      width: 100%;
      padding: 5px;
      margin: 5px 0;
      font-size: 0.85em;
    }

    #myQrDisplay {
      margin: 25px;
      width: calc(100% - 50px) !important;
      height: auto !important;
    }
  }

  #contacts {
    h3 {
      color: white;
    }
    #no-contacts {
      text-align: center;
      font-style: italic;
      margin: 50px 0;
    }
    .contact {
      border: 1px solid #7a7a7a;
      background-color: #222;
      padding: 10px;
      margin: 10px 0;
      line-height: 1.15;

      .contact-name {
        font-weight: bold;
        margin-bottom: 5px;
      }
      .contact-url {
        // font-style: italic;
        font-size: 0.85em;
        display:inline-block;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
        max-width: 36ch;
      }
      .contact-toolbar {
        display: flex;
        justify-content: space-around;
        margin-top: 10px;
        button {
          width: 45%;
        }
      }
    }
  }
}
</style>
