<template>
  <div class="relative fixed-center">
    <div class="q-pa-md q-gutter-sm">
      <q-banner inline-actions rounded class="bg-positive text-white">
        <div id class="text-h5 h5 full-width">
          <span><a id="clicks">1</a>. </span>
          <span id="pitanje"> {{ state.pitanje }} </span>
        </div>
      </q-banner>
      <q-img width="700px" height="350px" :src="state.image" :ratio="16 / 9" />
    </div>
    <div class="q-pa-md odgovori">
      <!-- pozivanje metode getLabelOdgovor koja provjerava pitanja i daje osgovore sukladno zadanome-->
      <q-radio
        v-for="odgovor in state.odgovori"
        v-bind:key="odgovor.id"
        v-model="state.odabraniOdgovor"
        :val="odgovor.id"
        :label="getLabelOdgovor(odgovor)"
        color="teal"
      />
    </div>

    <div class="q-pa-md q-gutter-sm">
      <q-btn
        id="PrihvatiOdgovor"
        color="white"
        text-color="black"
        label="Prihvati odgovor"
        @click="
          prikaziGumb();
          state.alert = true;
          state.odabraniOdgovor === state.tocanOdgovor.id
            ? (state.brojTocnih = state.brojTocnih + 1)
            : (state.brojNetocnih = state.brojNetocnih + 1);
        "
      />
      <q-btn
        id="PrihvatiIZavrsi"
        color="white"
        text-color="black"
        label="Prihvati i zavrsi"
        @click="
          state.odabraniOdgovor === state.tocanOdgovor.id
            ? (state.brojTocnih = state.brojTocnih + 1)
            : (state.brojNetocnih = state.brojNetocnih + 1);
          state.zavrsniPopup = true;
        "
        disabled
      />
      <q-btn
        id="Refresh"
        color="white"
        text-color="black"
        label="Ponovno pokreni kviz"
        disabled
      />
      <q-dialog v-model="state.alert" persistent>
        <q-card>
          <q-card-section>
            <div class="text-h6">
              {{
                state.odabraniOdgovor === state.tocanOdgovor.id
                  ? "Točno!"
                  : "Netočno!"
              }}
            </div>
          </q-card-section>

          <q-card-section class="q-pt-none">
            <!-- //biljna vrsta pripada u botaničku porodicu botanička porodica -->

            {{
              state.odabraniOdgovor === state.tocanOdgovor.id
                ? state.plant.latin_name +
                  " je latinski naziv za " +
                  state.tocanOdgovor.croatian_name
                : state.plant.latin_name +
                  " je latinski naziv za " +
                  state.tocanOdgovor.croatian_name
            }}
          </q-card-section>

          <q-card-actions align="right">
            <q-btn
              flat
              label="OK"
              color="primary"
              @click="
                handleClose();
                brPitanja();
              "
              v-close-popup
            />
          </q-card-actions>
        </q-card>
      </q-dialog>

      <!-- Završni popup -->
      <q-dialog v-model="state.zavrsniPopup" persistent>
        <div>
          <!--State alert popup (zadnji odgovor) spojen sa završnim popup-om zbog prikaza -->
          <q-card>
            <q-card-section class="text-center">
              <div class="text-h6">
                {{
                  state.odabraniOdgovor === state.tocanOdgovor.id
                    ? "Točno!"
                    : "Netočno!"
                }}
              </div>
            </q-card-section>

            <q-card-section class="q-pt-none">
              {{
                state.odabraniOdgovor === state.tocanOdgovor.id
                  ? state.plant.latin_name +
                    " je latinski naziv za " +
                    state.tocanOdgovor.croatian_name
                  : state.plant.latin_name +
                    " je latinski naziv za " +
                    state.tocanOdgovor.croatian_name
              }}
            </q-card-section>

            <!--Rezultat popup-->

            <q-card-section class="text-center">
              <div class="text-h6" style="margin-bottom: 10px">Rezultat</div>

              <q-card-section class="q-pt-none">
                Broj točnih odgovora: {{ state.brojTocnih }}
              </q-card-section>
              <q-card-section class="q-pt-none">
                Broj netočnih odgovora: {{ state.brojNetocnih }}
              </q-card-section>
              <q-card-actions align="center">
                <q-btn
                  flat
                  label="Početna stranica"
                  color="primary"
                  v-close-popup
                  href="/"
                ></q-btn>
              </q-card-actions>
            </q-card-section>
          </q-card>
        </div>
      </q-dialog>
      <!--Kraj završnog popup-a-->
    </div>
  </div>
