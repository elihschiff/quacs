<template>
  <div id="app">
    <div id="wrapper">
      <UnofficialScheduleModal />

      <nav class="navbar navbar-expand-lg navbar-light bg-light">
        <router-link class="navbar-brand" to="/"
          ><img
            src="@/assets/images/quacs_logo_thanksgiving.svg"
            alt="QuACS Home"
            style="height: 27px"
        /></router-link>
        <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>
        <b-collapse id="nav-collapse" is-nav>
          <b-input-group>
            <input
              id="search-bar"
              placeholder="Search Courses"
              aria-label="Search Courses"
              v-on:input="search($event.target.value)"
              v-on:keyup.enter="search($event.target.value, 0)"
            />
            <b-spinner
              label="Loading"
              v-if="searching || !wasmLoaded"
              class="loading-spinner"
            ></b-spinner>
          </b-input-group>
          <b-navbar-nav class="ml-auto">
            <b-navbar-nav>
              <CourseSetEdit></CourseSetEdit>
              <b-nav-item-dropdown left :title="shortSemToLongSem(currentSem)">
                <template v-slot:button-content>
                  <em class="nav-text" style="font-style: normal">{{
                    shortSemToLongSem(currentSem)
                  }}</em>
                </template>
                <b-dropdown-item
                  v-for="shortSem in allSems"
                  :key="shortSem"
                  :href="shortSemToURL(shortSem)"
                  :title="shortSemToLongSem(shortSem)"
                  >{{ shortSemToLongSem(shortSem) }}</b-dropdown-item
                >
              </b-nav-item-dropdown>
              <b-nav-item class="nav-text desktop-only" disabled>|</b-nav-item>
              <b-nav-item
                to="/prerequisites"
                class="nav-text"
                :active="this.$route.path == '/prerequisites'"
                >Prerequisites</b-nav-item
              >
              <b-nav-item
                to="/schedule"
                class="nav-text"
                :active="this.$route.path == '/schedule'"
                >Schedule</b-nav-item
              >
              <b-nav-item
                v-if="installable"
                class="nav-text mobile-only"
                @click="installPrompt()"
                >Install QuACS App</b-nav-item
              >
              <b-nav-item class="nav-text" v-b-modal.settings-modal>
                <font-awesome-icon
                  title="Settings"
                  :icon="['fas', 'cog']"
                ></font-awesome-icon>
              </b-nav-item>
            </b-navbar-nav>
          </b-navbar-nav>
        </b-collapse>
      </nav>

      <div class="container-fluid" style="margin-top: 1rem">
        <div class="row">
          <div class="col-lg-1"></div>
          <div class="col-lg">
            <router-view :key="wasmLoaded" v-if="wasmLoaded" />
            <!-- <b-alert
              variant="warning"
              class="fixed-bottom sticky-top"
              :show="shouldShowAlert"
              ><b-spinner
                style="width: 1.5rem; height: 1.5rem;"
                label="Spinning"
              ></b-spinner
              ><span class="warning-message">{{
                warningMessage
              }}</span></b-alert
            > -->
            <b-alert
              class="fixed-bottom sticky-top"
              :show="updateAvailable"
              dismissible
            >
              Updates available! Click to refresh and update.
              <b-button variant="success" @click="reloadPage()"
                >Update</b-button
              >
            </b-alert>
          </div>
          <div class="col-lg-1"></div>
        </div>
      </div>
    </div>
    <Settings></Settings>
    <footer class="footer">
      <div class="footer-links">
        <a
          href="https://discord.gg/yXaHkwU"
          v-on:click="track('Visit Discord', 'Footer')"
          rel="noopener"
          title="Join our development Discord server"
          aria-label="Join our development Discord server"
          target="_blank"
          ><font-awesome-icon :icon="['fab', 'discord']"></font-awesome-icon>
        </a>
        <a
          href="https://github.com/quacs/quacs"
          v-on:click="track('Visit GitHub', 'Footer')"
          rel="noopener"
          title="Visit our GitHub"
          aria-label="Visit our GitHub"
          target="_blank"
          ><font-awesome-icon :icon="['fab', 'github']"></font-awesome-icon>
        </a>
        <!--img
          id="footer-logo"
          src="@/assets/images/quacs_white.svg"
          alt="QuACS"
          style="height: 40px"
          @click="rotateLogo()"
        /-->
        <a
          href="https://patreon.com/quacs"
          v-on:click="track('Visit Patreon', 'Footer')"
          rel="noopener"
          title="Sponsor us on Patreon!"
          aria-label="Sponsor us on Patreon!"
          target="_blank"
          ><font-awesome-icon :icon="['fab', 'patreon']"></font-awesome-icon>
        </a>
      </div>
      <div class="footer-sponsors">
        <router-link to="/sponsors"> View our sponsors</router-link>
      </div>
      <div class="footer-updated">
        Last updated {{ lastUpdated }} (<a
          :href="'https://github.com/quacs/quacs/commit/' + quacsHash"
          style="color: var(--raw-link)"
          >site</a
        >,
        <a
          :href="'https://github.com/quacs/quacs-data/commit/' + dataHash"
          style="color: var(--raw-link)"
          >data</a
        >)
      </div>
      <div class="footer-copyright">
        &copy; {{ new Date().getFullYear() }} - Questionably Accurate Course
        Scheduler
      </div>
    </footer>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { mapGetters, mapState } from "vuex";
