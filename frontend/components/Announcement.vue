<!-- 
    Announcement.vue is for the Announcement Pane (top-left) This utilizes a plug-in called tiny slider which actively slides through the created div tags beneath it.
    The div tags are created dynamically through the v-for loop. They loop through all announcements currently present in the database. This also has code 
    beneath for refreshing the slider so it dynamically updates. The variable 'renderSlider' which is utilized in a v-if is used to do this.
    It changes to false when a new database item is loaded causing tiny slider to no longer render. It changes back to true immediately after which refreshes tiny slider.
 -->

<template>
  <section class="ticker__container" v-if="renderSlider && announcements.length">
    <client-only>
      <vue-tiny-slider
        ref="tinySlider"
        class="list--plain ticker"
        :items="1"
        slide-by="page"
        :controls="false"
        :nav="true"
        navPosition="bottom"
        :autoplay-button-output="false"
        :speed="3000"
        :autoplay="true"
        :autoplay-timeout="5000"
      >
        <div class="ticker__item tns-item" :key="m.Id" v-for="m in announcements">
          <div class="testtest">{{m.message}}</div>
        </div>
      </vue-tiny-slider>
    </client-only>
  </section>
</template>

<!-- The scripts for the component. This script contains data attributes and methods for this component. -->
<script>
import schedule from "node-schedule";
import Vue from "vue";

export default {
  name: "Announcement",
  props: ["url"],
  updated: function() {
    var that = this;
    window.test = this;

    if (!this.renderSlider) {
      Vue.nextTick(function() {
        that.renderSlider = true;
      });
    }
  },
  data: function() {
    return {
      numberOfSlides: 0,
      renderSlider: true,
      currentAnnouncements: [],
      hostURL: this.url
    };
  },
  methods: {
    // These two methods are helper methods used in the methods below.
    getCurrentDate: function() {
      return new Date()
        .toJSON()
        .slice(0, 10)
        .replace(/-/g, "-");
    },
    compareDates: function(date1, date2) {
      var d1 = Date.parse(date1);
      var d2 = Date.parse(date2);
      if (d1 <= d2) {
        return true;
      } else {
        return false;
      }
    },

    // deleteAnnouncement() uses an AJAX fetch to delete something in the database. It is called in the scheduler below.
    deleteAnnouncement(uniqueID) {
      var headers = {
        "Content-Type": "application/json"
      };
      var data = {
        message: this.message,
        name: this.name,
        id: uniqueID,
        date: this.startDate,
        title: this.eventTitle,
        startDate: this.startDate,
        endDate: this.endDate,
        startsAt: this.startDate + this.startTime,
        endsAt: this.endDate + this.endTime
      };
      fetch(this.hostURL + "announcement", {
        method: "DELETE",
        headers: headers,
        body: JSON.stringify(data)
      });
    },

    // scheduledDatabaseMaintenance() will repeat the given function on a given schedule.
    scheduledDatabaseMaintenance: function() {
      var self = this;
      schedule.scheduleJob("*/1 * * * *", function() {
        console.log("Announcements Called.");
        for (var i = 0; i < self.announcements.length; i++) {
          if (
            self.compareDates(
              self.announcements[i].expiresAt,
              self.getCurrentDate()
            )
          ) {
            console.log("DELETED");
            console.log(self.announcements[i].id);
            self.deleteAnnouncement(self.announcements[i].id);
            self.announcements.splice(i, i);
          }
        }
      });
    }
  },
  mounted() {
    this.scheduledDatabaseMaintenance();
  },
  watch: {
    announcements() {
      this.numberOfSlides = this.$store.state.announcement.all.length;
    },
    numberOfSlides: function(val, oldVal) {
      if (val != oldVal) {
        this.renderSlider = false;
      }
    }
  },
  computed: {
    announcements() {
      return this.$store.state.announcement.all;
    }
  }
};
</script>

<!-- Any styles for this component. These styles are scoped meaning they only hold value within the component. --> 
<style scoped>
  .tns-nav {
    width: 3em;
    height: 3em;
  }
</style>