</template>

<script>
import { onMounted, reactive } from "vue";
import { axios } from "../boot/axios";
var clicks = 1;
export default {
  setup() {
    const state = reactive({
      tip_pitanja: 1,
      plant: {},
      pitanje: "",
      odgovori: [],
      odabraniOdgovor: {},
      tocanOdgovor: {},
      brojTocnih: 0,
      brojNetocnih: 0,
      image: "",
      alert: false,
      zavrsniPopup: false,
    });

    onMounted(async () => {
      await randomPlant();
      await getRandomBotanicalPlant();
      await getImage();
    });

    async function handleClose() {
      await randomPlant();
      await getRandomBotanicalPlant();
      await getImage(); //dodala sam ovo
    }
    //za dohvat slike
    async function getImage() {
      const json = await axios.get(
        `http://localhost:3000/image/${state.plant.id}`
      );
      const data = json.data.data;

      if (data) {
        if (
          Object.getOwnPropertyNames(json.data).length === 0 ||
          json.data.data === undefined
        ) {
          state.image = "";
        } else {
          const image = data;
          state.image = image.image_url;
        }
      } else {
        state.image = "";
      }
    }

    // funkcija koja dohvaca random plant species i postavlja vrijednost u state.plant
    async function randomPlant() {
      const jsonObject = await axios.get(
        `http://localhost:3000/plant_species/`
      );
      let randomPlant =
        jsonObject.data.data[
          Math.floor(Math.random() * jsonObject.data.data.length)
        ];

      // plant se sprema u state.plant
      state.plant = randomPlant;

      // u state.pitanje spremamo tekst pitanja
      state.pitanje = [
        "Koji je latinski naziv za " + state.plant.croatian_name + "?",
        "Koji je hrvatski naziv za " + state.plant.latin_name + "?",
        "Kojoj botaničkoj porodici pripada " + state.plant.croatian_name + "?",
        "Koja biljna vrsta se nalazi na slici?",
      ];
      const randomQuestionIndex = Math.floor(
        Math.random() * state.pitanje.length
      );
      state.tip_pitanja = randomQuestionIndex;
      state.pitanje = state.pitanje[randomQuestionIndex];
    }

    //pokušaj pisanja funkcije
    async function getAnswers() {
      if (state.pitanje == 0 || state.pitanje == 2) {
        getRandomBotanicalPlant();
        return state.plant.croatian_name;
      } else if (state.pitanje == 1 || state.pitanje == 3) {
        getRandomBotanicalPlant();
        return state.plant.latin_name;
      }
      return {
        getAnswers,
      };
    }

    // funkcija koja dohvaca random botanicke vrste i postavlja ih u listu odgovora state.odgovori
    async function getRandomBotanicalPlant() {
      const json = await axios.get(`http://localhost:3000/botanical_family`);
      const botanicalFamily = json.data.data;

      // funkcija koja dohvaca tocan odgovor i sprema ga u state.tocanOdgovor
      await getCorrectAnswerFromBotanicalFamily();

      let botanicList = [];
      botanicList.push(state.tocanOdgovor);

      // dok lista nema 2 odgovora, trazi i dodaj novi
      while (botanicList.length < 4) {
        let index = Math.round(Math.random() * (botanicalFamily.length - 1));
        let botanicObject = {
          id: botanicalFamily[index].id,
          latin_name: botanicalFamily[index].latin_name,
          croatian_name: botanicalFamily[index].croatian_name, // Adding Croatian name
        };

        ///// gledamo ako je odogovor već u listi, ako je je true-> break
        var found = false;
        for (var i = 0; i < botanicList.length; i++) {
          if (botanicList[i].id == botanicObject?.id) {
            found = true;
            break;
          }
        }
        ///// ukoliko nije odgovor u listi, dodaj ga
        if (!found) {
          botanicList.push(botanicObject);
        }

        // dodaje odgovor u listu samo ako takav odgovor vec ne postoji i ako odgovor nije jednak tocnom odgovoru
        /*if (
          botanicObject.id !== botanicList[0]?.id &&
          botanicObject.id !== state.tocanOdgovor?.id
        ) {
          botanicList.push(botanicObject);
        }*/
      }
      // nakon sto u listi odgovora imamo 2 razlicita odgovora, u listu dodajemo tocan odgovor
      // botanicList.push(state.tocanOdgovor);

      // pitanja u listi se sortiraju kako bi poredak bio random
      state.odgovori = botanicList
        .map((value) => ({ value, sort: Math.random() }))
        .sort((a, b) => a.sort - b.sort)
        .map(({ value }) => value);

      // prvi odgovor u listi postavlja se kao selektirani
      state.odabraniOdgovor = state.odgovori[0].id;
    }

    // funkcija dohvaca tocan odgovor za random plant species -> state.plant
    async function getCorrectAnswerFromBotanicalFamily() {
      const json = await axios.get(
        `http://localhost:3000/plant_species/${state.plant.id}`
      );
      state.tocanOdgovor = json.data.data;
    }

    return {
      state,
      randomPlant,
      getRandomBotanicalPlant,
      getCorrectAnswerFromBotanicalFamily,
      handleClose,
    };
  },
  methods: {
    prikaziGumb() {
      ("use strict");
      let button1 = document.getElementById("PrihvatiOdgovor");
      let button2 = document.getElementById("PrihvatiIZavrsi");
      let button3 = document.getElementById("Refresh");
      let count = 0;
      function buttonPressed(e) {
        count++;
        if (count === 8) {
          button2.removeAttribute("disabled", false);
          button3.removeAttribute("disabled", false);
          button2.innerHTML = "Prihvati i zavrsi";
          button1.setAttribute("disabled", true);
          button1.style.display = "none";
        }
      }
      button1.addEventListener("click", buttonPressed, true);
      button3.onclick = () => {
        window.location.reload();
      };
    },
    // PRORADILO... - Emina i Hrvoje
    getLabelOdgovor(odgovor) {
      const pitanje = this.state.pitanje;
      if (pitanje.includes("latinski naziv")) {
        return odgovor.latin_name;
      } else if (pitanje.includes("hrvatski naziv")) {
        return odgovor.croatian_name;
      } else if (pitanje.includes("botaničkoj")) {
        return odgovor.latin_name;
      } else if (pitanje.includes("nalazi na slici")) {
        return odgovor.croatian_name;
      }
    },

    brPitanja() {
      clicks += 1;
      document.getElementById("clicks").innerHTML = clicks;
    },
  },
};
</script>

<style>
.odgovori {
  background: rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: center;
  align-items: flex-start;
  flex-direction: column;
  width: 700px;
  margin-left: 15px;
}

@media only screen and (max-width: 1200px) {
  .odgovori {
    width: 100%;
    margin-left: 0;
  }
}

@media only screen and (max-width: 768px) {
  .odgovori {
    flex-direction: column;
    align-items: center; /* Da bi odgovori bili centrirani u koloni */
  }

  .odgovori q-radio {
    width: 100%; /* Postavljamo širinu svakog elementa na 100% */
    margin-bottom: 10px; /* Dodajemo malo prostora između odgovora */
  }

  #pitanje {
    text-align: center;
    display: block;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    max-width: 100%;
    font-size: 16px; /* Smanjujemo font da stane na screen ispod 768px */
  }
  .q-toolbar__title {
    display: none; /* Uklanjamo logo */
  }
  .q-pa-md.q-gutter-sm {
    /* Flexbox -> buttoni jedan ispod drugoga */
    display: flex;
    flex-direction: column;
    align-items: center;
  }
}
</style>