import {
  BAlert,
  BButton,
  BCollapse,
  BDropdownItem,
  BInputGroup,
  BNavItem,
  BNavItemDropdown,
  BNavbar,
  BNavbarNav,
  BNavbarToggle,
  BSpinner,
  VBModal,
  VBTooltip,
} from "bootstrap-vue";
import Settings from "@/components/Settings.vue";
import CourseSetEdit from "@/components/CourseSetEdit.vue";
import UnofficialScheduleModal from "@/components/UnofficialScheduleModal.vue";
import { shortSemToLongSem, shortSemToURL, trackEvent } from "@/utilities";

@Component({
  components: {
    Settings,
    CourseSetEdit,
    UnofficialScheduleModal,
    "b-alert": BAlert,
    "b-button": BButton,
    "b-collapse": BCollapse,
    "b-input-group": BInputGroup,
    "b-navbar": BNavbar,
    "b-nav-item": BNavItem,
    "b-navbar-nav": BNavbarNav,
    "b-navbar-toggle": BNavbarToggle,
    "b-spinner": BSpinner,
    "b-nav-item-dropdown": BNavItemDropdown,
    "b-dropdown-item": BDropdownItem,
  },
  directives: {
    "b-modal": VBModal,
    "b-tooltip": VBTooltip,
  },
  computed: {
    ...mapGetters(["shouldShowAlert", "warningMessage"]),
    ...mapGetters("schedule", ["getCourseSets"]),
    ...mapState("schedule", ["wasmLoaded", "currentCourseSet", "courseSets"]),
    shortSemToURL,
    shortSemToLongSem,
    updateAvailable: {
      get() {
        return this.$store.state.updateAvailable;
      },
      set() {
        this.$store.commit("toggleUpdateNotice", false);
      },
    },
  },
})
export default class App extends Vue {
  searchCallback: number | null = null;
  searching = false;
  installable = false;
  installEvent: Event | null = null;

  get allSems(): string[] {
    return JSON.parse(process.env.VUE_APP_ALL_SEMS);
  }

  get currentSem(): string {
    return process.env.VUE_APP_CURR_SEM;
  }

  get lastUpdated(): string {
    let timeDifference =
      (new Date().getTime() -
        new Date(this.$store.state.dataStats.last_updated).getTime()) /
      1000;
    const seconds = Math.floor(timeDifference % 60);
    timeDifference = timeDifference / 60;
    const minutes = Math.floor(timeDifference % 60);
    timeDifference = timeDifference / 60;
    const hours = Math.floor(timeDifference % 24);
    const days = Math.floor(timeDifference / 24);
    if (days > 0) {
      return `${days} day${days !== 1 ? "s" : ""} ago`;
    } else if (hours > 0) {
      return `${hours} hour${hours !== 1 ? "s" : ""} ago`;
    } else if (minutes > 0) {
      return `${minutes} minute${minutes !== 1 ? "s" : ""} ago`;
    }
    return `${seconds} second${seconds !== 1 ? "s" : ""} ago`;
  }

  get quacsHash(): string {
    return process.env.VUE_APP_QUACS_HASH;
  }

  get dataHash(): string {
    return process.env.VUE_APP_DATA_HASH;
  }

