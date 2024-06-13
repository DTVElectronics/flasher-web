<template>
  <q-page class="q-pa-md">
    <div class="q-pa-md q-gutter-sm absolute-center">
      <q-select outlined v-model="lang" :options="options" label="Language" />
      <q-input outlined v-model="ip" label="Board IP address" />
      <q-btn
        dark-percentage
        unelevated
        color="secondary"
        @click="openFirstModal()"
        icon="cloud_download"
        style="width: 250px; height: 50px"
      />

      <q-dialog v-model="dialog" persistent :maximized="maximizedToggle">
        <q-card class="bg-white text-primary">
          <q-bar class="bg-white">
            <q-space />
            <q-btn
              dense
              flat
              icon="minimize"
              @click="maximizedToggle = false"
              :disable="!maximizedToggle"
            >
              <q-tooltip
                v-if="maximizedToggle"
                content-class="bg-white text-primary"
                >Minimize</q-tooltip
              >
            </q-btn>
            <q-btn
              dense
              flat
              icon="crop_square"
              @click="maximizedToggle = true"
              :disable="maximizedToggle"
            >
              <q-tooltip
                v-if="!maximizedToggle"
                content-class="bg-white text-primary"
                >Maximize</q-tooltip
              >
            </q-btn>
            <q-btn dense flat icon="close" v-close-popup>
              <q-tooltip content-class="bg-white text-primary">Close</q-tooltip>
            </q-btn>
          </q-bar>

          <q-card-section
            class="text-center q-mt-xs text-body2 text-weight-bold text-primary text-uppercase"
          >
            {{ lang == "en" ? "Categories" : "KATEGORIEN" }}
          </q-card-section>

          <q-separator />

          <q-card-section>
            <q-list bordered separator>
              <q-item
                v-for="OS in OSLists"
                @click="openImagesModals(OS)"
                :key="OS"
                clickable
                v-ripple
              >
                <q-item-section top>
                  <q-item-label class="col-1 gt-sm">{{
                    OS.name[lang]
                  }}</q-item-label>
                </q-item-section>

                <q-item-section top>
                  <q-item-label class="col-1 gt-sm">{{
                    OS.description[lang]
                  }}</q-item-label>
                </q-item-section>
              </q-item>
            </q-list>
          </q-card-section>
        </q-card>
      </q-dialog>

      <q-dialog v-model="imagesModal" persistent>
        <q-card class="bg-white text-primary">
          <q-card-section
            class="text-center q-mt-xs text-body2 text-weight-bold text-primary text-uppercase"
          >
            {{ lang == "en" ? "Images" : "Bilder" }}
          </q-card-section>

          <q-separator />

          <q-card-section>
            <q-list bordered separator>
              <q-item
                v-for="imgs in imagesList"
                @click="openFinalModals(imgs)"
                :key="imgs"
                clickable
                v-ripple
              >
                <q-item-section top>
                  <q-item-label class="col-1 gt-sm">{{
                    imgs.name[lang]
                  }}</q-item-label>
                </q-item-section>

                <q-item-section top>
                  <q-item-label class="col-1 gt-sm">{{
                    imgs.description[lang]
                  }}</q-item-label>
                </q-item-section>
              </q-item>
            </q-list>
          </q-card-section>

          <q-card-actions align="right" class="bg-white text-primary">
            <q-btn flat label="Close" v-close-popup />
          </q-card-actions>
        </q-card>
      </q-dialog>

      <q-dialog v-model="osModal" persistent>
        <q-card
          class="bg-white text-primary"
          style="width: 500px; max-width: 80vw"
        >
          <q-card-section
            class="text-center q-mt-xs text-body2 text-weight-bold text-primary text-uppercase"
          >
            {{ lang == "en" ? "OS list" : "OS Liste" }}
          </q-card-section>

          <q-separator />

          <q-card-section>
            <q-list separator bordered class="rounded-borders">
              <q-item
                v-for="(image, imageName) in osImagesList"
                :key="imageName"
              >
                <q-item-section top>
                  <q-icon name="computer" color="black" size="30px" />
                </q-item-section>

                <q-item-section top>
                  <q-item-label lines="1" class="q-mt-sm">{{
                    imageName
                  }}</q-item-label>
                </q-item-section>

                <q-item-section top side>
                  <div class="text-grey-8 q-gutter-xs">
                    <q-btn
                      class="gt-xs"
                      size="15px"
                      flat
                      dense
                      round
                      icon="download_for_offline"
                      @click="flashImage(image.url, image.sha512)"
                    />
                  </div>
                </q-item-section>
              </q-item>
            </q-list>
          </q-card-section>
          <q-card-actions align="right" class="bg-white text-primary">
            <q-btn flat label="Close" v-close-popup />
          </q-card-actions>
        </q-card>
      </q-dialog>
      <q-dialog v-model="loggingModal">
        <q-card style="width: 600px; max-width: 80vw">
          <q-card-section>
            <q-input type="textarea" filled v-model="messages" readonly />
          </q-card-section>
          <q-card-actions align="right">
            <q-btn flat label="Close" color="primary" v-close-popup />
          </q-card-actions>
        </q-card>
      </q-dialog>
    </div>
  </q-page>
