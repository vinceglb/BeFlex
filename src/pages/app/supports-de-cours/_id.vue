<template>
  <div>
    <v-file-input v-model="file" :loading="loading" label="File input" />
    <v-btn :loading="loading" :disabled="file === null" @click="upload(yo)"
      >Envoyer</v-btn
    >

    <v-progress-circular v-if="loading2" indeterminate></v-progress-circular>
    <h3 v-if="cards.length === 0 && !loading2">Aucun document</h3>

    <v-container>
      <v-row>
        <v-col v-for="card in cards" :key="card.title" class="d-flex" cols="12">
          <v-card class="flex-grow-1" ripple @click="downloadFile(card)">
            <v-card-text>
              <v-icon>mdi-xbox-controller</v-icon> {{ card.name }}
            </v-card-text>
          </v-card>
          <v-btn
            icon
            color="red"
            class="align-self-center ml-2"
            @click="deleteFile(card)"
          >
            <v-icon>mdi-delete</v-icon>
          </v-btn>
        </v-col>
      </v-row>
    </v-container>

    <!-- Snackbar -->
    <v-snackbar v-model="snackbarDisplay">
      {{ snackbarText }}
      <v-btn color="pink" text @click="snackbarDisplay = false"> Fermer </v-btn>
    </v-snackbar>
  </div>
</template>

<script>
export default {
  data() {
    return {
      file: null,
      loading: false,
      loading2: false,
      snackbarDisplay: false,
      snackbarText: 'null',
      cards: [],
    }
  },
  mounted() {
    this.getFiles()
  },
  methods: {
    upload() {
      if (this.file === null) {
        return
      }

      console.log('file', this.file)
      this.loading = true

      const storageRef = this.$fireStorage.ref()
      const sdcFolder = storageRef.child('sdc/' + this.$route.params.id)
      const fileName = this.file.name
      const docRef = sdcFolder.child(fileName)
      docRef
        .put(this.file)
        .then((snap) => {
          this.snackbarText = 'Ok, fichier envoyé ✔'
          this.snackbarDisplay = true
          console.log('Uploaded a blob or file! ✔', snap)
        })
        .catch((e) => {
          this.snackbarText = 'Une erreur est survenue'
          this.snackbarDisplay = true
          console.error("Erreur lors de l'upload", e)
        })
        .finally(() => {
          this.loading = false
          this.getFiles()
        })
    },
    getFiles() {
      console.log('ok')
      this.loading2 = true
      this.cards = []
      const storageRef = this.$fireStorage.ref()
      storageRef
        .child('sdc/' + this.$route.params.id)
        .listAll()
        .then((res) => {
          if (res.items.length === 0) {
            this.loading2 = false
          }

          res.items.forEach((itemRef) => {
            itemRef
              .getMetadata()
              .then((meta) => {
                console.log('item meta', meta)
                this.cards.push(meta)
              })
              .catch((e) => {
                console.error(
                  "Erreur lors de la récupération des metadata d'un fichier",
                  e
                )
              })
              .finally(() => {
                this.loading2 = false
              })
          })
        })
        .catch((e) => {
          console.error('Erreur lors de la récupération des fichiers', e)
          this.loading2 = false
        })
    },
    downloadFile(meta) {
      meta.ref.getDownloadURL().then((url) => {
        window.open(url, '_blank')
      })
    },
    deleteFile(meta) {
      meta.ref
        .delete()
        .then(() => {
          this.snackbarText = 'Fichier supprimé ✔'
          this.snackbarDisplay = true
        })
        .catch((error) => {
          this.snackbarText = 'Une erreur est survenue'
          this.snackbarDisplay = true
          console.error("Erreur lors de la suppression d'un fichier", error)
        })
        .finally(() => {
          this.getFiles()
        })
    },
  },
}
</script>

<style>
.yo {
  margin-bottom: 20px;
}
</style>