  search(input: string, searchTimeout = 250): void {
    this.searching = true;

    if (this.searchCallback !== null) {
      clearTimeout(this.searchCallback as number);
    }

    if (input.length === 0) {
      this.searching = false;
      this.$router.push("/").catch(() => {
        return;
      });
    } else {
      this.searchCallback = setTimeout(() => {
        this.$router.push(`/search?${encodeURIComponent(input)}`).catch(() => {
          this.searching = false;
          return;
        });
        this.searching = false;
      }, searchTimeout);
    }
  }

  reloadPage(): void {
    // The 'reload' function in location has a non-standard 'forceGet' operator
    // which clears the cache. Typescript doesn't like this; however, in browsers
    // that don't support it, the extra argument is harmless.
    // @ts-expect-error: see above
    window.location.reload(true);
  }

  rotateLogo(): void {
    const footer = document.getElementById("footer-logo");
    if (footer && !footer.classList.contains("footer-logo-rotate")) {
      footer.classList.add("footer-logo-rotate");
      setTimeout(function () {
        footer.classList.remove("footer-logo-rotate");
      }, 500);
    }
  }

  created(): void {
    window.addEventListener("beforeinstallprompt", (e) => {
      e.preventDefault();
      this.installEvent = e;
      this.installable = true;
    });
  }

  installPrompt(): void {
    if (this.installEvent !== null) {
      // @ts-expect-error: ts does understand this event
      this.installEvent.prompt();
      // @ts-expect-error: ts does understand this event
      this.installEvent.userChoice.then(() => {
        this.installEvent = null;
      });
    }
  }

  track(event_value: string, event_type: string): void {
    trackEvent(event_value, event_type);
  }
}
</script>

<style>
@import "./assets/styles/main.css";

.footer {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-top: 2rem;
  padding-top: 2rem;
  padding-bottom: 2rem;
  background: var(--footer-background);
}

.footer-links > * {
  color: var(--global-text);
  font-size: 2.4rem;
  padding: 0rem 1rem;
}

.footer-links > a:hover {
  color: DimGrey;
}

.footer-links {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 1rem;
}

.footer-sponsors > * {
  color: var(--global-text);
  font-size: 1.5rem;
  padding: 0rem 1rem;
}

.footer-sponsors > a:hover {
  color: DimGrey;
}

.footer-sponsors {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 1rem;
}

.footer-copyright {
  color: var(--global-text);
  font-size: 1rem;
  padding: 0rem 1rem;
  text-align: center;
}

.nav-text {
  font-size: 1.25rem;
}

#search-bar {
  width: 100%;
  border: 1px solid #eee;
  border-radius: 8px;
  padding: 6px 6px 6px 42px;
  box-sizing: border-box;
  position: relative;
  font-size: 16px;
  line-height: 1.5;
  /* flex: 1; */
  background-color: #eee;
  background-image: url("data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyNCIgaGVpZ2h0PSIyNCIgZmlsbD0ibm9uZSIgc3Ryb2tlPSIjNjY2IiBzdHJva2Utd2lkdGg9IjIiIHN0cm9rZS1saW5lY2FwPSJyb3VuZCIgc3Ryb2tlLWxpbmVqb2luPSJyb3VuZCI+PGNpcmNsZSBjeD0iMTEiIGN5PSIxMSIgcj0iOCIvPjxwYXRoIGQ9Ik0yMSAyMWwtNC00Ii8+PC9zdmc+");
  background-repeat: no-repeat;
  background-position: 12px;
}

#search-bar:focus {
  border-color: rgba(0, 0, 0, 0.12);
  background-color: #fff;
  outline: none;
  box-shadow: 0 2px 2px rgba(0, 0, 0, 0.16);
}

.warning-message {
  font-size: 1.5rem;
  margin-left: 1.5rem;
}

/* For sticky footer */
html,
body,
#app {
  height: 100%;
}

#app {
  display: flex;
  flex-direction: column;
}

#wrapper {
  flex: 1 0 auto;
}

.footer {
  flex-shrink: 0;
}
/* End for sticky footer */

@media (min-width: 992px) {
  #search-bar {
    width: 300px;
  }
}

.footer-logo-rotate {
  transform: rotate(360deg);
  transition-duration: 0.5s;
}
</style>
