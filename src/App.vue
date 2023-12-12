<template>
  <div class="user-links animate__animated animate__fadeIn animate__slow">
    <div
      v-for="userLink in userLinks"
      :key="userLink"
      class="link"
      :suggested="suggestedLink == userLink"
      @mouseenter="checkhold(userLink)"
      @mouseleave="removehold()"
    >
      <div
        v-if="this.hoveredLink == userLink && this.showRemove"
        class="remove-link animate__animated animate__slideInUp animate__faster"
      >
      <i class="las la-trash" @click="removeItem(userLink.name)"></i>
      </div>
      <img :src="userLink.img" alt="" @mouseup="gotoURL(userLink.url)" />
    </div>
  </div>

  <div class="input-wrap">
    <input
      id="inpMain"
      v-model="searchTerm"
      type="text"
      @focus="$event.target.select()"
      @keyup="checkSubmit($event)"
      @blur="this.searchTerm = ''"
    />
  </div>
</template>

<script>
import axios from "axios";

export default {
  mounted() {
    //focus the input
    setTimeout(() => {
      document.querySelector("#inpMain").focus();
    }, 300);

    //check for key inputs
    document.addEventListener("keyup", (e) => {
      if (document.querySelector("#inpMain") != document.activeElement) {
        //if the input is focused
        document.querySelector("#inpMain").focus();
        this.searchTerm += e.key;
      }
    });

    //get the user data
    this.loadUserLinks();
  },
  methods: {
    checkSubmit(e) {
      if (e.code == "Enter") {
        //check addnew
        if (this.searchTerm.toLowerCase().startsWith("addnew")) {
          this.addNewLink();
          return;
        }

        //check selected
        if (this.suggestedLink) {
          this.searchTerm = "";
          window.location.href = this.suggestedLink.url;
          return;
        }

        //begin the search
        window.location.href =
          "https://www.google.com/search?q=" + this.searchTerm;
        //console.log('https://www.google.com/search?q=' + this.searchTerm);
      } else {
        //check for up down
        if (this.searchTerm.includes(" ")) {
          this.suggestedLink = null;
          return;
        }
        //check for suggestions
        this.userLinks.forEach((userLink) => {
          //GC
          if (
            userLink.name
              .toLowerCase()
              .startsWith(this.searchTerm.toLowerCase()) &&
            this.searchTerm != ""
          ) {
            this.suggestedLink = userLink;
            return;
          } else {
            //this.suggestedLink = null;
            if (this.searchTerm == "") {
              this.suggestedLink = null;
            }
          }
        });
      }

      //alert(e.code);
    },

    checkhold(e) {
      this.hoveredLink = e;
      setTimeout(() => {
        //if we still hovering..
        if (this.hoveredLink) {
          this.showRemove = true;
        } else {
          this.showRemove = false;
        }
      }, 500);
    },

    removehold() {
      this.hoveredLink = null;
      this.showRemove = false;
    },

    gotoURL(link) {
      this.hoveredLink = null;
      window.location.href = link;
    },

    async loadUserLinks() {
      await axios
        .post("https://ashypls.com/endpoints/launcher.asmx/getUserLinks", {
          contentType: "application/json",
          username: this.user,
        })
        .then((response) => {
          this.userLinks = JSON.parse(response.data.d);
          //this.suggestedLink = this.userLinks[0];
          //this.hoveredLink = this.userLinks[0];
        });
    },

    async addNewLink() {
      await axios
        .post("https://ashypls.com/endpoints/launcher.asmx/saveNewLink", {
          contentType: "application/json",
          addString: this.searchTerm,
          userName: this.user,
        })
        .then((response) => {
          //log the response to the console
          console.log(response.data.d);
          this.loadUserLinks();
        });
    },

    async removeItem(itemName){
      await axios
        .post("https://ashypls.com/endpoints/launcher.asmx/removeLink", {
          contentType: "application/json",
          itemName: itemName,
          username: this.user,
        })
        .then((response) => {
          //log the response to the console
          console.log(response.data.d);
          this.loadUserLinks();
        });
    },

  },
  data() {
    return {
      searchTerm: "",
      user: "ash",
      userLinks: [],
      suggestedLink: null,
      hoveredLink: null,
      showRemove: false,
    };
  },
};
</script>

<style>
</style>