</template>

<script>
export default {
  data() {
    return {
      dialog: false,
      maximizedToggle: true,
      OSLists: [
        {
          name: {
            en: "General-purpose",
            de: "Allgemein",
          },
          description: {
            en: "General-purpose operating systems",
            de: "Allgemeine Betriebssysteme",
          },
          images: [
            {
              name: {
                en: "Debian GNU/Linux (CLI)",
                de: "Debian GNU/Linux (CLI)",
              },
              description: {
                en: "A free, community-driven operating system (Command Line Interface only)",
                de: "Ein freies, von der Community getriebenes Betriebssystem (nur Kommandozeile)",
              },
              images: {
                "radxa-cm3-rpi-cm4-io": {
                  url: "https://github.com/radxa-build/radxa-cm3-rpi-cm4-io/releases/download/b15/radxa-cm3-rpi-cm4-io_debian_bullseye_cli_b15.img.xz",
                  sha512:
                    "7d96dc52826134606b9c9727252570f2457e75ee5f3dc3546b98fc24d025b084e7736f6b9884986ad8d348935941739a93e699d6f0f5fe359270c7c034be52b3",
                },
                "radxa-cm3-sodimm-io": {
                  url: "https://github.com/radxa-build/radxa-cm3-sodimm-io/releases/download/b27/radxa-cm3-sodimm-io_debian_bullseye_cli_b27.img.xz",
                  sha512:
                    "e2ed35c78bb6806eaeba021ae560619dce9208b45b437daa1c1c8687d0b538729f471cb4fa246361598432c1209ee5b9a4016debadde6c80daa892cbf4c2665e",
                },
              },
            },
            {
              name: {
                en: "Nirvati",
                de: "Nirvati",
              },
              description: {
                en: "A free server management system",
                de: "Ein freies Server-Management-System",
              },
              images: {
                "radxa-cm3-rpi-cm4-io": {
                  url: "https://assets.nirvati.org/81e9e5c8/radxa-cm3-rpi-cm4-io_debian_bookworm_xfce.img.xz",
                  sha512:
                    "TODO // I haven't calculated the checksum here yet, it would be here in the final version",
                },
              },
            },
          ],
        },
        {
          name: {
            en: "Bitcoin",
            de: "Bitcoin",
          },
          description: {
            en: "Bitcoin-related operating systems",
            de: "Bitcoin-bezogene Betriebssysteme",
          },
          images: [
            {
              name: {
                en: "Citadel",
                de: "Citadel",
              },
              description: {
                en: "A completely FOSS Bitcoin full node stack",
                de: "Ein vollständig freier Bitcoin Full-Node Stack",
              },
              images: {
                "radxa-cm3-rpi-cm4-io": {
                  url: "https://assets.nirvati.org/4f8e77a6/radxa-cm3-rpi-cm4-io_debian_bookworm-test_xfce.img.xz",
                  sha512:
                    "TODO // I haven't calculated the checksum here yet, it would be here in the final version",
                },
              },
            },
          ],
        },
      ],
      lang: null,
      options: ["en", "de"],
      imagesModal: false,
      osModal: false,
      imagesList: [],
      osImagesList: [],
      socket: null,
      loggingModal: false,
      messages: "",
      ip: "",
    };
  },
  beforeDestroy() {
    if (this.socket) {
      this.socket.close();
    }
  },
  methods: {
    openFirstModal() {
      if (this.ip == "" || this.lang == null) {
        this.$q.notify({
          message: "Please fill in the IP address and select a language",
          color: "red",
          position: "top",
          timeout: 2000,
        });
        return;
      }
      this.dialog = true;
    },
    openImagesModals(images) {
      this.imagesList = images.images;
      this.imagesModal = true;
    },
    openFinalModals(osImages) {
      this.osImagesList = osImages.images;
      this.osModal = true;
    },
    connectToWS() {
      this.socket = new WebSocket("ws://" + this.ip + ":8000/ws");

      this.socket.onopen = () => {
        this.messages += "WebSocket is open now.\n";
      };

      this.socket.onmessage = (event) => {
        this.messages += event.data + "\n";
      };

      this.socket.onclose = () => {
        this.messages += "WebSocket is closed now.\n";
      };

      this.socket.onerror = (event) => {
        this.messages += "WebSocket error occurred.\n";
      };
    },
    flashImage(url, checksum) {
      this.messages = "";
      this.loggingModal = true;
      this.connectToWS();
      axios
        .post("http://" + this.ip + ":8000/flash", {
          file_url: url,
          checksum: checksum,
        })
        .then((response) => {
          this.messages += JSON.stringify(response.data) + "\n";
        })
        .catch((error) => {
          if (error.response) {
            // The request was made and the server responded with a status code
            // that falls out of the range of 2xx
            this.messages += JSON.stringify(error.response.data) + "\n";
          } else if (error.request) {
            // The request was made but no response was received
            this.messages += "No response received: " + error.message + "\n";
          } else {
            // Something happened in setting up the request that triggered an Error
            this.messages += "Error: " + error.message + "\n";
          }
        });
    },
  },
};
</script>
