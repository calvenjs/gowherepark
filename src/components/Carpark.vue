<template>
  <div id="carparkContainer">
    <div id="flex-layout">
      <h3 id="capark-name">{{ name }}</h3>
      <h5 id="carpark-distance">{{ distance }}</h5>
      <h5
        v-show="isGantry"
        id="carpark-lots"
        v-bind:style="{ color: textColor }"
      >
        {{ numLots }}
      </h5>
      <h5 id="carpark-type">{{ carparkType }} Carpark</h5>
      <button type="button" onclick="" id="availability">
        View Historical Parking Availability
      </button>
    </div>
    <div id="flex-layout1">
      <div id="icon-div" v-bind:style="{ height: iconDivHeight }">
        <i
          v-show="isCharging"
          class="fa-solid fa-charging-station fa-lg"
          id="charging"
        ></i>
        <i
          v-show="isHandicap"
          class="fa-brands fa-accessible-icon fa-lg"
          id="handicap"
        ></i>
      </div>
      <h5
        v-show="isCoupon"
        id="carpark-pricehr"
        v-bind:style="{ marginTop: marginTopPrice }"
      >
        ${{ priceHr }}/hr
      </h5>
      <button
        id="getdirection"
        type="button"
        @click="getDirections"
        v-bind:style="{ marginTop: marginTopButton }"
      >
        Directions
      </button>
      <br />
      <button
        v-show="isCoupon"
        type="button"
        @click="startParking"
        id="startsession"
      >
        Start Parking
      </button>
    </div>
    <div id="flex-layout2">
      <button id="bookmark" type="button" @click="getFav">
        <span id="bookmark-label">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512">
            <path
              v-bind:style="{ fill: localIsFavColor }"
              d="M0 190.9V185.1C0 115.2 50.52 55.58 119.4 44.1C164.1 36.51 211.4 51.37 244 84.02L256 96L267.1 84.02C300.6 51.37 347 36.51 392.6 44.1C461.5 55.58 512 115.2 512 185.1V190.9C512 232.4 494.8 272.1 464.4 300.4L283.7 469.1C276.2 476.1 266.3 480 256 480C245.7 480 235.8 476.1 228.3 469.1L47.59 300.4C17.23 272.1 .0003 232.4 .0003 190.9L0 190.9z"
            />
          </svg>
        </span>
      </button>
    </div>
  </div>
</template>

<script>
import firebaseApp from "../firebase.js";
import { getAuth, onAuthStateChanged } from "firebase/auth";
import { getFirestore } from "firebase/firestore";
import { doc, setDoc, deleteDoc, getDoc } from "firebase/firestore";
const db = getFirestore(firebaseApp);

export default {
  name: "Carpark",
  data() {
    return {
      user: false,
      localIsFavColor: this.isFavColor,
    };
  },
  props: {
    name: String,
    distance: String,
    numLots: String,
    carparkType: String,
    marginTopPrice: String,
    marginTopButton: String,
    priceHr: Number,
    textColor: { default: "black", type: String },
    isCoupon: Boolean,
    isGantry: Boolean,
    isFavColor: String,
    lat: Number,
    lng: Number,
    id: String,
    isCharging: { default: false, type: Boolean },
    isHandicap: { default: false, type: Boolean },
    iconDivHeight: String,
  },
  mounted() {
    const auth = getAuth(firebaseApp);
    onAuthStateChanged(auth, (user) => {
      if (user) {
        this.user = user;
      }
    });
  },
  methods: {
    getDirections: function () {
      let url = `https://www.google.com/maps/dir/?api=1&destination=${this.lat},${this.lng}&travelmode=driving`;
      window.open(url, "_blank");
    },
    getFav: async function () {
      if (this.user == false) {
        alert("Please login to bookmark carpark");
      } else {
        let bookmarked = await this.IsBookMarked(this.id);
        if (bookmarked) {
          await deleteDoc(
            doc(db, "Bookmarks", String(this.user.uid), "CarParks", this.id)
          );
          this.localIsFavColor = "black";
          alert("Bookmark deleted");
        } else {
          const docRef = doc(
            db,
            "Bookmarks",
            String(this.user.uid),
            "CarParks",
            this.id
          );
          if (this.isCoupon) {
            await setDoc(docRef, {
              CarParkID: this.id,
              name: this.name,
              carparkType: this.carparkType,
              marginTopPrice: "13px",
              marginTopButton: this.marginTopButton,
              priceHr: this.priceHr,
              textColor: this.textColor,
              isCoupon: this.isCoupon,
              isGantry: this.isGantry,
              isFavColor: "red",
              lat: this.lat,
              lng: this.lng,
              isHandicap: this.isHandicap,
              isCharging: this.isCharging,
              iconDivHeight: "0px",
            });
          } else {
            await setDoc(docRef, {
              CarParkID: this.id,
              name: this.name,
              carparkType: this.carparkType,
              marginTopPrice: this.marginTopPrice,
              marginTopButton: "35px",
              textColor: this.textColor,
              isCoupon: this.isCoupon,
              isGantry: this.isGantry,
              isFavColor: "red",
              lat: this.lat,
              lng: this.lng,
              isHandicap: this.isHandicap,
              isCharging: this.isCharging,
              iconDivHeight: "25px",
            });
          }
          this.localIsFavColor = "red";
          alert("Bookmark added");
        }
      }
      this.$emit("refreshBookmarks");
    },
    IsBookMarked: async function (id) {
      const docRef = doc(
        db,
        "Bookmarks",
        String(this.user.uid),
        "CarParks",
        id
      );
      let docSnap = await getDoc(docRef);
      if (docSnap.exists()) {
        return true;
      }
      return false;
    },
    startParking: function () {
      if (this.user == false) {
        alert("Please login to start parking");
      } else {
        this.$router.push({
          name: "Payment",
          params: {
            carparkName: this.name,
          },
        });
      }
    },
  },
};
</script>

<style>
#carparkContainer {
  display: flex;
  margin-top: 0.6rem;
  border-width: 2px;
  border-radius: 5px;
  flex-direction: row;
  padding-bottom: 0.5rem;
  border: 1px solid rgba(0, 0, 0, 0.12);
}
#flex-layout {
  margin-left: 20px;
  text-align: left;
}
h5 {
  margin: 0px;
  padding: 1px;
  font-size: 14px;
}
h3 {
  font-size: 20px;
  font-weight: bold;
  margin-top: 5px;
}
#flex-layout1 {
  margin-left: auto;
}
#icon-div {
  margin-top: 10px;
}
#charging {
  padding-right: 5px;
}
#bookmark {
  width: 30px;
  height: 30px;
  margin-top: 5px;
}
#flex-layout2 {
  margin-right: 20px;
}
#availability {
  /* remove default behavior */
  appearance: none;
  -webkit-appearance: none;
  /* usual styles */
  border: none;
  background-color: #3f51b5;
  color: #fff;
  border-radius: 5px;
  width: 300px;
  margin-top: 5px;
}
#getdirection {
  /* remove default behavior */
  appearance: none;
  -webkit-appearance: none;
  /* usual styles */
  border: none;
  background-color: #b22222;
  color: #fff;
  border-radius: 5px;
  margin-top: 5px;
  width: 100px;
}
#startsession {
  /* remove default behavior */
  appearance: none;
  -webkit-appearance: none;
  /* usual styles */
  border: none;
  background-color: #04d010;
  color: #fff;
  border-radius: 5px;
  margin-top: 5px;
  width: 100px;
}
</style>